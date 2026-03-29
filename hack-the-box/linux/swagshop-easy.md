---
icon: ubuntu
---

# Swagshop - Easy

<figure><img src="../../.gitbook/assets/image (3401).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/swagshop"><strong>SwagShop</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP Scan" overflow="wrap" expandable="true" %}
```shellscript
nmap -A T4 -p- -Pn 10.129.229.138 -oN scans/nmap-tcpall                                                                                         
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-25 20:21 -0400
Failed to resolve "T4".
Nmap scan report for 10.129.229.138
Host is up (0.039s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Did not follow redirect to http://swagshop.htb/
|_http-server-header: Apache/2.4.29 (Ubuntu)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 554/tcp)
HOP RTT      ADDRESS
1   95.94 ms 10.10.16.1
2   25.93 ms 10.129.229.138
```
{% endcode %}

We have a redirect detected by nmap I'll add it to my hosts file

{% code title="/etc/hosts" overflow="wrap" expandable="true" %}
```shellscript
10.129.229.138	swagshop.htb
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

### <mark style="color:$primary;">Tech Detection</mark>

{% code title="curl -I http://swagshop.htb" overflow="wrap" expandable="true" %}
```shellscript
HTTP/1.1 200 OK
Date: Thu, 26 Mar 2026 00:26:44 GMT
Server: Apache/2.4.29 (Ubuntu)
Set-Cookie: frontend=mkvgfogus4pbl39cnjdm2cc197; expires=Thu, 26-Mar-2026 01:26:45 GMT; Max-Age=3600; path=/; domain=swagshop.htb; HttpOnly
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
X-Frame-Options: SAMEORIGIN
Content-Type: text/html; charset=UTF-8
```
{% endcode %}

Apache Server, with an interesting set of cookies, feroxbuster also reveals it's running php

### <mark style="color:$primary;">Website</mark>

<figure><img src="../../.gitbook/assets/image (3402).png" alt=""><figcaption></figcaption></figure>

First thing that stood out to me was Magento, that name sounded like a CMS name was not sure but I kept it in mind

### <mark style="color:$primary;">Directory Busting</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
feroxbuster -u http://swagshop.htb -n
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3403).png" alt=""><figcaption></figcaption></figure>

Found something under var -> [http://swagshop.htb/var/package/](http://swagshop.htb/var/package/)

<figure><img src="../../.gitbook/assets/image (3404).png" alt=""><figcaption></figcaption></figure>

We see multiple references to magento with different versions so I am going to check out searchsploit.&#x20;

But first I want to check if there is any form of admin panel. I'll have feroxbuster run and than grep the file for anything

{% code overflow="wrap" expandable="true" %}
```shellscript
feroxbuster -u http://swagshop.htb
```
{% endcode %}

{% code title="querying feroxbuster out file for any admin endpoints" overflow="wrap" expandable="true" %}
```shellscript
jq -r '.responses[] | .url?' ferox-http_swagshop_htb_-1774485440.state | grep --color=always admin
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

I found one let's check it out

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Nice we found the admin panel. Default creds do not work here

### <mark style="color:blue;">Magento Ver 1.9.0.0 RCE</mark>

### <mark style="color:$primary;">Searchsploit</mark>

<figure><img src="../../.gitbook/assets/image (3405).png" alt=""><figcaption></figcaption></figure>

There is an RCE available I am going to download the script and check it out.

{% code overflow="wrap" expandable="true" %}
```shellscript
searchsploit -m 37977
```
{% endcode %}

The script will create an admin account with username forme and password forme

We have to edit the exploit to fit our target.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

!!!Note -> Copy only the code and remove the comments otherwise you will get this erro

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Nice it worked, and we are able to login

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Reverse Shell via Product Upload</mark>

**From the admin panel, navigate to:**

* Catalog -> Manage Product -> Edit a product
* Custom Options -> add New Option

**For the fields:**

* Title: s`hell`
* Input Type: `File`
* Price: `0.00`
* Allowed File Extensions: `php`

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

After saving, I returned to the product page and uploaded a **PHP reverse shell from:**

[https://www.revshells.com/](https://www.revshells.com/) -> Linux -> PHP Pentest Monkey -> saved as shell.php

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

After uploading your php shell click "add to cart"

We found the `/media` file in directory busting and I remember seeing the custom options

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

my exploit is living there

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

have a listener ready on your desired port than click on php script, you will receive a shell back

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as www-data</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

Interestingly enough the www-data user can run vi as the root user on any files in `/var/www/html/*`

### <mark style="color:blue;">Sudo vi /file -> GTFObins Privesc</mark>

{% embed url="https://gtfobins.org/gtfobins/vi/#shell" %}

<figure><img src="../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

* Open a random file in `/var/www/html` using vi with the sudo privileges

{% code overflow="wrap" expandable="true" %}
```shellscript
sudo /usr/bin/vi /var/www/html/privesc
```
{% endcode %}

* Then inside the terminal type

{% code overflow="wrap" expandable="true" %}
```shellscript
:!/bin/sh
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

it will drop you into a shell as the root user.

