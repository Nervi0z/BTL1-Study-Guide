# SIEM Analysis

Splunk SPL queries, log correlation and alert triage. The domain where most BTL1 time is spent.

---

## Splunk SPL essentials

### Basic search structure

```splunk
index=<index> sourcetype=<type> field=value
| command1 args
| command2 args
```

### Most-used commands

```splunk
# Count events by field
index=windows EventCode=4625
| stats count by src_ip, user
| sort -count

# Filter time window
index=* earliest=-24h latest=now

# Search for specific string
index=* "powershell" "DownloadString"
| table _time, host, CommandLine

# Top values
index=* sourcetype=syslog
| top limit=20 src_ip

# Rare values (anomaly hunting)
index=* sourcetype=syslog
| rare limit=10 user

# Transaction — group related events
index=* src_ip=185.220.101.45
| transaction src_ip maxspan=10m
| table _time, duration, eventcount

# Timechart — visualize over time
index=* EventCode=4625
| timechart span=1h count by src_ip

# Eval — create calculated fields
index=* bytes_out=*
| eval MB = round(bytes_out/1024/1024, 2)
| table _time, src_ip, dst_ip, MB
| sort -MB
```

---

## Windows Event IDs — quick reference

| Event ID | Description | Why it matters |
|----------|-------------|----------------|
| `4624` | Successful logon | Baseline normal logins |
| `4625` | Failed logon | Brute force detection |
| `4648` | Explicit credentials logon | Pass-the-hash, lateral movement |
| `4672` | Special privileges assigned | Admin logon |
| `4688` | Process creation | Command execution |
| `4698` | Scheduled task created | Persistence |
| `4720` | User account created | Account creation |
| `4732` | Added to security group | Privilege escalation |
| `7045` | New service installed | Malware persistence |
| `4776` | NTLM auth attempt | Credential attack |

---

## Common investigation queries

### Brute force detection

```splunk
index=windows EventCode=4625
| stats count as failures by src_ip, user
| where failures > 20
| sort -failures
```

### Lateral movement — unusual logon types

```splunk
index=windows EventCode=4624 Logon_Type=3
| stats count by src_ip, dest_host, user
| where src_ip != dest_host
| sort -count
```

### PowerShell execution

```splunk
index=windows EventCode=4688
| where CommandLine like "%powershell%"
| search CommandLine IN ("*DownloadString*","*IEX*","*EncodedCommand*","*bypass*")
| table _time, host, user, CommandLine
```

### Scheduled task creation

```splunk
index=windows EventCode=4698
| table _time, host, user, TaskName, TaskContent
| sort -_time
```

### Data exfiltration — large outbound transfers

```splunk
index=network
| eval MB = round(bytes_out/1024/1024, 2)
| where MB > 100
| table _time, src_ip, dst_ip, dst_port, MB
| sort -MB
```

---

## ELK / Kibana

### KQL equivalents

```kql
# Filter by field value
event.code: "4625" and source.ip: "185.220.101.45"

# Range query
event.code: 4624 and @timestamp >= "2024-03-01"

# Wildcard
process.command_line: *powershell* and process.command_line: *DownloadString*

# Exists
process.command_line: *
```

---

!!! tip "BTL1 SPL workflow"
    1. Start broad — `index=* | stats count by sourcetype` to understand what data you have
    2. Identify the timeframe of the incident
    3. Build progressively — add filters one at a time
    4. Always `| table _time, [relevant fields]` before sharing results
    5. Document the exact query in your report
