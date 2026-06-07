# Linux Privilege Escalation Playbook

A practical guide to understanding, identifying, and exploiting Linux privilege escalation vectors commonly encountered during penetration tests, security assessments, labs, and Capture The Flag (CTF) challenges.

This repository focuses on the methodology, techniques, and misconfigurations that can lead to privilege escalation on Linux systems. Each module covers the underlying concepts, enumeration techniques, exploitation methods, detection opportunities, and mitigation strategies to provide both offensive and defensive perspectives.

## Modules

### [01. Enumeration](#01-enumeration)

Learn how to systematically enumerate Linux systems, users, services, processes, network configurations, permissions, and potential privilege escalation vectors.

### [02. Credentials and Secrets](#02-credentials-and-secrets)

Identify and analyze exposed credentials, configuration files, SSH keys, environment variables, application secrets, and other sensitive information that may lead to privilege escalation.

### [03. Sudo Misconfigurations](#03-sudo-misconfigurations)

Understand sudo privileges, dangerous sudo rules, GTFOBins abuse, wildcard injection, and common misconfigurations that allow privilege escalation.

### [04. SUID and Capabilities Abuse](#04-suid-and-capabilities-abuse)

Explore SUID binaries, Linux capabilities, custom privileged executables, and techniques for abusing misconfigured permissions to gain elevated access.

### [05. Scheduled Tasks and Jobs](#05-scheduled-tasks-and-jobs)

Investigate cron jobs, at jobs, writable scripts, PATH hijacking opportunities, and task misconfigurations that can be leveraged for privilege escalation.

### [06. Services and Process Abuse](#06-services-and-process-abuse)

Analyze system services, systemd configurations, writable service files, environment variable abuse, and process-related privilege escalation opportunities.


### [07. Kernel Exploitation](#07-kernel-exploitation)

Understand kernel vulnerability identification, exploit selection, version verification, and the risks associated with kernel-level privilege escalation techniques.
