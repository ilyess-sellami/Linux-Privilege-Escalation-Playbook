# 02. Credentials and Secrets

## Overview

Credential discovery is one of the most reliable and high-impact privilege escalation techniques in Linux environments. Unlike kernel exploits or complex misconfigurations, credentials are often unintentionally exposed through files, logs, environment variables, or application configurations.

The objective of this module is to identify where credentials are commonly stored, how they are discovered during enumeration, and how they can be leveraged for privilege escalation.

---

## Common Credential Locations

### 1. Configuration Files

Applications often store credentials in plain text configuration files.

#### Example locations:

```bash
/etc/
/opt/
/var/www/
/home/user/app/
/srv/
```

#### Search for credentials:

```bash
grep -Ri "password" /etc /opt /var/www 2>/dev/null
```

#### Example vulnerable file:

```text
/var/www/html/config.php
DB_PASSWORD="admin123"
```

➡️ Exploitation:

* Reuse password for `sudo`
* SSH login as another user
* Database access → file read → privilege escalation chain

---

### 2. Environment Variables

Processes may expose sensitive values via environment variables.

```bash
env
printenv
```

#### Example:

```text
DB_PASS=rootpass123
AWS_SECRET_ACCESS_KEY=...
```

➡️ Exploitation examples:

* SSH login reuse
* Database connection abuse
* Cloud privilege escalation (metadata access)

---

### 3. Bash History (Very Common in CTFs and Real Systems)

```bash
cat ~/.bash_history
```

#### Example:

```text
mysql -u root -p'admin123'
ssh admin@localhost
sudo -S su
```

➡️ Exploitation:

* Reuse discovered passwords
* Identify internal services
* Find sudo passwords

---

### 4. SSH Keys (High-Value Target)

```bash
find /home -name "id_rsa" 2>/dev/null
```

#### Example:

```text
/home/user/.ssh/id_rsa
```

➡️ Exploitation:

* SSH into other internal users
* Lateral movement → privilege escalation chain
* Reuse key on other hosts

⚠️ If permissions are weak:

```bash
chmod 600 id_rsa
ssh -i id_rsa user@target
```

---

### 5. Database Credentials

Databases often run locally with weak or reused credentials.

#### Example:

```bash
ps aux | grep mysql
```

Config file:

```text
/var/lib/mysql/my.cnf
```

➡️ Exploitation paths:

* Dump sensitive tables
* Retrieve application users
* Extract password hashes → crack offline → reuse

---

### 6. Web Application Secrets

Web apps are one of the most common credential leakage sources.

#### Example files:

```text
config.php
settings.py
.env
database.yml
```

Search:

```bash
find /var/www -type f -name "*.php" 2>/dev/null
```

➡️ Exploitation:

* Admin panel login
* Database takeover
* Remote code execution via app features

---

### 7. Backup and Hidden Files

Backups often contain plaintext secrets.

#### Example:

```text
config.php.bak
database.sql
old.zip
```

Search:

```bash
find / -name "*.bak" 2>/dev/null
```

➡️ Exploitation:

* Extract credentials from backups
* Identify previous configurations
* Recover deleted secrets

---

### 8. Cron Jobs with Embedded Credentials

```bash
cat /etc/crontab
```

Example:

```text
root /opt/backup.sh
```

Inside script:

```bash
mysql -u root -padmin123 backup_db
```

➡️ Exploitation:

* Extract password from script
* Reuse for sudo / SSH / database login

---

## Real-World Privilege Escalation Chains

### Chain 1: Web App → Root

1. Find `/var/www/html/config.php`
2. Extract DB password
3. Login to MySQL
4. Read admin table
5. SSH as admin user
6. `sudo -l` → root access

---

### Chain 2: Bash History → Sudo

1. Read `~/.bash_history`
2. Find reused password
3. `sudo -l`
4. Escalate to root

---

### Chain 3: SSH Key Abuse

1. Find private key in `/home/user/.ssh/id_rsa`
2. SSH into another user
3. Access sudo privileges
4. Become root

---

## Key Enumeration Mindset

During real assessments, always ask:

* Where are applications storing secrets?
* What did users accidentally leave behind?
* Are credentials reused across services?
* Can I pivot using discovered secrets?
* Are there readable config files or backups?

---

## Detection & Defensive Perspective

From a DFIR standpoint:

* Monitor access to `/etc`, `/var/www`, `/home`
* Track reads of sensitive files
* Audit `.bash_history` access
* Detect unusual SSH key usage
* Alert on credential reuse patterns

---

## Summary

Credential-based privilege escalation is often:

> Faster than exploits, more reliable than kernel attacks, and easier to chain.

It remains one of the most important skill sets in Linux security assessments.
