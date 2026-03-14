<div align="center">

![](header.svg)

</div>

<br>

Practical reference for Security Blue Team Level 1 — focused on junior analysts and Tier 1 SOC roles.

This repo maps directly to the six BTL1 domains. Built for people learning by doing — not theory-first. If you're working toward your first SOC role and comfortable in a terminal, this is your field guide.

---

## Exam format

| | |
| :--- | :--- |
| **Format** | Fully practical — no multiple choice. Simulated corporate intrusion, work it like a real incident. |
| **Duration** | 24 hours to investigate, extract evidence, and write the report. |
| **Environment** | Cloud-hosted lab: multi-OS endpoints, network sensors, pre-configured SIEM (Splunk or Elastic). |
| **Evidence sources** | Disk images (`DD`/`E01`), memory dumps (`RAM`), packet captures (`PCAP`), SIEM logs. |
| **Deliverable** | Formal incident report — attack timeline, compromised assets, technical proof per finding. |

---

## Prerequisites

You don't need a security background to pass, but you do need these foundations:

**Networking** — TCP/IP stack, DNS resolution, HTTP/S traffic flow, common ports and protocols.

**Windows** — Registry hive structure, event logging (Sysmon, Security), process execution trees.

**Linux** — File system hierarchy, daemon execution, permission models.

**CLI** — Bash and PowerShell navigation, data manipulation, log parsing.

---

## Domain coverage

| Domain | Focus | Tools |
| :--- | :--- | :--- |
| Phishing analysis | SMTP header inspection, payload RE, URL pivoting | Any.run, PhishTool, CyberChef |
| Threat intelligence | TTP categorization, IOC enrichment, adversary mapping | MITRE ATT&CK, VirusTotal, OTX |
| Digital forensics | Disk & memory artifact analysis | Autopsy, Volatility 3, KAPE |
| SIEM analysis | Log correlation, SPL/KQL query writing | Splunk, Elastic |
| Network analysis | DPI, lateral movement detection, C2 identification | Wireshark, tshark, Zeek |
| Incident response | Live triage, containment, evidence preservation | Sysinternals, native OS CLI |

---

## Structure

```text
.
├── 00_introduction/        # triage methodology
├── 01_phishing/            # header analysis, attachment extraction
├── 02_threat_intel/        # IOC management, ATT&CK TTP mapping
├── 03_forensics/
│   ├── disk/               # NTFS, registry, file carving
│   └── memory/             # Volatility profiles, injection detection
├── 04_siem/                # SPL structures, correlation rules
├── 05_network/             # BPF filters, PCAP carving
└── 06_incident_response/   # IR lifecycle, live response
```

---

> General technical documentation and open-source tool references only. No proprietary exam content, lab infrastructure details, or restricted BTL1 materials — per SBT NDA.

---

MIT License · See `LICENSE`
