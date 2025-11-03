---
icon: ubuntu
---

# Pelican - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.159.98 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-03 02:54 EDT
Nmap scan report for 192.168.159.98
Host is up (0.030s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
631/tcp   open  ipp         CUPS 2.2
|_http-title: Forbidden - CUPS v2.2.10
|_http-server-header: CUPS/2.2 IPP/2.1
| http-methods: 
|_  Potentially risky methods: PUT
2181/tcp  open  zookeeper   Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
8080/tcp  open  http        Jetty 1.0
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(1.0)
8081/tcp  open  http        nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://192.168.159.98:8080/exhibitor/v1/ui/index.html
34051/tcp open  java-rmi    Java RMI
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -6h40m01s, deviation: 2h18m34s, median: -8h00m02s
| smb2-time: 
|   date: 2025-10-02T22:55:20
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: pelican
|   NetBIOS computer name: PELICAN\x00
|   Domain name: \x00
|   FQDN: pelican
|_  System time: 2025-10-02T18:55:19-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

TRACEROUTE (using port 110/tcp)
HOP RTT      ADDRESS
1   32.87 ms 192.168.45.1
2   27.80 ms 192.168.45.254
3   27.98 ms 192.168.251.1
4   28.18 ms 192.168.159.98
```

### <mark style="color:$primary;">HTTP Port 8081 TCP</mark>

Upong visiting port 8081 we get a redirect&#x20;

<figure><img src="../../.gitbook/assets/image (877).png" alt=""><figcaption></figcaption></figure>

We get a version and itlooks like a cronjob might be running in the background

### <mark style="color:$primary;">Exhibitor v1.x/1.7.1 RCE</mark>

I found a RCE exploit for Exhibitor at this Github [**repo**](https://github.com/thehunt1s0n/Exihibitor-RCE?tab=readme-ov-file) that works for this verion as well

```
git clone https://github.com/thehunt1s0n/Exihibitor-RCE.git
```

Now to get a reverse shell simply follow the below commands:

<figure><img src="../../.gitbook/assets/image (878).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (879).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Sudo gcore</mark>

[**GTFObins**](https://gtfobins.github.io/gtfobins/gcore/#sudo) **has** a privesc available fot gcore

<figure><img src="../../.gitbook/assets/image (880).png" alt=""><figcaption></figcaption></figure>

We need to find some interesting processes to checkout for some interesting info!

```
ps auxwww
```

Honestly ps auxwww is not colorful and makes it hard to look at. I'll get linpeas on the system and check it out over there

<figure><img src="../../.gitbook/assets/image (881).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$info;">**/usr/bin/password-store**</mark> sounds interesting let's take a lookt at that one!

```
sudo gcore 494
```

<figure><img src="../../.gitbook/assets/image (882).png" alt=""><figcaption></figcaption></figure>

I ran strings on it and checked for possible credentials we can make use of

<figure><img src="../../.gitbook/assets/image (884).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (883).png" alt=""><figcaption></figcaption></figure>

And we found it, let's try it out

<figure><img src="../../.gitbook/assets/image (885).png" alt=""><figcaption></figcaption></figure>
