---
icon: ubuntu
---

# Boolean - Intermediate

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.231 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-19 22:29 EDT
Nmap scan report for 192.168.118.231
Host is up (0.028s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 37:80:01:4a:43:86:30:c9:79:e7:fb:7f:3b:a4:1e:dd (RSA)
|   256 b6:18:a1:e1:98:fb:6c:c6:87:55:45:10:c6:d4:45:b9 (ECDSA)
|_  256 ab:8f:2d:e8:a2:04:e7:b7:65:d3:fe:5e:93:1e:03:67 (ED25519)
80/tcp    open   http
| http-title: Boolean
|_Requested resource was http://192.168.118.231/login
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|     HTTP/1.1 400 Bad Request
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Content-Type: text/html; charset=UTF-8
|_    Content-Length: 0
3000/tcp  closed ppp
33017/tcp open   http    Apache httpd 2.4.38 ((Debian))
|_http-title: Development
|_http-server-header: Apache/2.4.38 (Debian)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.95%I=7%D=9/19%Time=68CE11E2%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,55,"HTTP/1\.0\x20403\x20Forbidden\r\nContent-Type:\x20text/html;
SF:\x20charset=UTF-8\r\nContent-Length:\x200\r\n\r\n")%r(HTTPOptions,55,"H
SF:TTP/1\.0\x20403\x20Forbidden\r\nContent-Type:\x20text/html;\x20charset=
SF:UTF-8\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,1C,"HTTP/1\.1\x20
SF:400\x20Bad\x20Request\r\n\r\n")%r(X11Probe,1C,"HTTP/1\.1\x20400\x20Bad\
SF:x20Request\r\n\r\n")%r(FourOhFourRequest,55,"HTTP/1\.0\x20403\x20Forbid
SF:den\r\nContent-Type:\x20text/html;\x20charset=UTF-8\r\nContent-Length:\
SF:x200\r\n\r\n")%r(GenericLines,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:n\r\n")%r(RPCCheck,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(D
SF:NSVersionBindReqTCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(
SF:DNSStatusRequestTCP,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(
SF:Help,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(SSLSessionReq,1
SF:C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TerminalServerCookie,
SF:1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TLSSessionReq,1C,"HT
SF:TP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(Kerberos,1C,"HTTP/1\.1\x20
SF:400\x20Bad\x20Request\r\n\r\n")%r(SMBProgNeg,1C,"HTTP/1\.1\x20400\x20Ba
SF:d\x20Request\r\n\r\n")%r(LPDString,1C,"HTTP/1\.1\x20400\x20Bad\x20Reque
SF:st\r\n\r\n")%r(LDAPSearchReq,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:\r\n")%r(LDAPBindReq,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r
SF:(SIPOptions,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(LANDesk-
SF:RC,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(TerminalServer,1C
SF:,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(NCP,1C,"HTTP/1\.1\x204
SF:00\x20Bad\x20Request\r\n\r\n")%r(NotesRPC,1C,"HTTP/1\.1\x20400\x20Bad\x
SF:20Request\r\n\r\n")%r(JavaRMI,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:n\r\n")%r(WMSRequest,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r
SF:(oracle-tns,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(ms-sql-s
SF:,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(afp,1C,"HTTP/1\.1\x
SF:20400\x20Bad\x20Request\r\n\r\n")%r(giop,1C,"HTTP/1\.1\x20400\x20Bad\x2
SF:0Request\r\n\r\n");
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3000/tcp)
HOP RTT      ADDRESS
1   31.07 ms 192.168.45.1
2   30.77 ms 192.168.45.254
3   32.20 ms 192.168.251.1
4   32.27 ms 192.168.118.231
```

### HTTP Port 80 TCP

<figure><img src="../../.gitbook/assets/image (1603).png" alt=""><figcaption></figcaption></figure>

No luck with simple/default credentials

<figure><img src="../../.gitbook/assets/image (1604).png" alt=""><figcaption></figcaption></figure>

From here I registered an account, to see what the website is about.

<figure><img src="../../.gitbook/assets/image (1594).png" alt=""><figcaption></figcaption></figure>

But I was getting prompted for account login, I am going to open up Burpsuite and Intercept the request and analyze it.

<figure><img src="../../.gitbook/assets/image (1606).png" alt=""><figcaption></figcaption></figure>

It checks to see if the user is confirmed or not, I am going to modify the request and instead of the email variable I will change it to confirmed and set it to true.

<figure><img src="../../.gitbook/assets/image (1613).png" alt=""><figcaption></figcaption></figure>

It worked, we managed to bypass the verification! and got redirected to the file manager!

<figure><img src="../../.gitbook/assets/image (1614).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1615).png" alt=""><figcaption></figcaption></figure>

Playing around with the url, I discovered path Traversal by mistake.

### Path Traversal&#x20;

changing the current working directory to /etc using directory traversal and change the download file to passwd

<figure><img src="../../.gitbook/assets/image (1595).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1596).png" alt=""><figcaption></figcaption></figure>

**Navigate to remi’s .ssh directory:**

<figure><img src="../../.gitbook/assets/image (1597).png" alt=""><figcaption></figcaption></figure>

I generated a keypair and upload the public key as an authorized\_keys in her .ssh directory:

```
ssh-keygen -t rsa -b 2048 -f ./id_rsa -N ""
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
```

<figure><img src="../../.gitbook/assets/image (1598).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1599).png" alt=""><figcaption></figcaption></figure>

```
ssh -i id_rsa remi@192.168.118.231
```

<figure><img src="../../.gitbook/assets/image (1600).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Linpeas

<figure><img src="../../.gitbook/assets/image (1601).png" alt=""><figcaption></figcaption></figure>

Linpeas reveals root's ssh key stored in remi's .ssh folder, I am going to use it to SSH as root

<figure><img src="../../.gitbook/assets/image (1602).png" alt=""><figcaption></figcaption></figure>
