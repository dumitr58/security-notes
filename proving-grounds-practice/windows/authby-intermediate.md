---
icon: windows
---

# AuthBy - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.46 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 08:37 EDT
Nmap scan report for 192.168.224.46
Host is up (0.034s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           zFTPServer 6.0 build 2011-10-17
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| total 9680
| ----------   1 root     root      5610496 Oct 18  2011 zFTPServer.exe
| ----------   1 root     root           25 Feb 10  2011 UninstallService.bat
| ----------   1 root     root      4284928 Oct 18  2011 Uninstall.exe
| ----------   1 root     root           17 Aug 13  2011 StopService.bat
| ----------   1 root     root           18 Aug 13  2011 StartService.bat
| ----------   1 root     root         8736 Nov 09  2011 Settings.ini
| dr-xr-xr-x   1 root     root          512 Oct 04 19:38 log
| ----------   1 root     root         2275 Aug 08  2011 LICENSE.htm
| ----------   1 root     root           23 Feb 10  2011 InstallService.bat
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 extensions
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 certificates
|_dr-xr-xr-x   1 root     root          512 Aug 02  2024 accounts
242/tcp  open  http          Apache httpd 2.2.21 ((Win32) PHP/5.3.8)
|_http-server-header: Apache/2.2.21 (Win32) PHP/5.3.8
|_http-title: 401 Authorization Required
| http-auth: 
| HTTP/1.1 401 Authorization Required\x0D
|_  Basic realm=Qui e nuce nuculeum esse volt, frangit nucem!
3145/tcp open  zftp-admin    zFTPServer admin
3389/tcp open  ms-wbt-server Microsoft Terminal Service
| ssl-cert: Subject: commonName=LIVDA                                                                                                               
| Not valid before: 2024-08-01T10:50:21                                                                                                             
|_Not valid after:  2025-01-31T10:50:21                                                                                                             
|_ssl-date: 2025-10-04T12:39:21+00:00; +3s from scanner time.                                                                                       
| rdp-ntlm-info:                                                                                                                                    
|   Target_Name: LIVDA                                                                                                                              
|   NetBIOS_Domain_Name: LIVDA                                                                                                                      
|   NetBIOS_Computer_Name: LIVDA                                                                                                                    
|   DNS_Domain_Name: LIVDA                                                                                                                          
|   DNS_Computer_Name: LIVDA                                                                                                                        
|   Product_Version: 6.0.6001                                                                                                                       
|_  System_Time: 2025-10-04T12:39:16+00:00                                                                                                          
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port                                               
Device type: general purpose|phone                                                                                                                  
Running (JUST GUESSING): Microsoft Windows 2008|7|8.1|Phone|Vista (96%)                                                                             
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_vista                                                                                                              
Aggressive OS guesses: Microsoft Windows Server 2008 R2 or Windows 7 SP1 (96%), Microsoft Windows 7 or Windows Server 2008 R2 (92%), Microsoft Windows 7 SP1 or Windows Server 2008 R2 or Windows 8.1 (89%), Microsoft Windows Server 2008 R2 SP1 (88%), Microsoft Windows Server 2008 (87%), Microsoft Windows Server 2008 R2 or Windows 8 (87%), Microsoft Windows 8.1 Update 1 (87%), Microsoft Windows Phone 7.5 or 8.0 (87%), Microsoft Windows Vista or Windows 7 (87%)                                                                                                                                   
No exact OS matches for host (test conditions non-ideal).                                                                                           
Network Distance: 4 hops                                                                                                                            
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows                                                                                            
                                                                                                                                                    
Host script results:                                                                                                                                
|_clock-skew: mean: 2s, deviation: 0s, median: 2s                                                                                                   
                                                                                                                                                    
TRACEROUTE (using port 21/tcp)                                                                                                                      
HOP RTT      ADDRESS                                                                                                                                
1   37.27 ms 192.168.45.1                                                                                                                           
2   32.57 ms 192.168.45.254                                                                                                                         
3   34.05 ms 192.168.251.1                                                                                                                          
4   37.19 ms 192.168.224.46
```

### <mark style="color:$primary;">FTP Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (807).png" alt=""><figcaption></figcaption></figure>

The accounts folder revealed 3 users we cannot download these files. I'll try default admin admin and see if I can login as the admin user

<figure><img src="../../.gitbook/assets/image (808).png" alt=""><figcaption></figcaption></figure>

it worked! I am going to download these files.

<figure><img src="../../.gitbook/assets/image (814).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (809).png" alt=""><figcaption></figcaption></figure>

The .htpasswd file offers us some credentials.&#x20;

### <mark style="color:$primary;">Cracking Hash</mark>

```
john offsec.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (810).png" alt=""><figcaption></figcaption></figure>

We got some credentials now.

### <mark style="color:$primary;">HTTP Port 242 TCP</mark>

<figure><img src="../../.gitbook/assets/image (811).png" alt=""><figcaption></figcaption></figure>

The credentials we found earlier work here!

<figure><img src="../../.gitbook/assets/image (812).png" alt=""><figcaption></figcaption></figure>

This is the .htacces we found on the ftp location as admin this is where the website is stored

If we can place a reverse php shell there, we will be able to access it!

I am going to use [https://www.revshells.com/](https://www.revshells.com/) to generate a php reverse shell

<figure><img src="../../.gitbook/assets/image (815).png" alt=""><figcaption></figcaption></figure>

Saved it as reverse.php then uploaded it over ftp using admin login

<figure><img src="../../.gitbook/assets/image (816).png" alt=""><figcaption></figcaption></figure>

Now let's see if I get a reverse shell when I access it. Make sure you have a listener ready

<figure><img src="../../.gitbook/assets/image (817).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (818).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (819).png" alt=""><figcaption></figcaption></figure>

We can use a potato Attack to escalate our privileges to the admin user or

<figure><img src="../../.gitbook/assets/image (821).png" alt=""><figcaption></figcaption></figure>

We could go for a version exploit! I'll try the version Exploit

### <mark style="color:$primary;">MS11-046 Windows Local Privesc</mark>

<figure><img src="../../.gitbook/assets/image (822).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (823).png" alt=""><figcaption></figcaption></figure>

I'll download it to my machine and compile it:

```
searchsploit -m 40564
```

```
i686-w64-mingw32-gcc -o 40564.exe 40564.c -lws2_32
```

<figure><img src="../../.gitbook/assets/image (824).png" alt=""><figcaption></figcaption></figure>

Note certutil and wget are not on the machine. I am going to upload the exploit in the ftp folder and access it there

<figure><img src="../../.gitbook/assets/image (825).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (826).png" alt=""><figcaption></figcaption></figure>

Now just simply run the exploit&#x20;

<figure><img src="../../.gitbook/assets/image (827).png" alt=""><figcaption></figcaption></figure>
