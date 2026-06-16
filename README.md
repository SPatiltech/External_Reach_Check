# Internet Exposure Checker

Lightweight scripts to determine whether a machine is reachable from the public internet.  
Works on **Linux/macOS** (Bash) and **Windows** (PowerShell). No API keys required.

---

## How It Works

Each script runs **4 checks** and produces a clear verdict:

| Step | What it does |
|------|-------------|
| 1. Resolve IP | Gets the machine's public IP (local mode) or resolves a target hostname |
| 2. Port scan | Tests whether common ports (22, 80, 443, 3389…) respond from this host |
| 3. Shodan lookup | Queries Shodan InternetDB — a free index of internet-facing hosts |
| 4. Verdict | Combines results into a clear EXPOSED / NOT EXPOSED summary |

---

## Usage

### Linux / macOS — `Linux_Reachability_Checker`

```bash
# Make executable (first time only)
chmod +x Linux_Reachability_Checker

# Run on the machine itself
./Linux_Reachability_Checker

# Run from a proxy/jump host targeting another machine
./Linux_Reachability_Checker --target 203.0.113.45

# Target a hostname
./Linux_Reachability_Checker --target myserver.example.com

# Custom ports
./Linux_Reachability_Checker --target 203.0.113.45 --ports 22,80,443,8080
```

**Requirements:** `bash`, `curl`, `nc` (netcat) — all standard on major Linux distros and macOS.

---

### Windows — `Windows__Reachability_Checker.ps1`

```powershell
# Run on the machine itself
.\Windows__Reachability_Checker.ps1

# Run from a proxy targeting another machine
.\Windows__Reachability_Checker.ps1 -Target "203.0.113.45"

# Target a hostname
.\Windows__Reachability_Checker.ps1 -Target "myserver.example.com"

# Custom ports
.\Windows__Reachability_Checker.ps1 -Target "203.0.113.45" -Ports 22,80,443,3389

# Adjust timeout (default 2000ms)
.\Windows__Reachability_Checker.ps1 -Target "203.0.113.45" -Timeout 3000
```

**Requirements:** PowerShell 5.1+ (built-in on Windows 10 / Server 2016 and later).

> If you see an execution policy error, run:  
> `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`

---

## Two Deployment Modes

```
┌─────────────────────────────────────────────────────────────┐
│  MODE 1 — Run on the machine itself                         │
│                                                             │
│   [Target Machine] ──runs──▶ script                         │
│   Script detects its own public IP and checks exposure      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  MODE 2 — Run on a proxy / jump host                        │
│                                                             │
│   [Proxy Host] ──runs──▶ script --target <IP>              │
│   Script probes a remote machine from the proxy             │
└─────────────────────────────────────────────────────────────┘
```

> **Best practice:** For true internet-wide exposure testing, run Mode 2 from an  
> external host (e.g., a cloud VPS) outside your network perimeter.

---

## Sample Output

```
╔══════════════════════════════════════════╗
║     Internet Exposure Checker v1.0       ║
║     Linux / macOS Edition                ║
╚══════════════════════════════════════════╝

  Mode     : local
  Run time : 2026-06-15 14:32:11 UTC

[1/4] Resolving target...
  Local machine's public IP : 203.0.113.45

[2/4] Checking port reachability from this host...
  Port 22   →  OPEN / REACHABLE
  Port 80   →  OPEN / REACHABLE
  Port 443  →  OPEN / REACHABLE
  Port 3389 →  CLOSED / FILTERED
  Port 8080 →  CLOSED / FILTERED
  Port 8443 →  CLOSED / FILTERED

[3/4] Querying Shodan InternetDB (no API key required)...
  ⚠  Host IS indexed by Shodan (internet-exposed)
  Exposed ports  : 22 80 443
  Known CVEs     : None found

[4/4] Summary
─────────────────────────────────────────────
  VERDICT: HOST IS EXPOSED TO THE INTERNET
  Shodan has indexed this IP — it is visible from the public internet.

  Open ports (this scan) : 3 — 22, 80, 443
  Closed / filtered      : 3 — 3389, 8080, 8443
─────────────────────────────────────────────
```

---

## Ports Checked by Default

| Port | Service |
|------|---------|
| 22   | SSH |
| 80   | HTTP |
| 443  | HTTPS |
| 3389 | RDP (Windows Remote Desktop) |
| 8080 | HTTP Alternate |
| 8443 | HTTPS Alternate |

Custom ports can be passed via `--ports` (Linux) or `-Ports` (Windows).

---

## No External Dependencies

- Uses **Shodan InternetDB** (`internetdb.shodan.io`) — free, no API key needed
- Uses **ipify.org** to detect public IP — no account required
- No third-party libraries to install

---

## Notes & Limitations

- Shodan InternetDB only covers hosts that Shodan has **already scanned**. A missing entry doesn't guarantee the host is unexposed — it may just not have been indexed yet.
- Port checks from the same network won't reflect internet-facing rules. Always verify from an **external network** for full accuracy.
- This tool is for **authorized security assessments only**. Only run it against machines you own or have explicit permission to test.
