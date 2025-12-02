---
icon: ubuntu
---

# Help - Easy

<figure><img src="../../.gitbook/assets/image (2979).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/help"><strong>Help</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.121 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-01 21:11 EST
Nmap scan report for help.htb (10.10.10.121)
Host is up (0.044s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 199/tcp)
HOP RTT      ADDRESS
1   30.42 ms 10.10.16.1
2   59.31 ms help.htb (10.10.10.121)
```

### <mark style="color:$primary;">HTTP Port 3000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2980).png" alt=""><figcaption></figcaption></figure>

This port Hosts an API, we see a message about geting credentials with the correct query

