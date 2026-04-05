# 🔒 Akira Ransomware — Threat Analysis & Deep Dive

> **A comprehensive cybersecurity research report on the Akira Ransomware group, covering its RaaS model, attack chain, MITRE ATT&CK mapping, Cyber Kill Chain analysis, and defensive mitigations.**

---

## 📖 Introduction

**Akira Ransomware** is a cybercriminal group and ransomware family first discovered in **March 2023**. Operating under a **Ransomware-as-a-Service (RaaS)** model, the group rapidly gained notoriety for its professional operations, double extortion tactics, and ability to attack both **Windows** and **Linux/VMware ESXi** systems.

The group encrypts victims' data and demands ransom in **Bitcoin**, threatening to publish stolen data on their dark web leak site if payment is refused. Encrypted files receive the **`.akira`** extension, and victims receive a ransom note named **`akira_readme.txt`**.

> 📄 **Full Technical Report:** [`Akira_Ransomware_Analysis.docx`](./Akira_Ransomware_Analysis.docx)

---

## 📊 Threat Profile at a Glance

| Field | Details |
|---|---|
| **First Observed** | March 2023 |
| **Type** | Ransomware-as-a-Service (RaaS) |
| **Extortion Model** | Double Extortion (encrypt + steal + leak) |
| **Target OS** | Windows, Linux, VMware ESXi |
| **Encryption** | AES (symmetric) + RSA (asymmetric) |
| **File Extension** | `.akira` (later variants: `.powerranges`) |
| **Ransom Note** | `akira_readme.txt` |
| **Payment Method** | Bitcoin (BTC) |
| **Primary Targets** | SMEs: Education, Finance, Healthcare, Manufacturing, IT |
| **Suspected Origin** | Russian-speaking group (ex-Conti members suspected) |
| **Estimated Earnings** | Tens of millions USD |

---

## 🔗 The RaaS Model

```
┌──────────────────────────────────────────────────────────────┐
│                   AKIRA RaaS ECOSYSTEM                       │
├──────────────────────────┬───────────────────────────────────┤
│    CORE DEVELOPERS       │       AFFILIATES                  │
│                          │                                   │
│ • Build & maintain       │ • Execute actual attacks          │
│   ransomware             │ • Choose initial access method    │
│ • Manage leak site       │ • Perform lateral movement        │
│ • Handle negotiations    │ • Exfiltrate data                 │
│ • Update evasion tools   │ • Deploy ransomware               │
│                          │                                   │
│         ◄────── Revenue Share Split ──────►                  │
└──────────────────────────┴───────────────────────────────────┘
```

---

## ⏱️ Evolution Timeline

| Period | Event |
|---|---|
| **March 2023** | Akira first observed; targets Windows systems via RaaS model |
| **Early 2023** | Conti connection identified; code/financial overlaps discovered |
| **April–June 2023** | Linux/VMware ESXi variant deployed; cross-platform expansion |
| **July 2023** | Free decryptor released by Avast for original variant |
| **August 2023** | "Megazord" (Akira v2) deployed in Rust to defeat the decryptor |
| **Sept–Oct 2023** | Heavy exploitation of **CVE-2023-20269** (Cisco ASA/FTD VPN) |
| **2023–2024** | Hundreds of victims claimed; SME-focused targeting in US, EU, Australia |

---

## 🔄 Attack Chain — Step by Step

```
[1] INITIAL ACCESS
    └─► Exploit VPN vulnerabilities (Cisco CVE-2023-20269)
    └─► Stolen credentials / brute-force
    └─► Spear-phishing emails
          │
          ▼
[2] PRIVILEGE ESCALATION & LATERAL MOVEMENT
    └─► PowerShell, Cobalt Strike, Mimikatz
    └─► LSASS memory dumping
    └─► RDP lateral movement
    └─► Advanced IP Scanner / AdFind for network mapping
          │
          ▼
[3] DATA EXFILTRATION
    └─► WinRAR / FileZilla / Rclone / WinSCP
    └─► Data staged on dark web leak site as leverage
          │
          ▼
[4] DEFENSE EVASION
    └─► Disable Windows Defender
    └─► Delete Volume Shadow Copies (VSS)
    └─► Modify registry keys
          │
          ▼
[5] ENCRYPTION
    └─► AES + RSA hybrid encryption
    └─► Files renamed: report.xlsx → report.xlsx.akira
    └─► akira_readme.txt dropped in every folder
          │
          ▼
[6] DOUBLE EXTORTION
    └─► Demand ransom to decrypt files
    └─► Threaten to publish stolen data on dark web
```

---

## 🗺️ MITRE ATT&CK Mapping

### 1. Initial Access (TA0001)

| Technique ID | Technique Name | Description |
|---|---|---|
| T1190 | Exploit Public-Facing Application | Cisco CVE-2020-3259, CVE-2023-20269 — VPN without MFA |
| T1133 | External Remote Services | Compromised VPN accounts |
| T1078 | Valid Accounts | Stolen credentials for remote access |
| T1566.001/.002 | Phishing | Spear-phishing emails with attachments or links |

### 2. Persistence & Privilege Escalation (TA0003 / TA0004)

| Technique ID | Technique Name | Description |
|---|---|---|
| T1136.002 | Create Account: Domain Account | New domain accounts (e.g., `itadm`) |
| T1003.001 | OS Credential Dumping: LSASS Memory | Mimikatz / LaZagne |
| T1558 | Steal or Forge Kerberos Tickets | Kerberoasting |
| T1219 | Remote Access Software | AnyDesk, PuTTy for persistent access |

