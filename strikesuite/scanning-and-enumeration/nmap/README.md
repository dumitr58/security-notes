---
icon: eye
---

# Nmap

#### <mark style="color:$primary;">TCP Scan</mark>

```shellscript
nmap -A -T4 -p- -Pn $ip -oN scans/nmap-tcpall
```

#### <mark style="color:$primary;">UDP Scan</mark>

```shellscript
sudo nmap -p 161 -sU -A -v  $ip
```

```shellscript
sudo nmap -sU -p- --min-rate 10000 -Pn $ip -oN scans/nmap-udpall
```

```shellscript
sudo nmap -sU -p- --min-rate 10000 --open $ip -oA scans/nmap-udpall
```

#### <mark style="color:$primary;">Scanning Internal Network after ligolo Tunnel setup</mark>

Scan top 1000 ports

```shellscript
nmap -T4 -Pn -iL ip-toscan -oN scans/nmap-internal
```

Scan all ports

```shellscript
nmap -T4 -p- -Pn -iL ip-toscan -oN scans/nmap-internal
```

#### <mark style="color:$primary;">Scanning Internal Network with Proxy Chains</mark>

```shellscript
proxychains -q nmap -sT 10.10.10.10 -p 21,22,23,35 -Pn -v
```

#### <mark style="color:$primary;">Using Nmap locally on the machine</mark>

```shellscript
./nmap -v -sn 10.10.10.0/24 --open
```

```shellscript
./nmap 10.200.80.50 10.200.30.50 -v -Pn -oN nmap.log -T5
```
