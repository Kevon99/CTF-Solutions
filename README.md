# CTF-Solutions

A personal, hands-on collection of professional-grade write-ups documenting complete exploitation paths from TryHackMe, Hack The Box, DockerLabs and other CTF platforms. Each report follows a structured pentesting methodology — reconnaissance, enumeration, exploitation, privilege escalation — including command-by-command breakdowns, tool usage rationale, and remediation recommendations.

---

## Index

### Active Directory

| Machine | Platform | Key Techniques | Write-up |
|---------|----------|----------------|----------|
| **Operation Endgame** | TryHackMe | SMB Guest session · RID Cycling · Kerberoasting · Password spray (credential reuse) · ACL abuse (`GenericWrite` → forced AS-REP Roasting) · Plaintext credential discovery | [Report](active_directory/operation_endgame/Operation%20Endgame%20THM.md) |
| **Soupedecode 01** | TryHackMe | Null session SMB · RID Cycling · Username-as-password spray · Kerberoasting · Backup file disclosure (NTLM hashes) · Pass-the-Hash | [Report](active_directory/soupedecode01/soupedecode01.md) |
| **Support** | Hack The Box | Unauthenticated footprinting · LDAP enumeration · Credential hunting · Domain privilege escalation | [Report](active_directory/support/Support%20%28%20Hack%20the%20box%20%29%20-%20Professional%20Writeup.md) |

### Web & Services Exploitation

| Machine | Platform | Key Techniques | Write-up |
|---------|----------|----------------|----------|
| **Internal** | TryHackMe | WordPress compromise · Jenkins (Dockerized) pivoting · Credential extraction via `/proc` · Container escape / Linux privesc | [Report](Jenkins/internal_thm/Internal-TryHackMe.md) |
| **Watcher** | TryHackMe | Local File Inclusion (LFI) · FTP enumeration · Shell stabilization · Linux privilege escalation | [Report](LFI/watcher/watcher.md) |
| **SocialHub** | DockerLabs | Stored XSS · Authentication bypass workflow · Linux privilege escalation | [Report](XSS/socialhub/socialhub%20dockerlabs.md) |
| **0day** | TryHackMe | Directory brute-forcing (ffuf) · Vulnerability scanning (Nikto) · Public exploit adaptation · Shell stabilization · Kernel/Linux privesc | [Report](linux/0day-TryHackMe/0day.md) |

### Kubernetes & Cloud-Native

| Machine | Platform | Key Techniques | Write-up |
|---------|----------|----------------|----------|
| **Fireflow** | Hack The Box | HTTPS/NGINX enumeration · Parameter manipulation exploitation · Reverse shell delivery · Privilege scaling · Admin panel abuse | [Report](kubernetes/fireflow/fireflow.md) |

> Some machines appear in more than one category folder (e.g., `linux/internal`, `wordpress/internal`, `Jenkins/internal_thm`) because each copy highlights the write-up from a different technical angle. The table above links to the primary version of each report.

---

## Methodology

Every write-up documents the full attack chain:

1. **Reconnaissance** — full-port SYN scans and service/version fingerprinting with Nmap
2. **Enumeration** — manual inspection plus automated discovery (Gobuster, FFUF, WhatWeb)
3. **Exploitation** — validated attack paths with Impacket, NetExec, custom scripts and public exploits
4. **Privilege Escalation** — Linux (LinPEAS, SUID, cron, containers) and Windows/AD (Kerberoasting, AS-REP Roasting, ACL abuse, credential reuse, Pass-the-Hash)
5. **Findings & Remediation** — impact analysis and defensive recommendations

---

## Repository Structure

```text
CTF-Solutions/
├── active_directory/
│   ├── operation_endgame/
│   ├── soupedecode01/
│   └── support/
├── Jenkins/internal_thm/
├── wordpress/internal/
├── LFI/watcher/
├── XSS/socialhub/
├── kubernetes/fireflow/
└── linux/
    ├── 0day-TryHackMe/
    ├── fireflow/
    ├── internal/
    ├── socialhub/
    └── watcher/
```

---

*All techniques were executed against authorized lab infrastructure for educational purposes.*
