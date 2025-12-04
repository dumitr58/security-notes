---
icon: ubuntu
---

# LinkVortex - Easy

<figure><img src="../../.gitbook/assets/image.png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/linkvortex"><strong>LinkVortex</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.47 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-03 21:54 EST
Nmap scan report for linkvortex.htb (10.10.11.47)
Host is up (0.048s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
|_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
80/tcp open  http    Apache httpd
|_http-generator: Ghost 5.58
|_http-title: BitByBit Hardware
| http-robots.txt: 4 disallowed entries 
|_/ghost/ /p/ /email/ /r/
|_http-server-header: Apache
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT      ADDRESS
1   99.35 ms 10.10.16.1
2   37.09 ms linkvortex.htb (10.10.11.47)
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

Visiting the page we will get a redirect to linkvortex.htb. I'll add that to my hosts file

