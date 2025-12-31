---
icon: ubuntu
---

# Mentor - Medium

<figure><img src="../../.gitbook/assets/image.png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/mentor"><strong>Mentor</strong></a></p></figcaption></figure>

## <mark style="color:green;">Scanning & Enumeration</mark>

Nmap TCP scan:

```shellscript
## Nmap TCP
Nmap scan report for 10.10.11.193
Host is up (0.051s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c7:3b:fc:3c:f9:ce:ee:8b:48:18:d5:d1:af:8e:c2:bb (ECDSA)
|_  256 44:40:08:4c:0e:cb:d4:f1:8e:7e:ed:a8:5c:68:a4:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://mentorquotes.htb/
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: Host: mentorquotes.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT       ADDRESS
1   100.07 ms 10.10.16.1
2   32.15 ms  10.10.11.193
```

on Port 80 we have a redirect to `mentorquotes.htb` I'll add it to my hosts file

```shellscript
10.10.11.193	mentorquotes.htb
```

Nmap UDP scan:

```shellscript
## Nmap UDP
sudo nmap -sU -p- -Pn --min-rate 10000 10.10.11.193 -oN scans/nmap-udpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-30 21:12 EST
Warning: 10.10.11.193 giving up on port because retransmission cap hit (10).
Nmap scan report for mentorquotes.htb (10.10.11.193)
Host is up (0.098s latency).
Not shown: 65456 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT    STATE SERVICE
161/udp open  snmp

Nmap done: 1 IP address (1 host up) scanned in 72.90 seconds
```

Since SNMP is open on this box I am going to take a look at it firt

### <mark style="color:blue;">SNMP 161 UDP</mark>

I'll use [snmpbrute](https://github.com/SECFORCE/SNMP-Brute) to identify the community strings

```shellscript
python3 snmpbrute.py -t 10.10.11.193
```

<figure><img src="../../.gitbook/assets/image (3067).png" alt=""><figcaption></figcaption></figure>

(RO) stands for read only

There is an internal string I am going to dump everything using snmpbulkwalk and dig through it

```shellscript
snmpbulkwalk -v2c -c internal 10.10.11.193 . | tee snmp-internal.out
```

#### <mark style="color:$primary;">Enumerating SNMP internal v2c Dump</mark>

First executables

```shellscript
grep SWRunName snmp-internal.out
```

<figure><img src="../../.gitbook/assets/image (3068).png" alt=""><figcaption></figcaption></figure>

there is a login.py that stands out

<figure><img src="../../.gitbook/assets/image (3069).png" alt=""><figcaption></figcaption></figure>

There seems to be some sort of password here I'll note it for later use

```shellscript
kj23sadkj123as0-d213
```

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Subdomain Enumeration</mark>

{% code overflow="wrap" %}
```shellscript
wfuzz -c -f sub-fighter -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://mentorquotes.htb' -H 'HOST: FUZZ.mentorquotes.htb' --hw 26
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

A Subdomain Enumeration reveals an `api` subdomain. I'll add it to my hosts file

When visiting it it reveals nothing

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Directory busting on api.mentorquotes.htb</mark>

```shellscript
feroxbuster -u http://api.mentorquotes.htb
```

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

reveals 3 interesting endpoints

#### <mark style="color:yellow;">api.mentorquotes.htb/docs</mark>

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

`james@mentorquotes.htb` this seems to be the administrator email for the website.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

This is a token based API, when registering a user it would create a JWT token for that user. We will either have to spoof the token to become the administrator or we will have to find an injection point for RCE.

Now with access to the api endpoints we can create a user

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
