---
icon: user-secret
---

# Reconnaissance

## <mark style="color:blue;">Network Scanning</mark>&#x20;

* [ ] Nmap TCP scan

```shellscript
nmap -T4 -p- -Pn $ip -oN scans/nmap-tcpall
```

```shellscript
nmap -A -T4 -p- -Pn $ip -oN scans/nmap-fulltcpall
```



* [ ] Nmap UDP scan

```shellscript
sudo nmap -sU -p- --min-rate 10000 -Pn $ip -oN scans/nmap-udpall
```

```shellscript
sudo nmap -sU -p- --min-rate 10000 --open $ip -oN scans/nmap-udpall
```

### <mark style="color:$primary;">Scanning Internal Networks</mark>

* [ ] Scaning Internal Network

```shellscript
nmap -T4 -p- -Pn -iL ip-toscan -oN scans/nmap-internal
```

* [ ] Scanning Internal Network with ProxyChains

```shellscript
proxychains -q nmap -sT 10.10.10.10 -p 21,22,23,35 -Pn -v
```
