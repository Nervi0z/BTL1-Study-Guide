# Digital Forensics

Memory and disk acquisition, Volatility 3, Autopsy and artifact analysis.

---

## Memory analysis — Volatility 3

### Core command structure

```bash
python3 vol.py -f <memory.dump> <plugin>

# Get OS info first — always
python3 vol.py -f memory.dmp windows.info.Info
```

### Process analysis

```bash
# List running processes
python3 vol.py -f memory.dmp windows.pslist.PsList

# Process tree — shows parent-child relationships
python3 vol.py -f memory.dmp windows.pstree.PsTree

# Scan for hidden/injected processes
python3 vol.py -f memory.dmp windows.psscan.PsScan

# Compare pslist vs psscan — hidden processes show only in psscan
python3 vol.py -f memory.dmp windows.psscan.PsScan > psscan.txt
python3 vol.py -f memory.dmp windows.pslist.PsList > pslist.txt
diff psscan.txt pslist.txt

# Command line arguments
python3 vol.py -f memory.dmp windows.cmdline.CmdLine

# DLLs loaded by a process
python3 vol.py -f memory.dmp windows.dlllist.DllList --pid <PID>

# Dump a process executable
python3 vol.py -f memory.dmp windows.procdump.ProcDump --pid <PID> --dump-dir ./output/
```

### Network connections

```bash
# Active and recent connections
python3 vol.py -f memory.dmp windows.netstat.NetStat

# All connection artifacts (including closed)
python3 vol.py -f memory.dmp windows.netscan.NetScan
```

### Malware detection

```bash
# Check for code injection
python3 vol.py -f memory.dmp windows.malfind.Malfind

# Registry analysis
python3 vol.py -f memory.dmp windows.registry.hivelist.HiveList
python3 vol.py -f memory.dmp windows.registry.printkey.PrintKey --key "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Handles — files and registry keys open by a process
python3 vol.py -f memory.dmp windows.handles.Handles --pid <PID>
```

### Most useful Volatility 3 plugins — cheatsheet

| Plugin | Purpose |
|--------|---------|
| `windows.info.Info` | OS version, arch — run first |
| `windows.pslist.PsList` | Active processes |
| `windows.pstree.PsTree` | Process tree |
| `windows.psscan.PsScan` | Scan for hidden processes |
| `windows.cmdline.CmdLine` | Command line arguments |
| `windows.netscan.NetScan` | Network connections |
| `windows.netstat.NetStat` | Active connections |
| `windows.malfind.Malfind` | Injected code detection |
| `windows.dlllist.DllList` | Loaded DLLs |
| `windows.procdump.ProcDump` | Dump process executable |
| `windows.dumpfiles.DumpFiles` | Dump files from memory |
| `windows.registry.hivelist.HiveList` | Registry hives |
| `windows.hashdump.Hashdump` | Extract password hashes |

---

## Disk forensics — Autopsy

### Workflow

1. **Create new case** — set case name, examiner
2. **Add data source** — disk image (`.E01`, `.dd`, `.vmdk`)
3. **Run ingest modules** — select: File Type ID, Hash Lookup, Recent Activity, Keyword Search
4. **Navigate artifacts**:
   - `Results > Extracted Content > Web History` — browser activity
   - `Results > Extracted Content > Recent Documents` — recently opened files
   - `Results > Extracted Content > Installed Programs` — software list
   - `Results > Extracted Content > OS Accounts` — user accounts
5. **Timeline analysis** — `Tools > Timeline` for chronological activity view
6. **Export findings** — right-click artifact > Export File

### Key filesystem locations

```
# Windows artifacts
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\   # LNK files
C:\Users\<user>\AppData\Local\Microsoft\Windows\WebCache\   # Browser cache
C:\Windows\Prefetch\                                         # Program execution
C:\Windows\System32\winevt\Logs\                            # Event logs
C:\Users\<user>\NTUSER.DAT                                  # User registry hive
C:\Windows\System32\config\SAM                              # Password hashes
```

---

## Hashing and verification

```bash
# Generate hashes
sha256sum evidence.dd
md5sum evidence.dd

# Verify integrity after acquisition
sha256sum -c evidence.sha256

# Hash a directory
find /evidence/ -type f -exec sha256sum {} \; > hashes.txt
```

!!! warning "Chain of custody"
    Always hash evidence before and after analysis.
    Document hash values in your report as proof of integrity.
