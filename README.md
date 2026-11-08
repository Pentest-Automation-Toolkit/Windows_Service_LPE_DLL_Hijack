# Windows Local Privilege Escalation Lab — Service Misconfigurations & DLL Hijacking

**Author:** Juan Andrés Alba Peralta
**Type:** Offensive Security / Red Team Portfolio Project
**Lab Environment:** Isolated VirtualBox network — no production systems or third parties involved

## Overview

Hands-on project evaluating local privilege escalation (LPE) paths on a deliberately
vulnerable Windows host, starting from a low-privileged standard user and escalating
to `NT AUTHORITY\SYSTEM`. The engagement covers systematic enumeration, exploitation
of common Windows service misconfigurations, and a defensive remediation roadmap for
each vector identified.

Full technical write-up (methodology, evidence, and remediation steps):
[`report/Windows_LPE_Lab_Report.pdf`](report/Windows_LPE_Lab_Report.pdf)

## Disclaimer

This project was conducted entirely within an isolated, air-gapped home lab for
educational and professional-development purposes. No production systems, third-party
infrastructure, or real organizations were targeted. The repository contains only
the final report and non-weaponized reference material — no compiled payloads or
executable artifacts are distributed here.

## Lab Setup

| Component | Specification | Role |
|---|---|---|
| Attacking Machine | Kali Linux (x86_64) | Payload generation, exploitation framework, listener |
| Target Host | Vulnerable Windows lab VM (x64) | Host with weak ACLs, unquoted paths, legacy third-party services |
| Initial Access | Standard low-privileged local user | Starting foothold for enumeration and privilege escalation |

## Vectors Assessed

| # | Vector | Category |
|---|---|---|
| 1 | DLL Search Order Hijacking | Service / library load-order abuse |
| 2 | Unquoted Service Paths | Service binary path interception |
| 3 | Modifiable Service Binaries (weak file ACLs) | Service file-permission weakness |
| 4 | Modifiable Service Configuration (weak service DACLs) | Service registry/config permission weakness |

## Tooling

- **Enumeration & Auditing:** `icacls`, `sc.exe`, `net localgroup`, `tasklist`, Sysinternals Suite (AccessChk, ProcMon, Process Explorer)
- **Automated LPE Checks:** SharpUp, PowerUp.ps1 (GhostPack collection)
- **Exploitation Framework:** Metasploit Framework (`multi/handler`, Meterpreter)

## Methodology

1. System enumeration (services, file/registry permissions, running processes)
2. Vulnerability identification per vector (manual + automated checks)
3. Controlled exploitation to validate privilege escalation to `SYSTEM`
4. Verification of elevated context (`getuid`)
5. Documentation of defensive remediation and hardening controls for each finding

## Key Findings Summary

| Finding | Severity | MITRE ATT&CK | Status |
|---|---|---|---|
| Modifiable service binary (weak folder ACL allowing binary replacement) | Critical | T1574.010 – Services File Permissions Weakness | Remediation documented |
| Weak service configuration DACL (binPath reconfigurable by standard users) | High | T1574.011 – Services Registry Permissions Weakness | Remediation documented |
| DLL Search Order Hijacking on custom Windows service | High | T1574.001 – DLL Search Order Hijacking | Remediation documented |
| Unquoted service path with embedded spaces | Medium | T1574.009 – Path Interception by Unqualified Path | Remediation documented |

*(Full severity rationale, evidence, and step-by-step remediation guidance are in the full report.)*

## Hardening Recommendations (Summary)

- Enforce strict file-system ACLs on service directories and binaries
- Quote all service binary paths containing spaces
- Harden service DACLs via Group Policy; audit with AccessChk
- Disable `AlwaysInstallElevated` in both HKLM and HKCU
- Deploy EDR/Sysmon monitoring for `sc.exe` reconfiguration, non-standard process spawns, and DLL loads from writable/temp directories
- Apply least privilege, LAPS, and CIS baseline hardening across the environment

See the full report for the complete hardening roadmap.

## Related Projects

- [MetalBin-S.A.-Lab-infrastructure-assessment](https://github.com/) — pfSense-segmented network pentest
- [Raptor-Hunty](https://github.com/) — Automated Metasploit + Nmap recon tool
- [Pentest-lab-metasploit-exploitation](https://github.com/) — Manual & automated exploitation lab
