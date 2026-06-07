# Windows DFIR Investigation — bgoc.corp Enterprise Compromise

**Course:** Azerbaijan Cybersecurity Center — Technion Institute Blue Team Program  
**Date:** June 2026  
**Environment:** Controlled training lab (bgoc.corp domain)  
**Report:** [DFIR\_bgoc\_corp\_compromise.pdf](./report/DFIR_bgoc_corp_compromise.pdf) — 25 pages

---

## Overview

A full correlated Digital Forensics and Incident Response (DFIR) investigation of a multi-stage enterprise compromise. Two forensic disk images were acquired and examined:

| Host | Role | OS |
|------|------|----|
| BGOC111 | Domain-joined client (primary victim) | Windows 10 Education |
| BGOC00 | Domain server | Windows Server 2022 Datacenter |

Artifacts from both images were cross-correlated to reconstruct a complete, end-to-end attack chain spanning **November 2023 through November 2025**.

---

## Attack Chain — 5 Phases

| Phase | Period | Summary |
|-------|--------|---------|
| **1 — Initial Foothold** | Nov 2023 – Mar 2024 | DACL service backdoor installed on client; machine subsequently built into a full attack platform (Nmap, Wireshark, Burp Suite, Mimikatz, BloodHound, etc.) via Chocolatey in a single session |
| **2 — Reconnaissance** | Apr 2024 | BloodHound/SharpHound run against ACC domain; attack paths mapped; vulnerable internal services staged |
| **3 — Social Engineering & Staging** | Oct – Dec 2024 | User targeted via WhatsApp ClickFix payload; domain admin account compromised; Nishang reverse shell staged on attacker-controlled server (192.168.80.65) |
| **4 — Server Penetration & Lateral Movement** | May – Nov 2025 | Mimikatz credential dumping; Kerberos ticket export; fresh SharpHound collection from server; domain management tools accessed; inbound WinRM session received |
| **5 — Objectives & Cleanup** | 2025-11-04 | Data staged and exfiltrated; 46-second anti-forensic cleanup sequence executed — Recycle Bin not emptied, key artifacts recovered |

---

## Artifacts Analyzed

**Client (BGOC111)**
- Prefetch files — execution history and tool identification
- Shell bag artifacts — UNC path navigation to `\\DC\c$`, `\\DC\SYSVOL`, `\\DC\TOOLS`
- Chrome download history — WhatsApp ClickFix delivery chain reconstructed
- Registry hives — installed programs, OS profile, persistence mechanisms
- USB device history — attack date correlation and examiner acquisition confirmed
- Autopsy Interesting Items — RMM tools (Atera, mRemoteNG, VNC, PsExec), privacy programs (Tor, OpenVPN), credential tools (KeePass, Mimikatz)
- Windows Prefetch — `CERTUTIL.EXE` burst (×4 in 12 seconds), `SCHTASKS.EXE`, `WHOAMI.EXE`

**Server (BGOC00)**
- Recent Documents / JumpList — Administrator activity timeline
- Browser history (Edge) — SharpHound download from GitHub, exfiltration services
- Recycle Bin `$R` entries — deleted files recovered despite anti-forensic attempt
- Shell bags — loot directory and backup folder access
- BAM (Background Activity Monitor) — WinRM host process (`wsmprovhost.exe`) session timestamp
- MFT (`$MFT`) — MMC snap-in access (`gpmc.msc`, `dnsmgmt.msc`, `dssite.msc`, `lusrmgr.msc`)
- SQLite browser database — analyzed with DB Browser for SQLite

---

## Key Findings

- **Shared Administrator SID** present in artifacts on both machines — same privileged account used across both hosts, confirming full domain takeover
- **Account `groupf`** active simultaneously on client and server on 2025-11-04 — identified as the primary hands-on-keyboard operator
- **Multi-domain footprint** — Kerberos TGT export confirmed both `bgoc.corp` and `ACC.LAB` domains compromised
- **`\\DC\TOOLS` share** hosted 60+ offensive tools accessible via SMB (Mimikatz, Rubeus, BloodHound, Covenant, Sliver, ADConnectDump, and more)
- **Anti-forensic cleanup** completed in 46 seconds; Recycle Bin not subsequently emptied — deleted artifacts fully recovered via Autopsy

---

## Tools & Techniques

| Category | Tools Used |
|----------|------------|
| Disk Forensics | Autopsy, FTK Imager |
| Registry & Artifact Analysis | Registry Explorer, Autopsy Registry Viewer |
| Timeline & Artifact Parsing | Eric Zimmerman Tools (PECmd, LECmd, SBECmd, EvtxECmd) |
| Database Analysis | DB Browser for SQLite |
| Threat Framework | MITRE ATT&CK |

---

## Report

The full investigation is documented in the 25-page DFIR report linked above. It covers:

- Executive summary and network topology
- Full artifact-by-artifact evidence walkthrough
- Persistence mechanisms and C2 channel analysis  
- Privilege escalation and lateral movement evidence
- Data staging, exfiltration sequence, and anti-forensic activity
- Complete integrated timeline (2023–2025)
- Compromised and suspicious accounts summary
- Remediation recommendations and flags for further investigation

---

> **Note:** This investigation was conducted in a controlled training environment as part of the Azerbaijan Cybersecurity Center (Technion Institute) Blue Team program. All hostnames, domain names, and IP addresses are fictional training artifacts.
