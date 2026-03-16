# Incident Response

IR lifecycle, containment strategies, live response and evidence collection.

---

## IR lifecycle

```
Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned
```

| Phase | Goal | BTL1 focus |
|-------|------|-----------|
| **Preparation** | Policies, tools, playbooks in place | Know your tools before exam day |
| **Identification** | Detect and confirm the incident | SIEM alerts, log analysis |
| **Containment** | Stop the bleeding | Isolate, block, disable |
| **Eradication** | Remove root cause | Delete malware, patch vuln |
| **Recovery** | Restore normal operations | Verify clean, monitor |
| **Lessons Learned** | Improve defenses | Post-incident report |

!!! note "BTL1 exam focus"
    Most BTL1 tasks fall in the **Identification** and **Containment** phases —
    analyzing logs, identifying IOCs and recommending containment actions.

---

## Containment actions

### Short-term (immediate)

```bash
# Network isolation — Windows
netsh advfirewall set allprofiles state on
netsh advfirewall firewall add rule name="BLOCK C2" protocol=TCP dir=out remoteip=185.220.101.45 action=block

# Disable user account — PowerShell
Disable-ADAccount -Identity compromised_user

# Kill malicious process
taskkill /F /PID <PID>
Stop-Process -Id <PID> -Force

# Block at firewall
# Add deny rule for malicious IP / domain
```

### Long-term (post-incident)

- Patch exploited vulnerability
- Reset all credentials on affected systems
- Review and harden access controls
- Update detection signatures

---

## Live response — Windows

```powershell
# Running processes
Get-Process | Select-Object Name, Id, CPU, StartTime | Sort-Object CPU -Descending

# Network connections
netstat -ano
Get-NetTCPConnection | Where-Object State -eq "Established" | Sort-Object RemoteAddress

# Scheduled tasks
Get-ScheduledTask | Where-Object State -eq "Ready" | Select-Object TaskName, TaskPath

# Services — look for unusual names
Get-Service | Where-Object Status -eq "Running" | Sort-Object DisplayName

# Recently modified files
Get-ChildItem C:\Users -Recurse | Where-Object LastWriteTime -gt (Get-Date).AddDays(-7) | Sort-Object LastWriteTime -Descending

# Startup items
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location

# Logged-on users
query user

# Event log — last 100 failed logins
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625} -MaxEvents 100
```

---

## Evidence collection priorities

```
1. Memory dump          — most volatile, capture first
2. Running processes    — before any shutdown
3. Network connections  — active at time of incident
4. Event logs           — Windows Security, System, Application
5. Disk image           — after volatile data captured
6. File artifacts       — prefetch, LNK files, browser history
```

```bash
# Memory acquisition — Windows (as admin)
winpmem_mini_x64_rc2.exe memory.dmp

# Event log export
wevtutil epl Security C:\evidence\security.evtx
wevtutil epl System C:\evidence\system.evtx
wevtutil epl Application C:\evidence\application.evtx

# Disk image — FTK Imager or dd
# FTK Imager: File > Create Disk Image > Physical Drive
```

---

## Incident report structure

```markdown
## Executive Summary
What happened, when, impact in plain language.

## Timeline
Chronological sequence of events with timestamps.

## Technical Findings
- Initial access vector
- Tools and techniques used (MITRE ATT&CK IDs)
- Systems affected
- Data accessed or exfiltrated

## IOCs
| Type | Value | Source |
|------|-------|--------|
| IP | 185.220.101.45 | SIEM alert, Wireshark |
| Domain | evil-c2.ru | DNS logs |
| Hash | 44d88612... | AV alert |

## Containment Actions Taken
What was done to stop the incident.

## Recommendations
What to fix to prevent recurrence.
```

!!! tip "BTL1 report tips"
    - Be specific with timestamps — include timezone
    - Reference your evidence — say where you found each finding
    - Map techniques to MITRE ATT&CK IDs where possible
    - Keep the executive summary to 3-4 sentences max
