---
icon: ubuntu
---

# plum - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.28 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 09:47 EDT
Nmap scan report for 192.168.224.28
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: PluXml - Blog or CMS, XML powered !
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   30.69 ms 192.168.45.1
2   30.55 ms 192.168.45.254
3   30.72 ms 192.168.251.1
4   31.67 ms 192.168.224.28
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2359).png" alt=""><figcaption></figcaption></figure>

At the bottom of the page we find an admin link, let's check it out

<figure><img src="../../.gitbook/assets/image (2360).png" alt=""><figcaption></figcaption></figure>

Default admin:admin creds work

<figure><img src="../../.gitbook/assets/image (2361).png" alt=""><figcaption></figcaption></figure>

And we are presented with a version

### <mark style="color:$primary;">PluXml CMS v5.8.7 RCE</mark>

It seems we can edit static pages

<figure><img src="../../.gitbook/assets/image (2363).png" alt=""><figcaption></figcaption></figure>

I'll try editing the first page with a php pentest monkey reverse shell [**link**](https://www.revshells.com/)

<figure><img src="../../.gitbook/assets/image (2364).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2365).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready before clicking on view page

<figure><img src="../../.gitbook/assets/image (2366).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (2367).png" alt=""><figcaption></figcaption></figure>

Linpeas reveals some mail let's check it out

<figure><img src="../../.gitbook/assets/image (2368).png" alt=""><figcaption></figcaption></figure>

Email reveals root creds

<figure><img src="../../.gitbook/assets/image (2369).png" alt=""><figcaption></figcaption></figure>
