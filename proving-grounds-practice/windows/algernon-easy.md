---
icon: windows
---

# Algernon - Easy

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.65 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-23 14:23 EDT
Nmap scan report for 192.168.118.65
Host is up (0.027s latency).
Not shown: 65520 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 04-29-20  10:31PM       <DIR>          ImapRetrieval
| 09-23-25  11:22AM       <DIR>          Logs
| 04-29-20  10:31PM       <DIR>          PopRetrieval
|_04-29-20  10:32PM       <DIR>          Spool
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
9998/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-title: Site doesn't have a title (text/html; charset=utf-8).                                                                                 
|_Requested resource was /interface/root                                                                                                            
|_http-server-header: Microsoft-IIS/10.0                                                                                                            
| uptime-agent-info: HTTP/1.1 400 Bad Request\x0D                                                                                                   
| Content-Type: text/html; charset=us-ascii\x0D                                                                                                     
| Server: Microsoft-HTTPAPI/2.0\x0D                                                                                                                 
| Date: Tue, 23 Sep 2025 18:27:07 GMT\x0D                                                                                                           
| Connection: close\x0D                                                                                                                             
| Content-Length: 326\x0D                                                                                                                           
| \x0D                                                                                                                                              
| <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">\x0D                                                     
| <HTML><HEAD><TITLE>Bad Request</TITLE>\x0D                                                                                                        
| <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>\x0D                                                                 
| <BODY><h2>Bad Request - Invalid Verb</h2>\x0D                                                                                                     
| <hr><p>HTTP Error 400. The request verb is invalid.</p>\x0D                                                                                       
|_</BODY></HTML>\x0D                                                                                                                                
17001/tcp open  remoting      MS .NET Remoting services                                                                                             
49664/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49665/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49666/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49667/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49668/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
49669/tcp open  msrpc         Microsoft Windows RPC                                                                                                 
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).                                                 
TCP/IP fingerprint:                                                                                                                                 
OS:SCAN(V=7.95%E=4%D=9/23%OT=21%CT=1%CU=39592%PV=Y%DS=4%DC=T%G=Y%TM=68D2E68                                                                         
OS:E%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=108%TI=I%CI=I%TS=U)SEQ(SP=1                                                                         
OS:04%GCD=1%ISR=10F%TI=I%CI=I%TS=U)SEQ(SP=FB%GCD=1%ISR=10A%TI=I%CI=I%TS=U)S                                                                         
OS:EQ(SP=FE%GCD=1%ISR=105%TI=I%CI=I%TS=U)SEQ(SP=FF%GCD=1%ISR=10D%TI=I%CI=I%                                                                         
OS:TS=U)OPS(O1=M578NW8NNS%O2=M578NW8NNS%O3=M578NW8%O4=M578NW8NNS%O5=M578NW8                                                                         
OS:NNS%O6=M578NNS)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R                                                                         
OS:=Y%DF=Y%T=80%W=FFFF%O=M578NW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%                                                                         
OS:RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y                                                                         
OS:%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R                                                                         
OS:%O=%RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RU                                                                         
OS:CK=G%RUD=G)IE(R=N)                                                                                                                               
                                                                                                                                                    
Network Distance: 4 hops                                                                                                                            
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows                                                                                            
                                                                                                                                                    
Host script results:                                                                                                                                
| smb2-time: 
|   date: 2025-09-23T18:27:11
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 554/tcp)
HOP RTT      ADDRESS
1   26.34 ms 192.168.45.1
2   26.34 ms 192.168.45.254
3   26.41 ms 192.168.251.1
4   29.41 ms 192.168.118.65
```

### FTP Anonymous login

I am just going to download everything and look at it on my machine.

```
wget -r ftp://Anonymous:pass@192.168.118.65
```

```
cd Logs
cat *
```

I did find a username  in the logs and that there is webmail somewhere

<figure><img src="../../.gitbook/assets/image (1461).png" alt=""><figcaption></figcaption></figure>

### HTTP Port 9998&#x20;

<figure><img src="../../.gitbook/assets/image (1462).png" alt=""><figcaption></figcaption></figure>

Visiting the Site we are greeted by SmarterMail. I am going to do a quick search on searchsploit for SmarterMail even though I do not have  a version for it we might find a low haning fruit. Worth a try

### SmarterMail Build 6985 RCE

<figure><img src="../../.gitbook/assets/image (1463).png" alt=""><figcaption></figcaption></figure>

There is a python script that promises Remote Code Execution. I am going to download it and take a look at it:

```
searchsploit -m 49216
```

<figure><img src="../../.gitbook/assets/image (1464).png" alt=""><figcaption></figcaption></figure>

We need to update our ip and port of the target machine and our attack machine in the script.

<figure><img src="../../.gitbook/assets/image (1467).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1468).png" alt=""><figcaption></figcaption></figure>

