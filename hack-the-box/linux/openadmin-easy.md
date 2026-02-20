---
icon: ubuntu
---

# OpenAdmin - Easy

<figure><img src="../../.gitbook/assets/image (32).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/openadmin"><strong>OpenAdmin</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.6.147 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-30 20:24 -0500
Nmap scan report for 10.129.6.147
Host is up (0.039s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 256/tcp)
HOP RTT      ADDRESS
1   47.58 ms 10.10.16.1
2   26.24 ms 10.129.6.147
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Directory Busting</mark>

```shellscript
feroxbuster -u http://10.129.6.147/ -n
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Found 3 interesting endpoints

#### <mark style="color:$primary;">/music</mark>

