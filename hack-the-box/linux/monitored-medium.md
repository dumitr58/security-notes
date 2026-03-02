---
icon: ubuntu
---

# Monitored - Medium

<figure><img src="../../.gitbook/assets/image (3236).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/monitored"><strong>Monitored</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP" overflow="wrap" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.230.96 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-01 16:03 -0500
Nmap scan report for 10.129.230.96
Host is up (0.049s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 61:e2:e7:b4:1b:5d:46:dc:3b:2f:91:38:e6:6d:c5:ff (RSA)
|   256 29:73:c5:a5:8d:aa:3f:60:a9:4a:a3:e5:9f:67:5c:93 (ECDSA)
|_  256 6d:7a:f9:eb:8e:45:c2:02:6a:d5:8d:4d:b3:a3:37:6f (ED25519)
80/tcp   open  http       Apache httpd 2.4.56
|_http-title: Did not follow redirect to https://nagios.monitored.htb/
|_http-server-header: Apache/2.4.56 (Debian)
389/tcp  open  ldap       OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   Apache httpd 2.4.56 ((Debian))
|_http-server-header: Apache/2.4.56 (Debian)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=nagios.monitored.htb/organizationName=Monitored/stateOrProvinceName=Dorset/countryName=UK
| Not valid before: 2023-11-11T21:46:55
|_Not valid after:  2297-08-25T21:46:55
| tls-alpn: 
|_  http/1.1
|_http-title: Nagios XI
5667/tcp open  tcpwrapped
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: Host: nagios.monitored.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT      ADDRESS
1   28.67 ms 10.10.16.1
2   59.40 ms 10.129.230.96
```
{% endcode %}

{% code title="Nmap UDP" overflow="wrap" %}
```shellscript
sudo nmap -sU -p- -Pn --min-rate 10000 10.129.230.96 -oN scans/nmap-udpall
[sudo] password for kali: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-01 16:11 -0500
Warning: 10.129.230.96 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.230.96
Host is up (0.054s latency).
Not shown: 65455 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
PORT    STATE SERVICE
123/udp open  ntp
161/udp open  snmp
```
{% endcode %}

### <mark style="color:blue;">Enumerating SNMP</mark>

{% code overflow="wrap" %}
```shellscript
python3 ~/tools/SNMP-Brute/snmpbrute.py -t 10.129.230.96
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3237).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
snmpbulkwalk -v2c -c public 10.129.230.96 . | tee snmp-public-v2c.out
```
{% endcode %}

Let's dive through the output and see what we can find. I'll check the Executables first

{% code overflow="wrap" %}
```shellscript
cat snmp-public-v2c.out | grep hrSWRunName
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3238).png" alt=""><figcaption></figcaption></figure>

There is a command run with sudo let's take a look at it

{% code overflow="wrap" %}
```shellscript
cat snmp-public-v2c.out | grep 1420
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3239).png" alt=""><figcaption></figcaption></figure>

We come across some credentials! They might be of use to us later, I'll write them down&#x20;

{% code overflow="wrap" %}
```shellscript
svc:XjH7VCehowpR1xZB
```
{% endcode %}

### <mark style="color:blue;">HTTP/HTTPS - website</mark>

<figure><img src="../../.gitbook/assets/image (3240).png" alt=""><figcaption></figcaption></figure>

On port 80 we see a redirect to https (443) and a domain and subdomain name I will add them to the hosts file

{% code overflow="wrap" %}
```shellscript
10.129.230.96	monitored.htb nagios.monitored.htb
```
{% endcode %}

