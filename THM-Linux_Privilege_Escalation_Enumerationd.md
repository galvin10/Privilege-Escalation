# Linux Privilege Escalation: Enumeration

**Platform:** TryHackMe  
**Room:** Linux Privilege Escalation: Enumeration  
**Completion Date:** 22 June 2026  
**Category:** Linux Privilege Escalation  
**Difficulty:** Easy - Intermediate  
**Status:** Completed ✅

---

## Overview

Enumeration is the most important phase of privilege escalation. The objective is to gather information about the operating system, users, services, processes, scheduled tasks, installed software, network configuration, and file permissions to identify potential privilege escalation vectors.

---

# Operating System Enumeration

## hostname

### Purpose
Displays the hostname of the machine.

### Command

```bash
hostname
```

### Why It Matters
The hostname can reveal the role of the machine such as:

- WEB01
- SQL01
- DEV-SERVER

These names often provide insight into the target's purpose.

---

## uname

### Purpose
Displays kernel and system information.

### Command

```bash
uname -a
```

### Why It Matters
Used to identify:

- Kernel Version
- Architecture
- Potential Kernel Exploits

---

## /proc/version

### Purpose
Displays detailed kernel build information.

### Command

```bash
cat /proc/version
```

### Why It Matters

Provides:

- Kernel version
- Compiler information
- Build environment

Useful when researching kernel vulnerabilities.

---

## /etc/issue

### Purpose
Displays Linux distribution information.

### Command

```bash
cat /etc/issue
```

### Why It Matters

Helps identify:

- Ubuntu
- Debian
- CentOS
- Kali

Different distributions may have different vulnerabilities.

---

# Process Enumeration

## ps

### Purpose
View running processes.

### Command

```bash
ps
```

### Why It Matters

Shows processes associated with the current shell.

---

## ps aux

### Purpose
Display all running processes.

### Command

```bash
ps aux
```

### Why It Matters

Useful for identifying:

- Running services
- Applications
- Potentially vulnerable software

---

## ps axjf

### Purpose
Displays processes in a tree structure.

### Command

```bash
ps axjf
```

### Why It Matters

Helps understand:

- Parent-child process relationships
- Service hierarchy

---

# Cron Job Enumeration

## View System Cron Jobs

### Command

```bash
cat /etc/crontab
```

### Why It Matters

Cron jobs often run with elevated privileges.

Misconfigured cron jobs can lead to:

- Privilege Escalation
- Command Injection

---

## Additional Cron Locations

```bash
ls -la /etc/cron.d
ls -la /var/spool/cron
```

### Purpose

Identify scheduled tasks configured by users or administrators.

---

# Installed Packages Enumeration

## dpkg

### Purpose

Lists installed packages.

### Command

```bash
dpkg -l
```

### Why It Matters

Helps identify:

- Outdated packages
- Vulnerable software
- Exploitable applications

---

# User Enumeration

## id

### Purpose

Displays user and group information.

### Command

```bash
id
```

### Why It Matters

Shows:

- User ID
- Group Memberships
- Administrative Groups

---

## env

### Purpose

Display environment variables.

### Command

```bash
env
```

### Why It Matters

Useful variables:

```bash
PATH
HOME
USER
SHELL
```

Can reveal:

- Credentials
- PATH Hijacking opportunities

---

## history

### Purpose

Displays command history.

### Command

```bash
history
```

### Why It Matters

May reveal:

- Passwords
- Administrative commands
- Sensitive operations

---

## sudo -l

### Purpose

Shows commands executable via sudo.

### Command

```bash
sudo -l
```

### Why It Matters

One of the most common privilege escalation vectors.

Always check this command first.

---

# User Discovery

## /etc/passwd

### Purpose

List local users.

### Command

```bash
cat /etc/passwd
```

---

## Extract Usernames

```bash
cat /etc/passwd | cut -d ":" -f1
```

### Purpose

Quickly generate a list of usernames.

---

## Find Real Users

```bash
cat /etc/passwd | grep /home
```

### Purpose

Filter out service accounts.

---

# Network Enumeration

## ifconfig

### Purpose

Display network interfaces.

### Command

```bash
ifconfig
```

---

## Modern Alternative

```bash
ip addr
```

### Why It Matters

Helps identify:

