---
icon: ubuntu
---

# Tico - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.143 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 20:02 EDT
Nmap scan report for 192.168.231.143
Host is up (0.029s latency).
Not shown: 65428 filtered tcp ports (no-response), 101 closed tcp ports (reset)
PORT      STATE SERVICE    VERSION
21/tcp    open  ftp        vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Feb 01  2021 pub
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh        OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 85:35:fb:ca:b3:4b:30:d8:e5:8e:b3:25:58:6c:6e:70 (RSA)
|   256 de:67:a2:32:d5:ff:56:6e:82:5b:6a:17:7d:e2:44:ac (ECDSA)
|_  256 3a:a3:20:3b:32:cd:83:6f:dc:23:a2:66:f9:0f:c6:d3 (ED25519)
80/tcp    open  http       nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Markdown Editor
8080/tcp  open  http-proxy
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     X-DNS-Prefetch-Control: off
|     X-Frame-Options: SAMEORIGIN
|     X-Download-Options: noopen
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Referrer-Policy: strict-origin-when-cross-origin
|     X-Powered-By: NodeBB
|     set-cookie: _csrf=IHuUToz_pDmHgA_D-rMqUYhA; Path=/
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 15431
|     ETag: W/"3c47-AscUa5hyYXA2Xlv6ZN5wnI4jHJs"
|     Vary: Accept-Encoding
|     Date: Tue, 07 Oct 2025 00:04:10 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en-GB" data-dir="ltr" style="direction: ltr;" >
|     <head>
|     <title>Not Found | NodeBB</title>
|     <meta name="viewport" content="width&#x3D;device-width, initial-scale&#x3D;1.0" />
|     <meta name="content-type" content="text/html; charset=UTF-8" />
|     <meta name="apple-mobile-web-app-capable" content="yes" />
|     <meta name="mobile-web-app-capable" content="yes" />
|     <meta property="og:site_name"
|   GetRequest: 
|     HTTP/1.1 200 OK
|     X-DNS-Prefetch-Control: off
|     X-Frame-Options: SAMEORIGIN
|     X-Download-Options: noopen
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Referrer-Policy: strict-origin-when-cross-origin
|     X-Powered-By: NodeBB
|     set-cookie: _csrf=CEiqVyHTatIvASL3a2vYe7ce; Path=/
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 24233
|     ETag: W/"5ea9-V0zveZehyRHT1WtNZaePelyutJc"
|     Vary: Accept-Encoding
|     Date: Tue, 07 Oct 2025 00:04:10 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en-GB" data-dir="ltr" style="direction: ltr;" >
|     <head>
|     <title>Home | NodeBB</title>
|     <meta name="viewport" content="width&#x3D;device-width, initial-scale&#x3D;1.0" />
|     <meta name="content-type" content="text/html; charset=UTF-8" />
|     <meta name="apple-mobile-web-app-capable" content="yes" />
|     <meta name="mobile-web-app-capable" content="yes" />
|     <meta property="og:site_name" content="No
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     X-DNS-Prefetch-Control: off
|     X-Frame-Options: SAMEORIGIN
|     X-Download-Options: noopen
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Referrer-Policy: strict-origin-when-cross-origin
|     X-Powered-By: NodeBB
|     Allow: GET,HEAD
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 8
|     ETag: W/"8-ZRAf8oNBS3Bjb/SU2GYZCmbtmXg"
|     Vary: Accept-Encoding
|     Date: Tue, 07 Oct 2025 00:04:10 GMT
|     Connection: close
|     GET,HEAD
|   RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|_    Connection: close
| http-robots.txt: 3 disallowed entries 
|_/admin/ /reset/ /compose
|_http-title: Home | NodeBB
11211/tcp open  memcached  Memcached 1.5.6 (uptime 268 seconds; Ubuntu)
27017/tcp open  mongodb    MongoDB 4.1.1 - 5.0
| mongodb-info: 
|   MongoDB Build info
|     maxBsonObjectSize = 16777216
|     allocator = tcmalloc
|     buildEnvironment
|       cxx = /opt/mongodbtoolchain/v2/bin/g++: g++ (GCC) 5.4.0
|       target_arch = x86_64
|       distmod = ubuntu1804
|       target_os = linux
|       ccflags = -fno-omit-frame-pointer -fno-strict-aliasing -ggdb -pthread -Wall -Wsign-compare -Wno-unknown-pragmas -Winvalid-pch -Werror -O2 -Wno-unused-local-typedefs -Wno-unused-function -Wno-deprecated-declarations -Wno-unused-but-set-variable -Wno-missing-braces -fstack-protector-strong -fno-builtin-memcmp
|       distarch = x86_64
|       cxxflags = -Woverloaded-virtual -Wno-maybe-uninitialized -std=c++14
|       linkflags = -pthread -Wl,-z,now -rdynamic -Wl,--fatal-warnings -fstack-protector-strong -fuse-ld=gold -Wl,--build-id -Wl,--hash-style=gnu -Wl,-z,noexecstack -Wl,--warn-execstack -Wl,-z,relro
|       cc = /opt/mongodbtoolchain/v2/bin/gcc: gcc (GCC) 5.4.0
|     openssl
|       running = OpenSSL 1.1.1  11 Sep 2018
|       compiled = OpenSSL 1.1.1  11 Sep 2018
|     versionArray
|       0 = 4
|       3 = 0
|       2 = 22
|       1 = 0
|     gitVersion = 1741806fb46c161a1d42870f6e98f5100d196315
|     debug = false
|     storageEngines
|       0 = devnull
|       3 = wiredTiger
|       2 = mmapv1
|       1 = ephemeralForTest
|     ok = 1.0
|     bits = 64
|     modules
|     javascriptEngine = mozjs
|     sysInfo = deprecated
|     version = 4.0.22
|   Server status
|     ok = 0.0
|     code = 13
|     errmsg = command serverStatus requires authentication
|_    codeName = Unauthorized
| mongodb-databases: 
|   ok = 0.0
|   code = 13
|   errmsg = command listDatabases requires authentication
|_  codeName = Unauthorized
```

### <mark style="color:$primary;">FTP Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (410).png" alt=""><figcaption></figcaption></figure>

Anonymous login reveals a .pcap file. I opened it in wireshark, but was unable in finding anything usefull

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (411).png" alt=""><figcaption></figcaption></figure>

There does not seem to be anything here either, directory busting did not offer any other avenues either

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

<figure><img src="../../.gitbook/assets/image (412).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (413).png" alt=""><figcaption></figcaption></figure>

One exploit fits our description

### <mark style="color:$primary;">NodeBB Forum 1.12.2-1.14.2 Account Takeover</mark>

```
searchsploit -m 48875
```

<figure><img src="../../.gitbook/assets/image (414).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Step 1 -> Create a user**</mark>

<figure><img src="../../.gitbook/assets/image (415).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Step 2 -> Edit Profile -> Change Password -> Intercept request with Bursuite**</mark>

<figure><img src="../../.gitbook/assets/image (417).png" alt=""><figcaption></figcaption></figure>

&#x20;<mark style="color:yellow;">**Step 3 -> Replace the uid value with '1' than forward the request.**</mark>

<figure><img src="../../.gitbook/assets/image (403).png" alt=""><figcaption></figcaption></figure>

We can now login as the admin use with the new password we set

```
admin:password123!
```

<figure><img src="../../.gitbook/assets/image (404).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (405).png" alt=""><figcaption></figcaption></figure>

on the admin dashboard one plugin stood out Emoji! We found a Arbitrary File Write exploit earlier on searchsploit. I am going to download it

### <mark style="color:$primary;">NodeBB Plugin Emoji 3.2.1 - Arbitrary File Write</mark>

The exploit write our SSH public key into the `authorized_keys`  folder of the root user.

We need to change a couple of variable before executing it

<figure><img src="../../.gitbook/assets/image (407).png" alt=""><figcaption></figcaption></figure>

I've generate a new public key for this instance&#x20;

<figure><img src="../../.gitbook/assets/image (408).png" alt=""><figcaption></figcaption></figure>

Happy root!
