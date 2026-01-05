# Detection Rules

## Overview

This document describes the custom detection rules created for the SOC homelab. These rules extend Wazuh's built-in detection capabilities.

## Rule File Location

Custom rules are stored in:
```
/var/ossec/etc/rules/local_rules.xml
```

## Custom Rules

### Rule 100002: Failed Root SSH Login

**Purpose:** Detect failed SSH login attempts specifically targeting the root account.

**Severity:** Level 10 (High)

**Rationale:** Root login attempts are often indicators of targeted attacks or brute force attempts against privileged accounts.

```xml
<rule id="100002" level="10">
  <if_sid>5716</if_sid>
  <user>root</user>
  <description>Failed SSH login attempt for root user</description>
  <group>authentication_failed,root_attempt,</group>
</rule>
```

**Triggers when:** An SSH login attempt for the root user fails.

**MITRE ATT&CK:** T1110 (Brute Force), T1078 (Valid Accounts)

---

### Rule 100003: SSH Brute Force Detection

**Purpose:** Detect potential brute force attacks by identifying multiple failed login attempts in a short time period.

**Severity:** Level 12 (Critical)

**Rationale:** Multiple failed logins in rapid succession typically indicate an automated attack.

```xml
<rule id="100003" level="12" frequency="5" timeframe="120">
  <if_matched_sid>5716</if_matched_sid>
  <description>Possible SSH brute force attack (5 failures in 2 min)</description>
  <group>authentication_failed,brute_force,</group>
</rule>
```

**Triggers when:** 5 or more failed SSH logins occur within 2 minutes (120 seconds).

**MITRE ATT&CK:** T1110.001 (Password Guessing), T1110.003 (Password Spraying)

---

### Rule 100004: New User Account Created

**Purpose:** Alert when new user accounts are created on monitored systems.

**Severity:** Level 8 (Medium)

**Rationale:** Unauthorized user creation can indicate compromise or insider threat activity.

```xml
<rule id="100004" level="8">
  <if_sid>5901</if_sid>
  <description>New user account created on system</description>
  <group>account_created,</group>
</rule>
```

**Triggers when:** A new user account is created using useradd or similar commands.

**MITRE ATT&CK:** T1136.001 (Create Account: Local Account)

---

## File Integrity Monitoring (FIM)

### Configuration

FIM is configured in `/var/ossec/etc/ossec.conf` under the `<syscheck>` section:

```xml
<syscheck>
  <disabled>no</disabled>
  <frequency>43200</frequency>
  <scan_on_start>yes</scan_on_start>
  <alert_new_files>yes</alert_new_files>
  
  <!-- Standard directories -->
  <directories>/etc,/usr/bin,/usr/sbin</directories>
  <directories>/bin,/sbin,/boot</directories>
  
  <!-- Critical files with realtime monitoring -->
  <directories realtime="yes">/etc/passwd,/etc/shadow,/home</directories>
</syscheck>
```

### Monitored Paths

| Path | Monitoring Type | Purpose |
|------|-----------------|---------|
| /etc | Scheduled (12h) | System configuration files |
| /usr/bin, /usr/sbin | Scheduled (12h) | System binaries |
| /bin, /sbin | Scheduled (12h) | Essential binaries |
| /boot | Scheduled (12h) | Boot files |
| /etc/passwd | Realtime | User account database |
| /etc/shadow | Realtime | Password hashes |
| /home | Realtime | User home directories |

---

## Rule Severity Levels

| Level | Severity | Description |
|-------|----------|-------------|
| 0-3 | Low | Informational events |
| 4-7 | Medium | Notable events requiring attention |
| 8-11 | High | Important security events |
| 12-15 | Critical | Severe security incidents |

---

## Testing Rules

### Using wazuh-logtest

```bash
sudo /var/ossec/bin/wazuh-logtest
```

### Test Samples

**Test failed root login (Rule 100002):**
```
Dec 10 01:02:02 host sshd[1234]: Failed password for root from 1.1.1.1 port 1066 ssh2
```

**Test new user creation (Rule 100004):**
```
sudo useradd testuser
```

**Test file integrity monitoring:**
```
sudo touch /etc/passwd
```

---

## Adding New Rules

1. Edit the local rules file:
   ```bash
   sudo nano /var/ossec/etc/rules/local_rules.xml
   ```

2. Add new rules between `<group>` and `</group>` tags

3. Use rule IDs starting from 100000

4. Test with wazuh-logtest before restarting

5. Restart Wazuh manager:
   ```bash
   sudo systemctl restart wazuh-manager
   ```

---

## Built-in Rules Reference

Wazuh includes thousands of pre-built rules. Common parent rules:

| Rule ID | Description |
|---------|-------------|
| 5716 | SSHD authentication failed |
| 5901 | New user added |
| 5902 | User deleted |
| 5903 | User password changed |
| 550 | File integrity checksum changed |
| 554 | File added to system |