- Internal Networks
- VPN Interfaces
- Docker Networks

Useful for pivoting.

---

# Open Ports and Services

## netstat -a

### Purpose

Show all connections.

### Command

```bash
netstat -a
```

---

## netstat -lt

### Purpose

Show listening TCP ports.

### Command

```bash
netstat -lt
```

---

## netstat -tpln

### Purpose

Display listening services with process IDs.

### Command

```bash
netstat -tpln
```

### Why It Matters

Identify:

- Internal services
- Databases
- Web servers
- LDAP

---

## netstat -tp

### Purpose

Display active network connections.

### Command

```bash
netstat -tp
```

---

## netstat -s

### Purpose

Display network statistics.

### Command

```bash
netstat -s
```

---

## netstat -i

### Purpose

Display interface statistics.

### Command

```bash
netstat -i
```

---

## netstat -ano

### Purpose

Comprehensive network overview.

### Command

```bash
netstat -ano
```

---

## Modern Alternative

```bash
ss -tpl
```

---

# File Enumeration

## ls -la

### Purpose

Display all files including hidden files.

### Command

```bash
ls -la
```

### Why It Matters

Hidden files often contain:

- Credentials
- Configurations
- Tokens

---

# Finding Files

## Search Specific File

```bash
find . -name flag1.txt
```

```bash
find /home -name flag1.txt
```

---

## Find Directories

```bash
find / -type d -name config
```

---

## Find World-Writable Files

```bash
find / -type f -perm 0777
```

### Why It Matters

World-writable files can often be abused.

---

## Find Executable Files

```bash
find / -perm -a=x
```

---

## Find Files Owned By User

```bash
find /home -user frank
```

---

## Recently Modified Files

```bash
find / -mtime -10
```

---

## Recently Accessed Files

```bash
find / -atime -10
```

---

## Recently Changed Files

```bash
find / -cmin -60
```

---

## Recently Accessed Within Hour

```bash
find / -amin -60
```

---

## Large Files

```bash
find / -size +50M
```

```bash
find / -size +100M
```

### Why It Matters

Large files often contain:

- Backups
- Archives
- Databases

---

## Ignore Errors

```bash
find / -size +100M 2>/dev/null
```

### Purpose

Suppress permission denied errors.

---

# Writable Directories

## World Writable Directories

```bash
find / -writable -type d 2>/dev/null
```

```bash
find / -perm -222 -type d 2>/dev/null
```

```bash
find / -perm -o w -type d 2>/dev/null
```

### Why It Matters

Writable directories are frequently used during privilege escalation.

---

# Development Tools Discovery

## Perl

```bash
find / -name perl*
```

## Python

```bash
find / -name python*
```

## GCC

```bash
find / -name gcc*
```

### Why It Matters

These tools can be used to:

- Compile exploits
- Execute reverse shells
- Create payloads

---

# SUID Enumeration

## Find SUID Binaries

### Command

```bash
find / -perm -u=s -type f 2>/dev/null
```

### Why It Matters

SUID binaries execute with the owner's privileges.

Misconfigured SUID binaries are one of the most common Linux privilege escalation vectors.

Reference:
https://gtfobins.github.io/

---

# Enumeration Checklist

## System

```bash
hostname
uname -a
cat /proc/version
cat /etc/issue
```

## Users

```bash
id
env
history
sudo -l
cat /etc/passwd
```

## Processes

```bash
ps aux
ps axjf
```

## Scheduled Tasks

```bash
cat /etc/crontab
ls -la /etc/cron.d
```

## Network

```bash
ip addr
netstat -tpln
ss -tpl
```

## Files

```bash
ls -la
find / -perm -u=s -type f 2>/dev/null
find / -writable -type d 2>/dev/null
find / -size +100M 2>/dev/null
```

---

# Key Takeaways

- Enumeration is the foundation of privilege escalation.
- Always enumerate before attempting exploitation.
- Check sudo permissions first.
- Search for SUID binaries.
- Review cron jobs.
- Enumerate running services.
- Inspect hidden files.
- Look for writable directories.
- Identify vulnerable software versions.

---

**Room Completed:** 22 June 2026 ✅  
**Platform:** TryHackMe  
**Room:** Linux Privilege Escalation: Enumeration
