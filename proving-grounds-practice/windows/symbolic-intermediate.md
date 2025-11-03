---
icon: windows
---

# Symbolic - Intermediate

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.131.177 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-22 19:51 EDT
Nmap scan report for 192.168.131.177
Host is up (0.028s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:40:e2:ef:21:ea:c1:77:b6:14:a3:f7:04:59:45:28 (RSA)
|   256 f8:fb:e3:c6:16:3a:e2:62:d0:e2:ae:d4:f2:9e:6f:6d (ECDSA)
|_  256 94:5e:97:ad:f9:0f:81:b6:6b:3b:bd:98:43:c0:0d:6a (ED25519)
80/tcp open  http    Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
|_http-title: WebPage to PDF
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   29.05 ms 192.168.45.1
2   25.44 ms 192.168.45.254
3   25.47 ms 192.168.251.1
4   26.29 ms 192.168.131.177
```

### HTTP Port 80

<figure><img src="../../.gitbook/assets/image (2027).png" alt=""><figcaption></figcaption></figure>

A PDF converter website taking a url! I think p4yl0ad is a user. I am going to try hosting a HTML file and load it into the application and check to see if it gets executed.

### File Read Vulnerability

<figure><img src="../../.gitbook/assets/image (2028).png" alt=""><figcaption></figcaption></figure>

I am going to host using a simple python http server:

<figure><img src="../../.gitbook/assets/image (2029).png" alt=""><figcaption></figcaption></figure>

Before I upload the file I am going to start Burpsuite Intercept the request and check to see if the location is being leaked in the response.&#x20;

<figure><img src="../../.gitbook/assets/image (2030).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2031).png" alt=""><figcaption></figcaption></figure>

I did not really have to do this, the website automatically redirected us to the location:

<figure><img src="../../.gitbook/assets/image (2032).png" alt=""><figcaption></figcaption></figure>

Ok I am going to try and see if we can view any files on the system. I am going to add an iframe to my script and use the src function to get a file from the system:

<figure><img src="../../.gitbook/assets/image (2035).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2036).png" alt=""><figcaption></figcaption></figure>

It works! Okay we know that this box only has 2 ports, and on Linux boxes whenever I see ssh and http open with a file read vulnerability I check for ssh keys. I am going to do the same here, I am going to try and see if p4yl0ad is actually a user on this box and try to get his ssh key:

```
<html>
<body>
	<h1>
		<iframe src="C:/Users/p4yl0ad/.ssh/id_rsa" height=1200 width=1200>
		
		</iframe>
	</h1>
</body>
</html>
```

<figure><img src="../../.gitbook/assets/image (2037).png" alt=""><figcaption></figcaption></figure>

It worked!!!!&#x20;

<figure><img src="../../.gitbook/assets/image (2038).png" alt=""><figcaption></figcaption></figure>

I am going to see it and use it to login as p4yl0ad

<figure><img src="../../.gitbook/assets/image (2039).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2040).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

<figure><img src="../../.gitbook/assets/image (2041).png" alt=""><figcaption></figcaption></figure>

There is an interesting backup folder at the root directory

<figure><img src="../../.gitbook/assets/image (2042).png" alt=""><figcaption></figcaption></figure>

The `backup.ps1` file backs up the `C:\xampp\htdocs\logs\request.log` file to the `C:\backup\logs` directory and use date format as the backup log file name.

I am going to try and create a symbolic link, that links `request.log` to some admin file like their OpenSSH private\_key, then run the `backup.ps1` again. Then, we should be able to see the private key through the logs backup.

I am going to check what privileges I have on the file first

<figure><img src="../../.gitbook/assets/image (2043).png" alt=""><figcaption></figcaption></figure>

I have fullcontrol sweet!

<figure><img src="../../.gitbook/assets/image (2045).png" alt=""><figcaption></figcaption></figure>

I need admin privileges to create a symlink :hushed:

I have an idea! I am going to try and extract the admin ssh key the same way i extracted p4yl0ad ssh key, using the file read vulnerability.

<figure><img src="../../.gitbook/assets/image (2050).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2048).png" alt=""><figcaption></figcaption></figure>

it worked :hushed:

<figure><img src="../../.gitbook/assets/image (2049).png" alt=""><figcaption></figcaption></figure>

I don't think this was the intended route for privesc, but we made it work :smile:

After some googling I came across this method:

To do this without administrator rights, we need to create a `Mount Point` such that **`C:\xampp\htdocs\`** points to **`\RPC Control\`** object directory. We then create a Symlink such that **`\RPC Control\logs`** points to **`\?\C:\Users\Administrator.ssh\id_rsa`**.

We can do this by using [symboliclink-testing-tools](https://github.com/googleprojectzero/symboliclink-testing-tools/releases/download/v1.0/Release.7z), a tool for testing various symbolic link types of Windows. We need to save the 7zip file to our Kali machine and the unzip it.

We need to move the `CreateSymlink.exe` file to the target machine and use it to create the symlink.
