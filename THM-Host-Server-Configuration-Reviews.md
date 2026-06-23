# Host-Server Configuration Reviews (TryHackMe)

**Date:** 13 June 2026

## Overview

Completed the **Host-Server Configuration Reviews** room on TryHackMe. This room introduced the fundamentals of configuration-based privilege escalation, security baselines, compliance frameworks, and systematic host enumeration methodologies used during penetration testing.

## Key Concepts Learned

### Configuration-Based vs Vulnerability-Based Privilege Escalation

* Vulnerability-based escalation exploits software flaws and CVEs.
* Configuration-based escalation abuses insecure system settings and misconfigurations.
* A fully patched system can still be vulnerable due to poor configurations.

### Security Baselines

Learned about industry-standard hardening frameworks:

* CIS Benchmarks
* DISA STIGs
* NIST 800-53
* PCI-DSS
* ISO 27001

These frameworks provide secure configuration standards for operating systems, applications, and network devices.

### Configuration Review Tools

Explored tools commonly used for auditing and enumeration:

* Nessus
* Lynis
* OpenSCAP
* CIS-CAT
* LinPEAS
* WinPEAS
* PowerUp

### Common Misconfiguration Categories

#### User & Group Configuration

* Excessive administrative privileges
* Weak password policies
* Unnecessary privileged accounts
* Default accounts left enabled

#### File & Directory Permissions

* Misconfigured SUID/SGID binaries
* World-writable files and directories
* Sensitive files with excessive permissions
* Weak Windows ACLs

#### Service Configurations

* Services running with excessive privileges
* Writable service binaries or configuration files
* Unquoted service paths on Windows

#### Scheduled Tasks & Cron Jobs

* Privileged tasks executing writable scripts
* Cron job misconfigurations
* Wildcard injection vulnerabilities

#### Credential Storage

* Credentials stored in configuration files
* SSH keys with weak permissions
* Passwords in shell history
* Windows Credential Manager abuse
* Exposed deployment files and registry secrets

#### Network Configuration

* Unnecessary exposed services
* Weak firewall rules
* Services bound to all interfaces
* Insecure remote management protocols

### Enumeration Methodology

#### Phase 1 – Situational Awareness

* Identify current user and privileges
* Determine OS version and architecture
* Understand host role and environment
* Check domain membership

#### Phase 2 – Category-Based Enumeration

* Users and groups
* File permissions
* Services
* Scheduled tasks
* Credential storage
* Network configuration

#### Phase 3 – Prioritisation & Exploitation

* Prioritise findings based on exploitability
* Identify chains of related misconfigurations
* Focus on direct privilege escalation paths

## Key Takeaways

* Misconfigurations remain one of the most common privilege escalation vectors.
* Security baselines provide a structured way to identify weaknesses.
* Configuration reviews differ from vulnerability scans and exploit development.
* Systematic enumeration is critical for finding escalation opportunities.
* Automated tools help speed up assessments but should never replace manual analysis.

## Conclusion

This room provided a solid foundation for understanding host and server configuration reviews. It reinforced the importance of structured enumeration and demonstrated how configuration weaknesses can lead to privilege escalation even on fully patched systems.
