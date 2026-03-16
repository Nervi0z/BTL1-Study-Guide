# Phishing Analysis

Email header analysis, URL investigation, attachment triage and IOC extraction.

---

## Email header analysis

The full header reveals the actual routing path of an email, exposing spoofed senders and relay abuse.

```bash
# Key fields to check first
Received:          # Hop-by-hop routing — read bottom to top
From:              # Display name — easily spoofed
Reply-To:          # Where replies actually go — check for mismatch
Return-Path:       # Bounce address — often reveals real sender
X-Originating-IP:  # Original sending IP
Authentication-Results: # SPF / DKIM / DMARC results
```

### SPF / DKIM / DMARC

| Result | Meaning |
|--------|---------|
| `spf=pass` | Sender IP is authorized for that domain |
| `spf=fail` | IP not authorized — likely spoofed |
| `dkim=pass` | Message body not tampered |
| `dkim=fail` | Message was modified or signature invalid |
| `dmarc=fail` | Both SPF and DKIM failed alignment |

!!! danger "Red flags"
    - `Reply-To` domain different from `From` domain
    - `spf=fail` or `dmarc=fail` in Authentication-Results
    - `Received` chain shows unexpected countries or providers
    - Display name is a known executive but email domain is external

---

## URL analysis

Never click. Defang first, then investigate.

```bash
# Defanging — safe to share in reports
hxxps://evil[.]com/payload.exe
hxxp://192.168[.]1[.]100/gate.php

# Tools for URL investigation
# VirusTotal:  virustotal.com/gui/url
# URLScan:     urlscan.io
# URLhaus:     urlhaus.abuse.ch
# Any.run:     app.any.run
```

### URL IOC extraction with iocx

```bash
iocx url "hxxps://malicious[.]domain/payload.exe"
iocx domain suspicious-domain.com
```

---

## Attachment triage

```bash
# Get file hash before opening anything
sha256sum suspicious.pdf
md5sum suspicious.docx

# Check hash on VirusTotal
iocx hash <sha256_here>

# Static strings analysis
strings -n 8 suspicious.exe | grep -Ei 'http|cmd|powershell|wscript'

# File type verification — don't trust the extension
file suspicious.pdf
xxd suspicious.pdf | head -5   # Check magic bytes
```

### Common malicious file types in phishing

| Extension | What to look for |
|-----------|-----------------|
| `.docx` / `.xlsm` | Macros — check Enable Content prompt |
| `.pdf` | Embedded JavaScript, launch actions |
| `.html` | Credential harvesting pages |
| `.lnk` | PowerShell execution via shortcut |
| `.iso` / `.img` | Container to bypass Mark-of-the-Web |
| `.one` | OneNote with embedded scripts |

---

## PhishTool workflow

1. Upload the `.eml` file
2. Check the **Summary** tab — sender, recipient, subject, timestamp
3. Go to **Headers** — validate SPF/DKIM/DMARC, trace the hop chain
4. Open **Attachments** — hash each file, check VirusTotal
5. Expand **URLs** — defang, check each against URLhaus and VT
6. Tag the IOCs and export the report

---

## Reporting IOCs

```
# Format for incident report
Sender:          attacker@phishing-domain.com
Sending IP:      185.220.101.45
Subject:         Urgent: Invoice attached
SHA256 (attach): 44d88612fea8a8f36de82e1278abb02f
URLs found:      hxxps://evil[.]com/payload.exe
SPF result:      fail
DMARC result:    fail
Verdict:         MALICIOUS — credential harvesting campaign
```
