---
layout: post
title:  "Defcon RedTeam Village CTF all programming challenges solution"
date:   2020-08-08
categories: CTF
thumbnail: /img/tps.webp
tags: ctf,defcon,redteamvillage
---
In the qualifiers round of Defcon RedTeam Village CTF, there were 8 Jeopardy-style challenges in the Programming section. The highest point was in the "TPS Report System - 2" challenge which only unlocked after solving the first one and had very few solves. 

## # TPS Report System - 1

<img src="/img/tps.webp" width="50%"> 

Upon connecting to the host via netcat it displayed the above message. After sending `1` as input it showed below message:
```
Enter the Report you would like to print (TPS-XXXX):

``` 
Found the right TPS report id by brute-forcing numbers from 0000-9999 via Python3 script using `pwntools`:
```python
import sys
from pwn import *

r = remote("161.35.239.216",5000)
r.recv()
r.sendline("1")
d = r.recv()

for x in range(0,9999):
    code = str(x).zfill(4)
    code = 'TPS-'+str(code)
    print('[+] Trying code : %s' % code)
    r.sendline(code)
 
    resp = r.recv().decode()
    print(resp)
    if 'Lumbergh' in resp:
        continue
    else:
        print(resp)
```
This was what I came up with and it isn't neat, it would crash whenever a TPS report ID returned some sort of valid response instead of the default `A cover sheet wasn't attached to that report` message. To complete the challenge in a rush I just adjusted the first range value to the next id in the script and kept relaunching it until I found the right ID. 

After sending TPS-8352, saw a different Error message 
```
Wrong
Report is password protected
```
The challenge was to find this ID and it was the flag.

## # TPS Report System - 2

After submitting the flag for the first one, this challenge was unlocked. This challenge required cracking the password for the report. Many even tried rockyou.txt for this but failed. Obviously it required bruteforce, and the wordlist for this was the movie script of Office Space! what I did was copied the whole transcript from [here](http://www.script-o-rama.com/movie_scripts/o/office-space-script-transcript.html) and saved in a file, split on space and sorted uniquely which generated a 3029 line wordlist :
```bash
$ cat movie.txt | tr ' ' '\n' | sort -u > wordlist.txt
$ wc -l wordlist.txt                                                            
3029 wordlist.txt
```
Next updated my previous script to bruteforce for the right password:
```python
from pwn import *

words = open('wordlist.txt').read().replace('\r','').split('\n')
r = remote("161.35.239.216",5000)
r.recv()
r.sendline("1")
d = r.recv()
r.sendline("TPS-8352")
e =r.recv().decode()
print(e)

for passwd in words:
    print('[+] Trying : %s' % passwd)
    r.sendline(passwd)
    resp = r.recvline()
    if b'Wrong' in resp:
        r.recvline()
        r.recvline()
        continue
    print(str(resp))
    if not b'Wrong' in resp:
        sys.exit('[+] Found! \n%s' % resp)

```
Found the right password after 456 attempts. It was `Chotchkie's`. The flag is displayed after submitting right password.

## # Ping Pong
This was a very straight forward challenge. All it did was print a Character and we had to type it back. It printed each character of the flag line by line.

## # Ping Pong 2
This time there was a time limit so it required fast input. So I scraped together below script to quickly submit input and receive the flag:
```python
from pwn import *

def loopwithwhatihave(string):
	r = remote("164.90.147.2",2346)
	for x in string:
		r.sendline(x)
	lc = r.recv().decode().replace('\n','').replace('\r','')
	return(lc)

string = "T"
while True:
	lc = loopwithwhatihave(string)
	string = string + lc
	print(string)
```
Due to the time limit I decided to grab one more char everytime and re-initiate the connection. Maybe this could have been done more cleaner way but this was what I came up with at that time.

## # Networks
This challenge provided a list of 1995 lines which contained a list in `<ip>,<cidr>` format. To solve this it was required to check if the ip address was valid for the given subnet range. wrote below script to solve it:
```python
from ipaddress import ip_network, ip_address

dl = open('cidr_ip.txt','r').read().replace('\r','').split('\n')
if dl[-1] == '':
	dl.pop()

dc = open('dump.txt','a+') 
for xline in dl:
	try:
		ipaddr,cidr = xline.split(',')
	except:
		print('error : %s' %xline)
		continue
	net = ip_network(cidr)
	if ip_address(ipaddr) in net:
		dc.write(ipaddr+'\n')
		print('valid : %s' % ipaddr)
	else:
		print('invalid : %s' % ipaddr)

dc.close()
```
It saved 48 IP addresses in the dump.txt and the count was the flag for this challenge.

## # Roll for Initiative 1
In this challenge we had to play a dice20 game where we had to input a number between 1 to 20 and if the number matched with what the program has already chosen for us we would get correct message. The first challenge required winning 10 times in a row to get the flag. The flaw here was that it chose exact same numbers everytime so I just basically kept guessing and recording which one was correct next and got the flag quickly. The sequence was :
```
14 7 3 11 19 17 2 10 5 20
```
## # Roll for Initiative 2
The challenge and the flaw was same but we had to guess 100 times and doing it manually would take a long time so coded below script to get the job done :
```python
from pwn import *
import re

def loopwithwhatihave(arr):
	r = remote("164.90.147.2",1235)
	for x in arr:
		r.sendline(x)
		print(r.recv().decode())
	print(r.recv().decode())
	r.sendline('1337')
	fsc = r.recv().decode()
	fc = re.findall(r'looking for was (\d+)\n',fsc,re.DOTALL)
	return(fc[0])

arr = ['15']
while len(arr) < 202:
	lc = loopwithwhatihave(arr)
	arr.append(lc)
	print(arr)

```
After some time reached 100 correct guesses and got the flag.
## # Roll for Initiative 3
This time the numbers were random so we couldn't use the same tactic like last two times. The good thing was it required only 2 right guesses so decided to brute instead, if the first guess was right only then attempted second guess:
```python
from pwn import *
import re,random

def loopwithwhatihave(arr):
	r = remote("164.90.147.2",1236)
	r.recv()
	r.sendline(arr[0])
	try:
		dx = r.recv().decode()
		if "Correct" in dx:
			print(dx)
			r.sendline(arr[1])
			dy = r.recv().decode()
			print(dy)
	except:
		return
	


while True:
	arr = [str(random.randint(1,10)),str(random.randint(1,10))]
	loopwithwhatihave(arr)
``` 
After few minutes, guessed the right number two times in a row and got the flag.

This was my first ever participation in a CTF contest. There were 690 teams comprised of 1428 users and I managed to get into 112th place under the given 24 hours playing Solo! I spent way more time solving some hard ones, later solved many more challenges after the qualifiers time for closure and the organizers were very kind to leave the CTF open for us. Decided to start my own blog to document some of my experiences as I hack and learn now and in the future!