---
icon: ubuntu
---

# Amaterasu - Easy TBC

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.249 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 20:24 EDT
Nmap scan report for 192.168.118.249
Host is up (0.030s latency).
Not shown: 65524 filtered tcp ports (no-response)
PORT      STATE  SERVICE          VERSION
21/tcp    open   ftp              vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.158
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp    closed ssh
111/tcp   closed rpcbind
139/tcp   closed netbios-ssn
443/tcp   closed https
445/tcp   closed microsoft-ds
2049/tcp  closed nfs
10000/tcp closed snet-sensor-mgmt
25022/tcp open   ssh              OpenSSH 8.6 (protocol 2.0)
| ssh-hostkey: 
|   256 68:c6:05:e8:dc:f2:9a:2a:78:9b:ee:a1:ae:f6:38:1a (ECDSA)
|_  256 e9:89:cc:c2:17:14:f3:bc:62:21:06:4a:5e:71:80:ce (ED25519)
33414/tcp open   http             Werkzeug httpd 2.2.3 (Python 3.9.13)
|_http-server-header: Werkzeug/2.2.3 Python/3.9.13
|_http-title: 404 Not Found
40080/tcp open   http             Apache httpd 2.4.53 ((Fedora))
|_http-server-header: Apache/2.4.53 (Fedora)
|_http-title: My test page
| http-methods: 
|_  Potentially risky methods: TRACE
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Unix

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   32.08 ms 192.168.45.1
2   32.08 ms 192.168.45.254
3   32.59 ms 192.168.251.1
4   34.01 ms 192.168.118.249

```

### <mark style="color:$primary;">HTTP Port 33414 TCP</mark>

<mark style="color:$primary;">**Directory Busting**</mark>

```
dirsearch -u http://192.168.118.249:33414
```

<figure><img src="../../.gitbook/assets/image (1361).png" alt=""><figcaption></figcaption></figure>

Looks like I found 2 endponts

<figure><img src="../../.gitbook/assets/image (1362).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1363).png" alt=""><figcaption></figcaption></figure>

The **`file-list?dir=/tmp`** endpoint allowed me to view the contents of the /tmp directory.

<figure><img src="../../.gitbook/assets/image (1364).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1365).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1366).png" alt=""><figcaption></figcaption></figure>

So alfredo has&#x20;
