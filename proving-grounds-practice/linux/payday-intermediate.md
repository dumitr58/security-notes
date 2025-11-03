---
icon: ubuntu
---

# PayDay - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.161.39 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 17:35 EDT
Nmap scan report for 192.168.161.39
Host is up (0.030s latency).
Not shown: 65527 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
| ssh-hostkey: 
|   1024 f3:6e:87:04:ea:2d:b3:60:ff:42:ad:26:67:17:94:d5 (DSA)
|_  2048 bb:03:ce:ed:13:f1:9a:9e:36:03:e2:af:ca:b2:35:04 (RSA)
80/tcp  open  http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
|_http-server-header: Apache/2.2.4 (Ubuntu) PHP/5.2.3-1ubuntu6
|_http-title: CS-Cart. Powerful PHP shopping cart software
110/tcp open  pop3        Dovecot pop3d
|_ssl-date: 2025-10-01T21:36:33+00:00; +8s from scanner time.
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_pop3-capabilities: TOP UIDL STLS RESP-CODES PIPELINING SASL CAPA
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
143/tcp open  imap        Dovecot imapd
|_ssl-date: 2025-10-01T21:36:33+00:00; +8s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
|_imap-capabilities: completed LOGIN-REFERRALS LITERAL+ Capability IMAP4rev1 UNSELECT OK SASL-IR THREAD=REFERENCES SORT IDLE LOGINDISABLEDA0001 CHILDREN MULTIAPPEND STARTTLS NAMESPACE
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
445/tcp open  netbios-ssn Samba smbd 3.0.26a (workgroup: MSHOME)
993/tcp open  ssl/imap    Dovecot imapd
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_ssl-date: 2025-10-01T21:36:32+00:00; +7s from scanner time.
|_imap-capabilities: LOGIN-REFERRALS LITERAL+ completed IMAP4rev1 UNSELECT Capability SASL-IR THREAD=REFERENCES SORT IDLE OK CHILDREN AUTH=PLAINA0001 MULTIAPPEND NAMESPACE
995/tcp open  ssl/pop3    Dovecot pop3d
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
|_ssl-date: 2025-10-01T21:36:32+00:00; +7s from scanner time.
| ssl-cert: Subject: commonName=ubuntu01/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2008-04-25T02:02:48
|_Not valid after:  2008-05-25T02:02:48
|_pop3-capabilities: TOP UIDL USER RESP-CODES PIPELINING SASL(PLAIN) CAPA
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/1%OT=22%CT=1%CU=31790%PV=Y%DS=4%DC=T%G=Y%TM=68DD9ED
OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=C6%GCD=1%ISR=EF%TI=Z%CI=Z%II=I%TS=7)SEQ(S
OS:P=CF%GCD=1%ISR=EF%TI=Z%CI=Z%II=I%TS=7)SEQ(SP=D0%GCD=1%ISR=EF%TI=Z%CI=Z%I
OS:I=I%TS=7)SEQ(SP=D5%GCD=1%ISR=EF%TI=Z%CI=Z%II=I%TS=7)SEQ(SP=D9%GCD=1%ISR=
OS:EF%TI=Z%CI=Z%II=I%TS=7)OPS(O1=M578ST11NW5%O2=M578ST11NW5%O3=M578NNT11NW5
OS:%O4=M578ST11NW5%O5=M578ST11NW5%O6=M578ST11)WIN(W1=16A0%W2=16A0%W3=16A0%W
OS:4=16A0%W5=16A0%W6=16A0)ECN(R=Y%DF=Y%T=40%W=16D0%O=M578NNSNW5%CC=N%Q=)T1(
OS:R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S
OS:=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R
OS:=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=40%IPL=164%
OS:UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: PAYDAY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.26a)
|   Computer name: payday
|   NetBIOS computer name: 
|   Domain name: 
|   FQDN: payday
|_  System time: 2025-10-01T17:36:30-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 40m07s, deviation: 1h37m58s, median: 6s

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   29.01 ms 192.168.45.1
2   28.84 ms 192.168.45.254
3   29.29 ms 192.168.251.1
4   29.83 ms 192.168.161.39
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2308).png" alt=""><figcaption></figcaption></figure>

Default admin:admin creds work to login

<figure><img src="../../.gitbook/assets/image (2309).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2310).png" alt=""><figcaption></figcaption></figure>

We got a version for the website! I am going to look for an exploit

<figure><img src="../../.gitbook/assets/image (2311).png" alt=""><figcaption></figcaption></figure>

There is an authenticated RCE we already have creds. I am going to download it and check out the steps!

### <mark style="color:$primary;">CS-CART 1.3.3 RCE \[Authenticated]</mark>

```
subl 48891.txt
```

<figure><img src="../../.gitbook/assets/image (2312).png" alt=""><figcaption></figcaption></figure>

It looks like a simple file upload vulnerability, I found a more detailed guide [**here**](https://gist.github.com/momenbasel/ccb91523f86714edb96c871d4cf1d05c)

<figure><img src="../../.gitbook/assets/image (2313).png" alt=""><figcaption></figcaption></figure>

I used pentest monkeys reverse php, I saved it as reverse.phtml

Upload your reverse shell here

<figure><img src="../../.gitbook/assets/image (944).png" alt=""><figcaption></figcaption></figure>

After that make sure you have a listener ready than visit this url:

<figure><img src="../../.gitbook/assets/image (945).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (946).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (951).png" alt=""><figcaption></figcaption></figure>

I'll keep patrick in mind while enumerating

<figure><img src="../../.gitbook/assets/image (947).png" alt=""><figcaption></figcaption></figure>

We have access to the root directory and there is a .cap file there I am going to download it to my machine and inspect it using wireshark.

```
impacket-smbserver share . -smb2support
```

<figure><img src="../../.gitbook/assets/image (949).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (948).png" alt=""><figcaption></figcaption></figure>

Now I am going to load the file in wireshark and check it out

<figure><img src="../../.gitbook/assets/image (950).png" alt=""><figcaption></figcaption></figure>

The pcap file contains creds for brett logging in via ftp. Unfortunately brett is not on the system and the password doesn’t work for any other user.  What a rabbit hole :rofl:&#x20;

I checked for poor password policy and found patrick:patrick worked

<figure><img src="../../.gitbook/assets/image (952).png" alt=""><figcaption></figcaption></figure>

Patrick can run any commands as sudo! Let's go get root!

<figure><img src="../../.gitbook/assets/image (953).png" alt=""><figcaption></figcaption></figure>

We got root!
