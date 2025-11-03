---
icon: ubuntu
---

# PC - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.161.210 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 18:54 EDT
Nmap scan report for 192.168.161.210
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
8000/tcp open  http    ttyd 1.7.3-a2312cb (libwebsockets 3.2.0)
|_http-title: ttyd - Terminal
|_http-server-header: ttyd/1.7.3-a2312cb (libwebsockets/3.2.0)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   27.39 ms 192.168.45.1
2   27.28 ms 192.168.45.254
3   27.92 ms 192.168.251.1
4   28.25 ms 192.168.161.210
```

### <mark style="color:$primary;">HTTP Port 8000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (936).png" alt=""><figcaption></figcaption></figure>

We have access to a webshell console? I guess I'll just go straight into trying to esclate my privileges

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (937).png" alt=""><figcaption></figcaption></figure>

Googling about libwebsockets 3.2.0 I came a cross an Unauthenticated RCE for rpc.py server on Github: [https://github.com/ehtec/rpcpy-exploit/tree/main](https://github.com/ehtec/rpcpy-exploit/tree/main)

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (939).png" alt=""><figcaption></figcaption></figure>

the root user is running rpc.py, I can use the exploit I just found to escalate my privileges.&#x20;

### <mark style="color:$primary;">Libwebsockets 3.2.0 Unauthenticated RCE</mark>

```
git clone https://github.com/ehtec/rpcpy-exploit.git
```

I am going to make a couple of changes to the exploit:

<figure><img src="../../.gitbook/assets/image (940).png" alt=""><figcaption></figcaption></figure>

I am going to have it set the SUID bit on bash so we can escalate our privileges to root.

Now all that is left is to download and run the script.

<figure><img src="../../.gitbook/assets/image (941).png" alt=""><figcaption></figcaption></figure>

Ok we are root, This has to be the easiest machine so far!
