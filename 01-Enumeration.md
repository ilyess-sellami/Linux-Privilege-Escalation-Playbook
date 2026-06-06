# 01. Enumeration

## Overview

Enumeration is the foundation of every Linux privilege escalation assessment. Before attempting exploitation, it is essential to understand the target system, identify potential attack surfaces, and discover misconfigurations that may lead to elevated privileges.

The objective of enumeration is to answer the following questions:

* Who am I?
* What privileges do I have?
* What users exist on the system?
* What services are running?
* What software is installed?
* What files and directories can I modify?
* What credentials or secrets are exposed?
* Are there any known privilege escalation vectors present?

A thorough enumeration process significantly increases the likelihood of discovering privilege escalation opportunities while reducing unnecessary noise and guesswork.

---

## Enumeration Methodology

A structured approach helps ensure that no important information is overlooked.

1. Collect basic system information.
2. Identify the current user and group memberships.
3. Enumerate other users and administrative accounts.
4. Inspect running processes and services.
5. Review network configurations and listening ports.
6. Search for sensitive files and credentials.
7. Examine file and directory permissions.
8. Check sudo privileges and SUID binaries.
9. Investigate scheduled tasks and automated jobs.
10. Identify potential kernel vulnerabilities.

---

## System Information

Understanding the operating system and kernel version is critical because many privilege escalation techniques depend on the specific Linux distribution and kernel release.

### Hostname

```bash
hostname
```

### Kernel Information

```bash
uname -a
```

### Distribution Information

```bash
cat /etc/os-release
```

### Architecture

```bash
arch

# or

uname -m
```

---

## User Enumeration

Determine the current user context and identify potential privilege assignments.

### Current User

```bash
whoami
```

### User and Group Information

```bash
id
```

### Logged-In Users

```bash
w

# or

who
```

### Local Users

```bash
cat /etc/passwd
```

Look for:

* Administrative accounts
* Service accounts
* Users with interactive shells
* Unexpected user accounts

### Password Hashes

Linux stores password hashes in ``/etc/shadow``, not in ``/etc/passwd``.

```bash
cat /etc/shadow
```

Common hash formats:

| Prefix                 | Algorithm |
| ---------------------- | --------- |
| `$1$`                  | MD5       |
| `$2a$`, `$2b$`, `$2y$` | bcrypt    |
| `$5$`                  | SHA-256   |
| `$6$`                  | SHA-512   |
| `$y$`                  | yescrypt  |

Look for:

* Password hashes accessible to non-root users
* Weak or legacy hash algorithms
* Additional privileged accounts
* Misconfigured permissions on `/etc/shadow`

---

## Environment Enumeration

Environment variables may contain credentials, tokens, paths, or application secrets.

### Display Environment Variables

```bash
env

# or

printenv
```

Pay attention to:

* PATH
* HOME
* USER
* AWS credentials
* API tokens
* Database credentials

---

## Process Enumeration

Running processes often reveal applications, services, credentials, and privilege escalation opportunities.

### Running Processes

```bash
ps aux
```

### Process Tree

```bash
ps auxf
```

Look for:

* Processes running as root
* Custom applications
* Scripts executed with elevated privileges
* Database services
* Monitoring agents

Example:

```bash
root  /usr/local/bin/backup.sh
```

Check:

```bash
ls -l /usr/local/bin/backup.sh
```

If writable:

```
echo "chmod +s /bin/bash" >> backup.sh
```

---

## Service Enumeration

Services frequently execute with elevated privileges and may contain insecure configurations.

### systemd Services

```bash
systemctl list-units --type=service
```

### Service Status

```bash
systemctl --type=service --state=running
```

Look for:

* Custom services
* Third-party applications
* Writable service files
* Services running as root

---

## Network Enumeration

Network services may expose management interfaces or reveal additional attack vectors.

### Network Interfaces

```bash
ip a
```

### Routing Table

```bash
ip route
```

### Listening Ports

```bash
ss -tunlp
```

or

```bash
netstat -tunlp
```

Look for:

* Locally exposed services
* Databases
* Administrative interfaces
* Internal-only applications

---

## File and Directory Enumeration

Insecure permissions frequently lead to privilege escalation.

### World-Writable Files

```bash
find / -type f -perm -0002 2>/dev/null
```

### World-Writable Directories

```bash
find / -type d -perm -0002 2>/dev/null
```

### Writable Files Owned by Root

```bash
find / -writable -user root 2>/dev/null
```

Look for:

* Configuration files
* Scripts
* Service binaries
* Backup files

---

## Automated Enumeration Tools

Automated tools can accelerate the discovery process but should never replace manual enumeration.

### LinPEAS

Comprehensive Linux privilege escalation enumeration tool.

### Linux Smart Enumeration (LSE)

Structured privilege escalation checks.

### LinEnum

Automated Linux enumeration script.

### LES (Linux Exploit Suggester)

Kernel exploit recommendation tool.

---

## Enumeration Checklist

Before moving to exploitation, verify that the following areas have been reviewed:

* System information
* Users and groups
* Environment variables
* Running processes
* Services
* Network configuration
* File permissions
* Credentials and secrets
* Sudo permissions
* SUID binaries
* Scheduled tasks
* Kernel version

Enumeration should always be performed methodically and repeatedly throughout an assessment, as new information obtained during the engagement may reveal previously overlooked privilege escalation paths.
