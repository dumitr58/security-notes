---
icon: ubuntu
---

# LinkVortex - Easy

<figure><img src="../../.gitbook/assets/image.png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/linkvortex"><strong>LinkVortex</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.47 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-03 21:54 EST
Nmap scan report for linkvortex.htb (10.10.11.47)
Host is up (0.048s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
|_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
80/tcp open  http    Apache httpd
|_http-generator: Ghost 5.58
|_http-title: BitByBit Hardware
| http-robots.txt: 4 disallowed entries 
|_/ghost/ /p/ /email/ /r/
|_http-server-header: Apache
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT      ADDRESS
1   99.35 ms 10.10.16.1
2   37.09 ms linkvortex.htb (10.10.11.47)
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

Visiting the page we will get a redirect to linkvortex.htb. I'll add that to my hosts file

```
10.10.11.47	linkvortex.htb
```

<figure><img src="../../.gitbook/assets/image (3027).png" alt=""><figcaption></figcaption></figure>

The posts are made by the admin user.  The page footer reveals that it is powered by Ghost, we can also see that in the nmap scan and the version of Ghost 5.58

<figure><img src="../../.gitbook/assets/image (3025).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">/ghost</mark>

<figure><img src="../../.gitbook/assets/image (3029).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">404 Page</mark>

<figure><img src="../../.gitbook/assets/image (3028).png" alt=""><figcaption></figcaption></figure>

Is custom made to the site

#### <mark style="color:yellow;">HTTP Response Headers</mark>

```shellscript
HTTP/1.1 200 OK
Date: Tue, 06 May 2025 17:48:26 GMT
Server: Apache
X-Powered-By: Express
Cache-Control: public, max-age=0
Content-Type: text/html; charset=utf-8
Content-Length: 12148
ETag: W/"2f74-5/LV/EQ3vUHmemz6InxVzYS/+2w"
Vary: Accept-Encoding
```

Directory busting led me nowhere I will try subdomain enumeration

#### <mark style="color:yellow;">Subdomain Enumeration</mark>

{% code overflow="wrap" %}
```shellscript
wfuzz -c -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u http://linkvortex.htb -H "HOST: FUZZ.linkvortex.htb" --hw 20
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3024).png" alt=""><figcaption></figcaption></figure>

I'll update my hosts file with the new subdomain

```shellscript
10.10.11.47	linkvortex.htb dev.linkvortex.htb
```

#### <mark style="color:yellow;">nmap on dev.linkvortex.htb</mark>

```shellscript
nmap -p 80 -sCV dev.linkvortex.htb
```

```shellscript
## Nmap TCP
nmap -p 80 -sCV dev.linkvortex.htb
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-04 08:51 EST
Nmap scan report for dev.linkvortex.htb (10.10.11.47)
Host is up (0.023s latency).
rDNS record for 10.10.11.47: linkvortex.htb

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd
|_http-server-header: Apache
|_http-title: Launching Soon
| http-git: 
|   10.10.11.47:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|     Remotes:
|_      https://github.com/TryGhost/Ghost.git
```

Based on the remote, it seems to have been cloned from the [**legit Ghost repo**](https://github.com/TryGhost/Ghost.git).

#### <mark style="color:yellow;">dev.linkvortex.htb</mark>

<figure><img src="../../.gitbook/assets/image (3030).png" alt=""><figcaption></figcaption></figure>

The site has a coming soon message

#### <mark style="color:yellow;">HTTP Response Headers</mark>

```shellscript
HTTP/1.1 200 OK
Date: Thu, 04 Dec 2025 13:53:18 GMT
Server: Apache
Last-Modified: Fri, 01 Nov 2024 08:22:52 GMT
ETag: "9ea-625d5a41f4118"
Accept-Ranges: bytes
Content-Length: 2538
Vary: Accept-Encoding
Content-Type: text/html
```

The HTTP response headers do not show Express like the other. The Apache header is the same

#### <mark style="color:yellow;">404 Page</mark>

<figure><img src="../../.gitbook/assets/image (3031).png" alt=""><figcaption></figcaption></figure>

The 404 is the same as the default Apache 404 except that the server and version information is not there

Directory busting finds nothing here either

#### <mark style="color:yellow;">Git Repo</mark>

`nmap` identified a Git repo on `dev.linkvortex.htb`. I’ll use [git-dumper](https://github.com/arthaud/git-dumper) to collect the repo

{% code overflow="wrap" %}
```shellscript
python3 ~/tools/git-dumper/git_dumper.py http://dev.linkvortex.htb dev.linkvortex.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3032).png" alt=""><figcaption></figcaption></figure>

The repo is not currently in a branch, but it does have two modified files awaiting commit. I can see the diff in each of these using `git diff --cached`. The `Dockerfile.ghost` is completely new

```shellscript
git diff --cached Dockerfile.ghost
```

<figure><img src="../../.gitbook/assets/image (3033).png" alt=""><figcaption></figcaption></figure>

This was likely created for LinkVortex. I’ll note that a config file is located at `/var/lib/ghost/config.production.json`

{% code overflow="wrap" %}
```shellscript
git diff --cached ghost/core/test/regression/api/admin/authentication.test.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3034).png" alt=""><figcaption></figcaption></figure>

The diff on `authentication.test.js` shows a password change

This file is a unit test for the Ghost framework. That line is present in the [current Ghost code](https://github.com/TryGhost/Ghost/blob/e44c2117d150f89515f2ea1a7a620f6e678acace/ghost/core/test/regression/api/admin/authentication.test.js#L56) with the “thisissupersafe” password. The new password is likely useful

#### <mark style="color:yellow;">login linkvortex.htb/ghost</mark>

I’ll try these creds at `/ghost`, but they don’t work

<figure><img src="../../.gitbook/assets/image (3036).png" alt=""><figcaption></figcaption></figure>

It says there is not user with that email, address I tried `admin@linkvortex.htb` with the same password, and it worked!

```shellscript
admin@linkvortex.htb:OctopiFociPilfer45
```

<figure><img src="../../.gitbook/assets/image (3037).png" alt=""><figcaption></figcaption></figure>

A quick google search for Ghost version 5.58, yeilds a CVE

<figure><img src="../../.gitbook/assets/image (3038).png" alt=""><figcaption></figcaption></figure>
