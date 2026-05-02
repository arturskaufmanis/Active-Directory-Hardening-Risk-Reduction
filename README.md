# Active Directory Hardening & Risk Reduction — PingCastle Assessment

## Overview

This project documents a structured Active Directory security hardening exercise conducted on a Windows Server 2025 homelab domain (`adforest.local`). Using PingCastle as the primary assessment tool, the domain risk score was reduced from **60 → 35 → 20** across three assessment cycles through targeted remediation of identified findings.

The project demonstrates practical skills in AD security assessment, risk prioritisation, GPO-based hardening, LAPS deployment, backup compliance, and portfolio-grade documentation — directly relevant to SOC Analyst and IT/Security Administrator roles.

---

## Environment

| Component | Detail |
|---|---|
| Domain | adforest.local |
| Domain Controller | WIN-ESVD1CAD1FJ |
| OS | Windows Server 2025 |
| Domain Functional Level | Windows Server 2016 (Level 10) |
| Forest Functional Level | Windows Server 2016 (Level 10) |
| Schema Version | 91 |
| Assessment Tool | PingCastle Free Edition |

---

## Score Progression

| Assessment | Global Score | Stale Objects | Privileged Accounts | Trusts | Anomalies |
|---|---|---|---|---|---|
| Baseline (Report 2) | 60 | — | — | — | — |
| Post-Hardening (Report 12) | 35 | 20 | 20 | 0 | 35 |
| Final (Report 13) | **20** | 20 | **0** | 0 | 20 |

> Lower is better. PingCastle scores 0–100 per category; the Global Score equals the highest category score.

---

## Hardening Actions Taken

### 1. Password Policy
- Enforced minimum password length and complexity via Default Domain Policy GPO
- Eliminated weak or default password settings flagged in baseline scan

### 2. NTLMv1 Disabled
- Configured `LAN Manager authentication level` to send NTLMv2 only and refuse LM/NTLMv1
- Applied via GPO: `Computer Configuration → Windows Settings → Security Settings → Local Policies → Security Options`

### 3. AES Encryption Enforced
- Configured Kerberos to use AES128/AES256 encryption types only
- Removed RC4/DES from supported encryption types on privileged accounts

### 4. Windows LAPS Deployed
- Deployed Windows LAPS (Local Administrator Password Solution) via GPO
- Configured automatic password rotation for local administrator accounts
- Passwords stored securely in AD and retrievable only by authorised accounts

### 5. AD Recycle Bin Enabled
- Enabled AD Recycle Bin via Active Directory Administrative Center
- Provides object recovery without authoritative restore

### 6. MachineAccountQuota Set to 0
- Set `ms-DS-MachineAccountQuota` to 0 on the domain root
- Prevents non-privileged users from joining machines to the domain

### 7. Privileged Account Structure
- Created dedicated `SecAdmin` account for administrative tasks
- Renamed built-in Administrator account to `BreakGlass` and disabled it
- Separated day-to-day and privileged access following least-privilege principles

### 8. AD Backup Compliance (A-BackupMetadata)
- Identified that no Microsoft-standard AD backup had been performed (+15 points)
- Created a 40GB Dynamic VHD (`C:\BackupVHD\adbackup.vhd`) mounted as drive D:
- Executed `wbadmin start systemstatebackup -backuptarget:D:` — completed successfully 02/05/2026 13:11
- DIT Database Partition Backup Signature updated — finding cleared on next scan

---

## Findings Remaining (Score 20)

| Category | Score | Notes |
|---|---|---|
| Stale Objects | 20 | 8 rules matched — includes inactive accounts and computer objects requiring cleanup |
| Anomalies | 20 | 11 rules matched — includes DNS record permissions, PowerShell audit policy |
| Privileged Accounts | 0 | 1 residual rule — near-clean |
| Trusts | 0 | No trust relationships configured |

### Notable Remaining Findings
- **Authenticated Users can create DNS records** — medium risk, mitigable via DNS ACL hardening
- **PowerShell audit configuration not fully enabled** — ScriptBlock and Module logging not fully configured
- **DSHeuristics CVE-2021-42291 mitigation** — not yet applied

---

## AD Backup Architecture

```
C:\
├── BackupVHD\
│   └── adbackup.vhd     ← 40GB Dynamic VHD (mounted as D:)
└── (ADBackup\ removed)  ← Old non-standard backup, deleted post-migration

D: (ADBackup VHD)
└── WindowsImageBackup\  ← wbadmin system state backup
```

**Why VHD?** `wbadmin` requires a separate volume from the source being backed up. A dynamically expanding VHD mounted as a separate drive letter satisfies this requirement without needing additional physical storage.

---

## Tools Used

| Tool | Purpose |
|---|---|
| PingCastle Free Edition | AD health and risk assessment |
| Group Policy Management Console | GPO creation and linking |
| Active Directory Users and Computers | Account management |
| Windows LAPS | Local admin password management |
| wbadmin | System state backup |
| Hyper-V PowerShell Module | VHD creation and management |
| PowerShell | Automation and configuration |

---

## Files in This Repository

| File | Description |
|---|---|
| `README.md` | This document |
| `pingcastle-report-baseline.pdf` | PingCastle report — pre-hardening (score 60) |
| `pingcastle-report-intermediate.pdf` | PingCastle report — post initial hardening (score 35) |
| `pingcastle-report-final.pdf` | PingCastle report — final hardened state (score 20) |
| `AD-Hardening-Portfolio.docx` | Detailed technical write-up |

---

## Key Takeaways

- PingCastle provides an objective, repeatable metric for AD security posture — ideal for demonstrating measurable improvement
- A 67% score reduction (60 → 20) was achieved through configuration hardening alone, with no third-party tooling beyond LAPS
- The backup compliance finding illustrates how infrastructure gaps (not just misconfiguration) contribute to risk scoring
- VHD-based backup targets are a practical solution for single-disk lab environments where wbadmin volume restrictions apply

---

*Part of a cybersecurity homelab portfolio targeting SOC Analyst and IT/Security Administrator roles.*
*Domain: adforest.local | DC: WIN-ESVD1CAD1FJ | Assessment date: 02/05/2026*
