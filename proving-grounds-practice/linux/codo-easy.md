---
icon: ubuntu
---

# Codo - Easy

## Gaining Access

Nmap Scan:

```
nmap -A -T4 -p- -Pn 192.168.118.23 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-23 15:18 EDT
Nmap scan report for 192.168.118.23
Host is up (0.030s latency).
Not shown: 65533 filtered tcp ports (no-response)
Bug in http-generator: no string output.
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: All topics | CODOLOGIC
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%), Linux 4.15 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   27.41 ms 192.168.45.1
2   27.27 ms 192.168.45.254
3   27.44 ms 192.168.251.1
4   31.62 ms 192.168.118.23
```

### HTTP Port 80 TCP

<figure><img src="../../.gitbook/assets/image (1490).png" alt=""><figcaption></figcaption></figure>

I am going to start with directory busting.

```
dirsearch -u http://192.168.118.23/
```

<figure><img src="../../.gitbook/assets/image (1485).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1492).png" alt=""><figcaption></figcaption></figure>

There is an admin login page, if we visit it and try the admin:admin credentials they will work

<figure><img src="../../.gitbook/assets/image (1493).png" alt=""><figcaption></figcaption></figure>

We got a version, let's see what searchsploit has for us:

### CodoForum v5.1 RCE

<figure><img src="../../.gitbook/assets/image (1487).png" alt=""><figcaption></figcaption></figure>

There is an RCE available I am going to download it and try it:

<figure><img src="../../.gitbook/assets/image (1488).png" alt=""><figcaption></figcaption></figure>

I checked the help and ran the exploit but it did not work. I noticed that the exploit tries to upload an image to this location:

<figure><img src="../../.gitbook/assets/image (1494).png" alt=""><figcaption></figcaption></figure>

So I moved to the webpage and started searching for places to upload an object. Under “Global Settings” I found a place to upload an image for the logo

<figure><img src="../../.gitbook/assets/image (1495).png" alt=""><figcaption></figcaption></figure>

I made a pentest monkey reverse.php shell for port 80 using [https://www.revshells.com/](https://www.revshells.com/) and uploaded it

<figure><img src="../../.gitbook/assets/image (1496).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1497).png" alt=""><figcaption></figcaption></figure>

Now it shows my reverse.php instead of the old logo

Then I visited the shell on the same payload path and got a shell on the listener

```
http://192.168.118.23/sites/default/assets/img/attachments/reverse.php
```

<figure><img src="../../.gitbook/assets/image (1499).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1498).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

I am going to take a look at the conifg files first for some credentials:

```
find . -name '*conf*' -type f 2>/dev/null
```

<figure><img src="../../.gitbook/assets/image (1500).png" alt=""><figcaption></figcaption></figure>

We found some credentials, let's see how many users have access to a shell

```
grep 'sh$' /etc/passwd
```

<figure><img src="../../.gitbook/assets/image (1501).png" alt=""><figcaption></figcaption></figure>

i am going to try the creds on root

<figure><img src="../../.gitbook/assets/image (1502).png" alt=""><figcaption></figcaption></figure>

Credential reuse! we got root!
