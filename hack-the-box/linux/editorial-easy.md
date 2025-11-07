---
icon: ubuntu
---

# Editorial - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
Nmap scan report for editorial.htb (10.10.11.20)
Host is up (0.057s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
|_  256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Editorial Tiempo Arriba
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 993/tcp)
HOP RTT      ADDRESS
1   34.72 ms 10.10.16.1
2   86.34 ms editorial.htb (10.10.11.20)
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

Upon visiting the site we get a redirect to editorial.htb I'll add it to my hosts file

```bash
10.10.11.20     editorial.htb
```

<figure><img src="../../.gitbook/assets/image (2798).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2799).png" alt=""><figcaption></figcaption></figure>

On the Publish with us page there seems to be a form that can take a url. I will setup a simple http.server to test and see if It can reach my machine.

<figure><img src="../../.gitbook/assets/image (2800).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2801).png" alt=""><figcaption></figcaption></figure>

It does! Let's open up burpsuite and further inspect this:

<figure><img src="../../.gitbook/assets/image (2802).png" alt=""><figcaption></figcaption></figure>

When trying the localhost port 80 it hangs for a good 10 seconds

<figure><img src="../../.gitbook/assets/image (2803).png" alt=""><figcaption></figcaption></figure>

I'll save the POST request to a file and FUZZ to see if there are any other open ports on the machine

<figure><img src="../../.gitbook/assets/image (2806).png" alt=""><figcaption></figcaption></figure>

in Burp, right click -> Copy to file

#### Fuzzing for open ports

{% code overflow="wrap" %}
```bash
ffuf -u http://editorial.htb/upload-cover -request upload-cover.req -w <( seq 0 65535) -ac
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2807).png" alt=""><figcaption></figcaption></figure>

We discover port 5000 to be open!&#x20;

The request works instantly as well and we are getting data back

<figure><img src="../../.gitbook/assets/image (2809).png" alt=""><figcaption></figcaption></figure>

This looks like an endpoint.

{% code overflow="wrap" %}
```bash
curl http://editorial.htb/static/uploads/cb53b08d-557b-4eea-a7df-0d713a5d9bbe -s | jq .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2810).png" alt=""><figcaption></figcaption></figure>

Thu .
