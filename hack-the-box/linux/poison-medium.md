---
icon: ubuntu
---

# Poison - Medium

<figure><img src="../../.gitbook/assets/image (11).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/poison"><strong>Poison</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" overflow="wrap" %}
```shellscript
nmap -A -T4 -Pn 10.129.1.254 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-12 11:54 -0400
Nmap scan report for 10.129.1.254
Host is up (0.041s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
|_  256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=3/12%OT=22%CT=1%CU=44417%PV=Y%DS=2%DC=T%G=Y%TM=69B2E1E
OS:3%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=106%TI=Z%CI=Z%II=RI%TS=22)S
OS:EQ(SP=103%GCD=1%ISR=104%TI=Z%CI=Z%II=RI%TS=22)SEQ(SP=104%GCD=1%ISR=106%T
OS:I=Z%CI=Z%II=RI%TS=21)SEQ(SP=107%GCD=1%ISR=108%TI=Z%CI=Z%II=RI%TS=21)SEQ(
OS:SP=FE%GCD=1%ISR=106%TI=Z%CI=Z%II=RI%TS=22)OPS(O1=M542NW6ST11%O2=M542NW6S
OS:T11%O3=M280NW6NNT11%O4=M542NW6ST11%O5=M218NW6ST11%O6=M109ST11)WIN(W1=FFF
OS:F%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FFFF)ECN(R=Y%DF=Y%T=40%W=FFFF%O=M54
OS:2NW6SLL%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=Y%DF=
OS:Y%T=40%W=FFFF%S=O%A=S+%F=AS%O=M109NW6ST11%RD=0%Q=)T4(R=Y%DF=Y%T=40%W=0%S
OS:=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R
OS:=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=
OS:AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=38%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%R
OS:UD=G)IE(R=Y%DFI=S%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   87.21 ms 10.10.16.1
2   25.16 ms 10.129.1.254
```
{% endcode %}

Nmap Reveals FreeBSD OS

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

{% code overflow="wrap" %}
```shellscript
curl -I http://10.129.1.254                                                                                                                     
HTTP/1.1 200 OK
Date: Thu, 12 Mar 2026 16:01:02 GMT
Server: Apache/2.4.29 (FreeBSD) PHP/5.6.32
X-Powered-By: PHP/5.6.32
Content-Type: text/html; charset=UTF-8
```
{% endcode %}

An apache server hosting a site whose backed is written in PHP.

#### <mark style="color:$primary;">Website</mark>

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

I put in listfiles.php and it reveals and interesting text file let's check it out

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

It reveals a password that has been base64 encoded 13 times. Let's decode it using some command line fu

{% code title="Save text to a variable" overflow="wrap" %}
```shellscript
data=$(cat pwd.b64)
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Let's base64 decoded it 13 times and get our password

{% code overflow="wrap" %}
```shellscript
for i in $(seq 1 13); do data=$(echo "$data" | tr -d ' \n\r' | base64 -d 2>/dev/null); done
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Nice we have a password now we need a user!

This website is one big LFI

### <mark style="color:blue;">LFI</mark>

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

We got a couple of users, but by the looks of it our password might match with charix

{% code overflow="wrap" %}
```shellscript
charix:Charix!2#4%6&8(0
```
{% endcode %}

Using this password we can login via ssh as the charix user

{% code overflow="wrap" %}
```shellscript
ssh charix@10.129.1.254
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as Charix</mark>

#### <mark style="color:$primary;">Manual Enumeration</mark>

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

There is an interesting zip file in charix's home directory password protected. I'll download it to my machine&#x20;

{% code overflow="wrap" %}
```shellscript
scp charix@10.129.1.254:secret.zip .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

I was able to unzip it using the same password we discovered earlier: <mark style="color:$success;">**Charix!2#4%6&8(0**</mark>

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Taking a closer look at the file it is exactly <mark style="color:$success;">8 bytes in size</mark>, <mark style="color:$success;">a binary blob</mark> located in <mark style="color:$success;">/home/charix</mark> folder this is a <mark style="color:$success;">**TightVNC password**</mark>!

&#x20;Next I checked the running processes <mark style="color:$warning;">**(this is freeBSD so the commands are a little different)**</mark>

{% code overflow="wrap" %}
```shellscript
ps -auxwww -U root
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

TightVNC is running on this machine as the root user!&#x20;

* `:1` - display number 1
* `-rfbauth /root/.vnc/passwd` - specifies the file containing the password used to auth viewers
* `-rfbport 5901` - tells us which port to connect to
* `localhost` - only listen locally

{% code overflow="wrap" %}
```shellscript
netstat -an -p tcp
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

5801 & 5901 are VNC ports, for remote desktop access.

I know it says it's using the `/root/.vnc/passwd` password file to authenticate users. I might be able to use the secret file we got earlier.

I'll use SSH Tunneling to forward port 5901 from the target's localhost to my local Kali machine

### <mark style="color:blue;">SSH Tunnel</mark>

{% code overflow="wrap" %}
```shellscript
ssh -L 5901:127.0.0.1:5901 charix@10.129.1.254
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

With the tunnel set, we can use vncviewer and point it to 127.0.0.1:5901 (using the secret file we got earlier for authentication)

{% code overflow="wrap" %}
```shellscript
vncviewer -passwd secret 127.0.0.1:5901
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

and we managed to get a shell as the root user!
