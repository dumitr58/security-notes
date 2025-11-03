---
icon: ubuntu
---

# Catto - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

nmap scan:

```bash
# Nmap TCP
map -A -T4 -p- -Pn 192.168.124.139 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-23 15:55 EDT
Nmap scan report for 192.168.124.139
Host is up (0.092s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
8080/tcp  open  http    nginx 1.14.1
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Identity by HTML5 UP
|_http-server-header: nginx/1.14.1
18080/tcp open  http    Apache httpd 2.4.37 ((centos))
|_http-title: CentOS \xE6\x8F\x90\xE4\xBE\x9B\xE7\x9A\x84 Apache HTTP \xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xB5\x8B\xE8\xAF\x95\xE9\xA1\xB5
|_http-server-header: Apache/2.4.37 (centos)
| http-methods: 
|_  Potentially risky methods: TRACE
30330/tcp open  http    Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
39783/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, ms-sql-s, oracle-tns: 
|     HTTP/1.1 400 Bad Request
|_    Connection: close
42022/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 cc:21:51:f2:c6:2a:ad:d6:ca:07:04:de:70:5f:fa:13 (RSA)
|   256 05:e4:90:d2:00:2b:9d:14:e3:9f:44:68:d2:8e:bc:dc (ECDSA)
|_  256 ca:80:49:73:f0:c8:05:ae:bd:2b:42:37:1d:13:e0:71 (ED25519)
44487/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-cors: HEAD GET POST PUT DELETE PATCH
50400/tcp open  http    Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-title: Error
```

### <mark style="color:$primary;">HTTP 8080 TCP</mark>

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

On this port I only discovered a static website. Directory busting lead nowhere, so I switched my focus to the next port 30330.

### <mark style="color:$primary;">HTTP 30330 TCP</mark>

<figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.gatsbyjs.com/docs/how-to/custom-configuration/typescript/" %}

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

Graphql is mentioned in the documentation!&#x20;

Visiting the `___graphql` endpoint we see that it is viable!

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

Checking the queries on the left I discovered some hidden directories under `allSitePage`&#x20;

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

Visiting the `new-server-config-mc` endpoint I discovered a password.

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

This password is for a Minecraft server.

Checking the `/minecraft` server There are some possible usernames we can extract from this paragraph and create a wordlist.

<figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

```
keralis
xisuma
zombiecleo
mumbojumbo
sabel
yvette
zahara
sybilla
marcus
tabbatha
tabby
```

We can use these usernames and the password we found earlier to try and brute force ssh.

### <mark style="color:$primary;">SSH Brute force</mark>

#### Hydra

Using hydra to brute force creds

{% code overflow="wrap" %}
```bash
hydra -L usernames -p WallAskCharacter305 192.168.124.139 ssh -s 42022
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

Hydra reveals the password works for the user marcus over ssh.

Since the list is small you can use netexec for brute forcing as well. It will take a little longer

#### Netexec&#x20;

{% code overflow="wrap" %}
```bash
netexec ssh 192.168.124.139 -u usernames -p 'WallAskCharacter305' --port 42022
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

```bash
ssh marcus@192.168.124.139 -p 42022
```

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

There is an interesting file that belongs to root in marcus home directory. The string looks base64 encode but when I tried decoding it did not work.

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

checking out the base binaries I saw an interesting one named `base64key`. This binary takes an encoded string and a password. I tried using the string I discoved in the .bash file and Marcus password and it worked!

```bash
base64key F2jJDWaNin8pdk93RLzkdOTr60== WallAskCharacter305 1
```

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

This password belongs to the root user

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>