### 3. Discovery & Lateral Movement (TA0007 / TA0008)

| Technique ID | Technique Name | Description |
|---|---|---|
| T1018 | Remote System Discovery | Advanced IP Scanner, MASSCAN |
| T1482 | Domain Trust Discovery | Nltest, AdFind |
| T1021.001 | Remote Services: RDP | RDP with stolen credentials |
| T1570 | Lateral Tool Transfer | Transferring ransomware payload across network |

### 4. Defense Evasion & C2 (TA0005 / TA0011)

| Technique ID | Technique Name | Description |
|---|---|---|
| T1562.001 | Impair Defenses: Disable Tools | Disable Windows Defender |
| T1059.001 | Command and Scripting Interpreter: PowerShell | Delete VSS copies |
| T1112 | Modify Registry | Disable security features, hide accounts |

### 5. Exfiltration & Impact (TA0010 / TA0040)

| Technique ID | Technique Name | Description |
|---|---|---|
| T1560.001 | Archive Collected Data: via Utility | WinRAR for archiving before exfil |
| T1567.002 | Exfiltration Over Web Service: Cloud Storage | Rclone, WinSCP/SFTP |
| T1486 | Data Encrypted for Impact | ChaCha20/RSA encryption |
| T1490 | Inhibit System Recovery | Delete Volume Shadow Copies |
| T1531 | Account Access Removal | Delete admin accounts before encryption |

---

## ⚔️ Cyber Kill Chain Mapping

| Phase | Akira TTPs | Description |
|---|---|---|
| **1. Reconnaissance** | Target selection based on sector/region | Focus on SMEs with high ransom potential |
| **2. Weaponization** | Package Akira/Megazord payload + post-exploit tools | Custom ransomware + remote access kit |
| **3. Delivery** | VPN exploits, credential stuffing, spear-phishing | CVE-2023-20269, RDP with stolen creds |
| **4. Exploitation** | Execute code via VPN flaw; gain initial foothold | Cisco ASA/FTD exploitation |
| **5. Installation** | AnyDesk/RustDesk; new domain accounts | Persistent remote access |
| **6. C2** | AnyDesk, Ngrok, Cloudflare Tunnels | Remote command and control |
| **7. Actions on Objectives** | Data theft + encryption + double extortion | `.akira` extension; ransom note dropped |

---

## 🧬 Malware Variants

| Variant | Period | Language | Notes |
|---|---|---|---|
| **Akira v1** | March 2023 | C++ | Original; decryptor released by Avast (July 2023) |
| **Megazord / Akira v2** | August 2023+ | Rust | Response to public decryptor; uses `.powerranges` extension |
| **Linux / ESXi** | April–June 2023 | C++ | Targets VMware ESXi hypervisors |

---

## 🛡️ Defensive Recommendations

### Immediate Priority Controls

| Control | Action |
|---|---|
| **MFA on VPNs** | Enforce MFA on all remote access / VPN endpoints — most Akira intrusions exploit MFA-less VPNs |
| **Patch VPN Appliances** | Immediately patch Cisco ASA/FTD and other VPN products (CVE-2023-20269, CVE-2020-3259) |
| **Disable Unused RDP** | Restrict or disable RDP where not needed; use jump hosts |
| **EDR Deployment** | Deploy Endpoint Detection & Response; monitor LSASS access |
| **Backup Isolation** | Maintain offline/air-gapped backups; Akira deletes VSS copies |
| **Network Segmentation** | Limit lateral movement with Zero Trust micro-segmentation |

### Detection Signatures

```
# Volume Shadow Copy Deletion (PowerShell)
Get-WmiObject Win32_Shadowcopy | Remove-WmiObject

# Akira file extension
*.akira (encrypted files)
akira_readme.txt (ransom note)

# Suspicious tools to monitor
- AnyDesk, RustDesk (remote access)
- Rclone, FileZilla, WinSCP (exfiltration)
- Advanced IP Scanner, Netscan (discovery)
- Mimikatz, LaZagne (credential theft)
```

---

## 📁 Repository Contents

```
akira-ransomware/
├── README.md                     ← This file
└── Akira_Ransomware_Analysis.docx ← Full technical report
```

---

## 📚 Key Takeaways

1. **MFA is non-negotiable** — the majority of Akira intrusions begin with MFA-less VPN access.
2. **RaaS lowers the barrier** — affiliates with limited skill can execute devastating attacks using Akira's toolkit.
3. **Double extortion doubles the pressure** — encryption alone is no longer the only threat; data theft precedes it.
4. **Adapt or lose** — Akira released Megazord within weeks of a public decryptor, showing rapid operational adaptability.
5. **Backups alone aren't enough** — VSS deletion means local and cloud-connected backups can be destroyed; offline copies are critical.

---

## 🔗 References

- [CISA Advisory AA23-187A — Akira Ransomware](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-187a)
- [Avast Akira Decryptor (July 2023)](https://www.avast.com/ransomware-decryption-tools)
- [Cisco CVE-2023-20269](https://nvd.nist.gov/vuln/detail/CVE-2023-20269)
- [MITRE ATT&CK — Akira](https://attack.mitre.org/software/S1129/)

---

> **Disclaimer:** This repository is for educational and research purposes only. All analysis is based on publicly available information.
