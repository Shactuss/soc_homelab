# Wazuh SIEM Setup

## Overview

Wazuh is an open-source security platform that provides unified XDR and SIEM protection. In this homelab, it serves as the central security monitoring system.

## Components

- **Wazuh Indexer:** Stores and indexes security data
- **Wazuh Manager:** Analyzes data, triggers alerts, manages agents
- **Wazuh Dashboard:** Web interface for visualization

## Installation

### Prerequisites
- Ubuntu Server 24.04 LTS
- Minimum 4GB RAM (8GB recommended)
- 50GB disk space
- Network connectivity to pfSense LAN

### Step 1: Create Virtual Machine

```
Name: Wazuh
Type: Linux
Version: Ubuntu (64-bit)
Memory: 4096 MB
Processors: 2
Hard Disk: 50 GB (VDI)
```

### Step 2: Configure Network

**Adapter 1:**
- Attached to: Internal Network
- Name: LabNetwork

**Adapter 2:**
- Attached to: NAT
- Port Forwarding: Host 9443 → Guest 443

### Step 3: Install Ubuntu Server

1. Boot from Ubuntu Server ISO
2. Follow installation wizard
3. Set hostname: `wazuh`
4. Create user account (e.g., `soc`)
5. Enable OpenSSH server
6. Complete installation and reboot

### Step 4: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 5: Install Wazuh

Run the all-in-one installer:

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

**Important:** Save the admin password displayed at the end!

Installation takes approximately 10-15 minutes.

## Post-Installation

### Enable NAT Adapter (for dashboard access)

After each reboot, run:

```bash
sudo ip link set enp0s8 up && sudo dhcpcd enp0s8
```

### Access Dashboard

1. Open browser on host machine
2. Navigate to: `https://127.0.0.1:9443`
3. Accept the self-signed certificate warning
4. Login with:
   - Username: `admin`
   - Password: (saved during installation)

## Service Management

### Check Service Status

```bash
# Wazuh Manager
sudo systemctl status wazuh-manager

# Wazuh Indexer
sudo systemctl status wazuh-indexer

# Wazuh Dashboard
sudo systemctl status wazuh-dashboard
```

### Restart Services

```bash
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard
```

## Important File Locations

| File | Purpose |
|------|---------|
| `/var/ossec/etc/ossec.conf` | Main configuration |
| `/var/ossec/etc/rules/local_rules.xml` | Custom detection rules |
| `/var/ossec/logs/ossec.log` | Wazuh logs |
| `/var/ossec/logs/alerts/alerts.log` | Alert logs |

## Testing Rules

Use the log testing tool:

```bash
sudo /var/ossec/bin/wazuh-logtest
```

Paste sample log entries to see which rules match.

## Troubleshooting

### Dashboard Not Accessible
1. Check if Wazuh services are running
2. Verify NAT adapter is up: `ip a`
3. Check port forwarding in VirtualBox

### Rules Not Loading
1. Check syntax: `sudo /var/ossec/bin/wazuh-logtest`
2. View errors: `sudo cat /var/ossec/logs/ossec.log | tail -30`
3. Restart manager after rule changes

### High Memory Usage
Wazuh indexer is memory-intensive. Ensure VM has at least 4GB RAM.
