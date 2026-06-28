# Linux Privilege Escalation - Apache2 Sudo & SUID Enumeration

## Apache2 Sudo Misconfiguration

### Overview

If a user is allowed to execute `apache2` with `sudo`, Apache can be instructed to use an alternate configuration file using the `-f` option. Apache parses the supplied file as a configuration file, which may result in information disclosure through parser errors.

---

### Prerequisites

- Linux Target
- `sudo` access to `apache2`
- Ability to execute:

```bash
sudo -l
```

---

### Enumeration

Check sudo permissions:

```bash
sudo -l
```

Look for entries similar to:

- `apache2`
- `/usr/sbin/apache2`
- `NOPASSWD` permissions

---

### Apache Options

Display available options:

```bash
apache2 -h
```

Useful options:

| Option | Description |
|---------|-------------|
| `-f` | Specify an alternate ServerConfigFile |
| `-C` | Process directives before reading the configuration |
| `-c` | Process directives after reading the configuration |

---

### Command

```bash
sudo apache2 -C "LoadModule mpm_event_module /usr/lib/apache2/modules/mod_mpm_event.so" -f /etc/shadow
```

---

### Why the `LoadModule` Directive?

Apache requires a Multi-Processing Module (MPM) before parsing configuration files.

The `-C` option loads the required module prior to reading the supplied configuration file.

---

### Required Tools

- sudo
- apache2

---

### Important Points

- Always begin with `sudo -l`.
- Verify that `apache2` is permitted to run as root.
- The `-f` option replaces Apache's default configuration file.
- Apache interprets the supplied file as a configuration file.
- Parsing failures may disclose portions of the file.
- Behaviour varies depending on Apache version and Linux distribution.
- This method depends on insecure sudo configuration rather than an Apache vulnerability.

---

# SUID Enumeration

## Overview

SUID (Set User ID) is a special file permission that allows an executable to run with the privileges of the file owner instead of the user executing it.

If a binary owned by `root` has the SUID bit set, it executes with root privileges.

---

## Enumeration

Search for SUID binaries:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

---

## Permission Representation

Example:

```text
-rwsr-xr-x
```

The **`s`** replacing the owner's execute bit indicates that the SUID bit is enabled.

---

## Verification

Display permissions of a binary:

```bash
ls -l /path/to/binary
```

---

## Common Enumeration Tools

- find
- ls
- stat

---

## GTFOBins

After identifying SUID binaries, compare them against GTFOBins to determine whether they have documented privilege escalation techniques.

https://gtfobins.github.io/

---

## Important Points

- Enumerate all SUID binaries during every privilege escalation assessment.
- Pay special attention to uncommon or custom binaries.
- Third-party applications frequently introduce insecure SUID binaries.
- Standard Linux binaries are not always exploitable even if SUID is set.
- Custom-developed binaries deserve additional analysis.
- Compare discovered binaries with GTFOBins before investigating manually.
- File ownership is important—root-owned SUID binaries present the greatest privilege escalation risk.
- Some distributions intentionally set SUID on certain binaries; validate whether their behavior is expected.

---

# Quick Enumeration Checklist

## Sudo

```bash
sudo -l
```

## Apache Help

```bash
apache2 -h
```

## SUID Enumeration

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

## Verify Permissions

```bash
ls -l <binary>
```

---

# References

- Apache HTTP Server Documentation
- GTFOBins
- Linux File Permissions Documentation
