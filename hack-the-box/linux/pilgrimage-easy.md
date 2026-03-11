---
icon: ubuntu
---

# Pilgrimage - Easy

<figure><img src="../../.gitbook/assets/image (3277).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/pilgrimage"><strong>Pilgrimage</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" overflow="wrap" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.10.163 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-05 17:36 -0500
Nmap scan report for 10.129.10.163
Host is up (0.048s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
|_  256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://pilgrimage.htb/
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   26.92 ms 10.10.16.1
2   57.46 ms 10.129.10.163
```
{% endcode %}

Port 80 reveals a domain name, I will add it to my hosts file.

{% code overflow="wrap" %}
```shellscript
10.129.10.163	pilgrimage.htb
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Website</mark>

<figure><img src="../../.gitbook/assets/image (3272).png" alt=""><figcaption></figcaption></figure>

The site shrinks images&#x20;

If I give it an image and click shrink it returns a URL to the smaller image

<figure><img src="../../.gitbook/assets/image (3274).png" alt=""><figcaption></figcaption></figure>

The URI does lead to a smaller version of the image I uploaded.

Visiting `/shrunk` returns a 403

If I create an account and upload a couple of images I can see them under the `/dashboard`&#x20;

<figure><img src="../../.gitbook/assets/image (3275).png" alt=""><figcaption></figcaption></figure>

The images are renamed on each upload, even if the image name is the same. It also always starts with `69af5` so it's probably no a hash making the name

#### <mark style="color:$primary;">Tech Detection</mark>

{% code overflow="wrap" %}
```shellscript
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Mon, 09 Mar 2026 18:33:21 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Set-Cookie: PHPSESSID=883tud4do7v8dmb5gg1dhudop6; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
```
{% endcode %}

We know its an nginx server hosting a site whose backed is written in PHP

#### <mark style="color:$primary;">Directory Busting</mark>

{% code overflow="wrap" %}
```shellscript
feroxbuster -u http://pilgrimage.htb -n -w ~/tools/SecLists/Discovery/Web-Content/raft-small-files.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3273).png" alt=""><figcaption></figcaption></figure>

Directory busting reveals a git repo

#### <mark style="color:$primary;">Git Repo</mark>

I’ll use [git-dumper](https://github.com/arthaud/git-dumper) to collect the repo

{% code overflow="wrap" %}
```shellscript
python3 ~/tools/git-dumper/git_dumper.py http://pilgrimage.htb/.git repo
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3276).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Source Code Analysis**</mark>

{% code title="index.php" overflow="wrap" %}
```php
...[snip]...
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
  $image = new Bulletproof\Image($_FILES);
  if($image["toConvert"]) {
    $image->setLocation("/var/www/pilgrimage.htb/tmp");
    $image->setSize(100, 4000000);
    $image->setMime(array('png','jpeg'));
    $upload = $image->upload();
...[snip]...
```
{% endcode %}

It takes the POST request and creates a file object & save it to `/tmp`

{% code title="index.php" overflow="wrap" %}
```php
...[snip]...
if($upload) {
      $mime = ".png";
      $imagePath = $upload->getFullPath();
      if(mime_content_type($imagePath) === "image/jpeg") {
        $mime = ".jpeg";
      }
      $newname = uniqid();
...[snip]...
```
{% endcode %}

It takes the result and creates a new file name using `uniqid`

{% code title="index.php" overflow="wrap" %}
```php
exec("/var/www/pilgrimage.htb/magick convert /var/www/pilgrimage.htb/tmp/" . $upload->getName() . $mime . " -resize 50% /var/www/pilgrimage.htb/shrunk/" . $newname . $mime);
      unlink($upload->getFullPath());
```
{% endcode %}

Then it runs what looks like a binary on the system `magick` to convert it by shrinking it by 50% and deletes the original file:

There is a copy of magick in the repo let's check it out

<figure><img src="../../.gitbook/assets/image (3278).png" alt=""><figcaption></figcaption></figure>

I was able to find a CVE POC for a lower verions on github that seems to work here

{% embed url="https://github.com/Sybil-Scan/imagemagick-lfi-poc" %}

### <mark style="color:blue;">ImageMagick LFI - CVE-2022-44268</mark>

