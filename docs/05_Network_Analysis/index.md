# Network Analysis

Wireshark display filters, PCAP analysis, protocol identification and C2 detection.

---

## Wireshark display filters — cheatsheet

### By IP

```wireshark
ip.addr == 185.220.101.45          # traffic to or from IP
ip.src == 10.0.0.5                 # only traffic FROM this IP
ip.dst == 8.8.8.8                  # only traffic TO this IP
ip.addr == 192.168.1.0/24          # entire subnet
not ip.addr == 192.168.1.1         # exclude noise
ip.addr == 1.2.3.4 and ip.addr == 5.6.7.8   # conversation between two IPs
```

### By port

```wireshark
tcp.port == 443          # HTTPS
tcp.port == 3389         # RDP
tcp.port == 22           # SSH
tcp.port == 445          # SMB
udp.port == 53           # DNS
tcp.dstport == 4444      # common C2 port
```

### By protocol

```wireshark
http
dns
icmp
arp
smb2
tls
ftp
smtp
```

### TCP flags

```wireshark
# SYN scan detection
tcp.flags.syn == 1 and tcp.flags.ack == 0

# SYN-ACK
tcp.flags.syn == 1 and tcp.flags.ack == 1

# Connection reset
tcp.flags.reset == 1

# FIN — clean close
tcp.flags.fin == 1
```

### DNS analysis

```wireshark
dns                                          # all DNS
dns.qry.name == "evil-c2.ru"
dns.qry.name contains "pastebin"
dns.flags.rcode != 0                         # errors — NXDOMAIN etc
dns.qry.type == 28                           # AAAA queries (IPv6)
```

### HTTP analysis

```wireshark
http.request                                 # requests only
http.response                                # responses only
http.request.method == "POST"
http.response.code >= 400                    # errors
http.request.uri contains ".php"
http.user_agent contains "curl"
http.user_agent == ""                        # empty user-agent — suspicious
```

### Payload search

```wireshark
tcp contains "password"
http contains "cmd.exe"
tcp contains "PASS "                         # FTP credentials
```

---

## Detection patterns

### Port scan (Nmap SYN scan)

Many SYN packets to different ports from same source, no SYN-ACK responses:

```wireshark
tcp.flags.syn == 1 and tcp.flags.ack == 0 and ip.src == <suspect_ip>
```

### ARP spoofing

```wireshark
arp.duplicate-address-detected
```

### DNS tunneling

Long query names or high query volume from single host:

```wireshark
dns.qry.name matches "^[a-z0-9]{20,}\."
```

### C2 beaconing

Regular outbound connections at fixed intervals. Use `Statistics > IO Graph` with 1-second resolution to spot periodic spikes.

```wireshark
ip.src == <internal_ip> and tcp.dstport == 443
```

---

## Follow TCP stream

Right-click any TCP packet → `Follow > TCP Stream`

Shows the full session in plain text. If HTTP is in cleartext, you see the complete request and response — credentials, POST data, shell commands.

## Export HTTP objects

`File > Export Objects > HTTP`

Automatically extracts all files transferred over HTTP — executables, documents, scripts.

## Statistics menu

| Menu | Use |
|------|-----|
| `Statistics > Conversations` | Top IPs by volume — spot exfiltration |
| `Statistics > Protocol Hierarchy` | Protocol distribution |
| `Statistics > IO Graph` | Traffic over time — beaconing detection |
| `Statistics > DNS` | Top queried domains |
| `Analyze > Expert Information` | Errors, retransmissions, resets |
