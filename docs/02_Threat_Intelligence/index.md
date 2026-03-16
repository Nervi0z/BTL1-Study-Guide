# Threat Intelligence

MITRE ATT&CK, IOC enrichment, CTI frameworks and threat actor profiling.

---

## MITRE ATT&CK

The ATT&CK framework maps adversary behavior into Tactics, Techniques and Procedures (TTPs).

**Tactics** — the *why* (what the attacker is trying to achieve)  
**Techniques** — the *how* (specific method used)  
**Sub-techniques** — more specific variation of a technique

### Tactic order — attack lifecycle

| # | Tactic | ID | Description |
|---|--------|----|-------------|
| 1 | Reconnaissance | TA0043 | Gather target information |
| 2 | Resource Development | TA0042 | Build/acquire infrastructure |
| 3 | Initial Access | TA0001 | Get into the environment |
| 4 | Execution | TA0002 | Run malicious code |
| 5 | Persistence | TA0003 | Maintain foothold |
| 6 | Privilege Escalation | TA0004 | Gain higher permissions |
| 7 | Defense Evasion | TA0005 | Avoid detection |
| 8 | Credential Access | TA0006 | Steal credentials |
| 9 | Discovery | TA0007 | Learn the environment |
| 10 | Lateral Movement | TA0008 | Move to other systems |
| 11 | Collection | TA0009 | Gather target data |
| 12 | Command & Control | TA0011 | Communicate with implant |
| 13 | Exfiltration | TA0010 | Steal data |
| 14 | Impact | TA0040 | Disrupt / destroy / ransom |

---

## IOC enrichment workflow

```bash
# IP reputation
iocx ip 185.220.101.45

# Domain reputation
iocx domain evil-c2.ru

# File hash
iocx hash 44d88612fea8a8f36de82e1278abb02f

# Bulk scan from file with HTML report
iocx scan iocs.txt --output report.html
```

### Key OSINT sources

| Source | Best for |
|--------|----------|
| [VirusTotal](https://virustotal.com) | Hash, IP, domain, URL reputation |
| [AbuseIPDB](https://abuseipdb.com) | IP abuse reports |
| [Shodan](https://shodan.io) | Open ports, banners, services |
| [URLhaus](https://urlhaus.abuse.ch) | Malware distribution URLs |
| [MalwareBazaar](https://bazaar.abuse.ch) | Malware sample lookup |
| [AlienVault OTX](https://otx.alienvault.com) | Threat intel pulses |
| [MITRE ATT&CK](https://attack.mitre.org) | TTP mapping |

---

## Pyramid of Pain

Understanding which IOC type hurts the attacker most when blocked:

```
        /\
       /  \   ← TTPs (hardest to change — most valuable)
      /----\
     / Tools \  ← Malware hashes, C2 frameworks
    /----------\
   /  Network   \  ← IP addresses, domains
  /  Artifacts   \
 /-----------------\
/   Hash Values     \  ← MD5/SHA1 (easiest to change — least valuable)
```

!!! tip
    Focus your detection on TTPs and tools — they are much harder
    for adversaries to change than IPs or hashes.
