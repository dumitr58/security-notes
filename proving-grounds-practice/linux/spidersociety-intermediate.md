---
icon: ubuntu
---

# SpiderSociety - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.194.214 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 17:35 EDT
Nmap scan report for 192.168.194.214
Host is up (0.033s latency).
Not shown: 55531 filtered tcp ports (no-response), 10001 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f2:5a:a9:66:65:3e:d0:b8:9d:a5:16:8c:e8:16:37:e2 (ECDSA)
|_  256 9b:2d:1d:f8:13:74:ce:96:82:4e:19:35:f9:7e:1b:68 (ED25519)
80/tcp   open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Spider Society
2121/tcp open  ftp     vsftpd 3.0.5
Aggressive OS guesses: Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 40986/tcp)
HOP RTT      ADDRESS
1   31.95 ms 192.168.45.1
2   32.23 ms 192.168.45.254
3   32.27 ms 192.168.251.1
4   32.71 ms 192.168.194.214
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (626).png" alt=""><figcaption></figcaption></figure>

The login button does not work! Nothing else interesting on the site so I will do some directory busting

#### Directory Busting

Small lists will not work in this case, so I used a big one and found something

```
feroxbuster -u http://192.168.194.214 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<figure><img src="../../.gitbook/assets/image (627).png" alt=""><figcaption></figcaption></figure>

the /libspider endpoint led us to a login page

<figure><img src="../../.gitbook/assets/image (630).png" alt=""><figcaption></figcaption></figure>

Default credentials admin:admin work

<figure><img src="../../.gitbook/assets/image (629).png" alt=""><figcaption></figcaption></figure>

Clicking on communications reveals a username and password

<figure><img src="../../.gitbook/assets/image (631).png" alt=""><figcaption></figcaption></figure>

```
ss_ftpbckuser:ss_WeLoveSpiderSociety_From_Tech_Dept5937!
```

### <mark style="color:$primary;">Enumerating FTP Port 2121</mark>

The credentials we found earlier work on ftp

<figure><img src="../../.gitbook/assets/image (632).png" alt=""><figcaption></figcaption></figure>

Found the root directory of the website

<figure><img src="../../.gitbook/assets/image (633).png" alt=""><figcaption></figcaption></figure>

Found a strange file inside the libspider directory

It definitely looked suspicious, but I couldn’t read it with `less` or download it using `get`. I then tried accessing it with `curl` and finally got a response.

<figure><img src="../../.gitbook/assets/image (634).png" alt=""><figcaption></figcaption></figure>

Found more creds

```
spidey:WithGreatPowerComesGreatSecurity99!
```

```
netexec ssh 192.168.194.214 -u spidey -p 'WithGreatPowerComesGreatSecurity99!'
```

<figure><img src="../../.gitbook/assets/image (635).png" alt=""><figcaption></figcaption></figure>

They work over ssh! We can get a shell as spidey

```
ssh spidey@192.168.194.214
```

<figure><img src="../../.gitbook/assets/image (636).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (637).png" alt=""><figcaption></figcaption></figure>

We can restart the spiderbackup.service!

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (638).png" alt=""><figcaption></figcaption></figure>

Linpeas pointed out two custom .service files

<figure><img src="../../.gitbook/assets/image (639).png" alt=""><figcaption></figcaption></figure>

And spiderbackup.service is writable. That’s all I needed to privesc

### <mark style="color:$primary;">Custom writable service file privesc</mark>

<figure><img src="../../.gitbook/assets/image (640).png" alt=""><figcaption></figcaption></figure>

I edited it to set the suid on the /bin/bash

<figure><img src="../../.gitbook/assets/image (641).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (642).png" alt=""><figcaption></figcaption></figure>

Then reloaded and restarted the service

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl restart spiderbackup.service
```

<figure><img src="../../.gitbook/assets/image (643).png" alt=""><figcaption></figcaption></figure>

Now to escalate to root just execute **`/bin/bash -p`**

<figure><img src="../../.gitbook/assets/image (2420).png" alt=""><figcaption></figcaption></figure>
