# pfSense Firewall Setup

## Overview

pfSense is an open-source firewall/router that serves as the network gateway for this SOC homelab. It provides routing, DHCP, and firewall services for all lab VMs.

## Installation

### Prerequisites
- VirtualBox installed
- pfSense ISO (Netgate Installer v1.1.1 or later)
- Minimum 2GB RAM, 20GB disk space

### Step 1: Download pfSense

1. Go to https://www.pfsense.org/download/
2. Create a free Netgate account
3. Download: **AMD64 ISO IPMI/Virtual Machines**
4. Extract the `.iso.gz` file using 7-Zip

### Step 2: Create Virtual Machine

```
Name: pfSense
Type: BSD
Version: FreeBSD (64-bit)
Memory: 2048 MB
Processors: 1
Hard Disk: 20 GB (VDI)
```

### Step 3: Configure Network Adapters

**Adapter 1 (WAN):**
- Attached to: NAT
- Purpose: Internet connectivity

**Adapter 2 (LAN):**
- Attached to: Internal Network
- Name: LabNetwork
- Purpose: Lab network gateway

### Step 4: Install pfSense

1. Start VM and boot from ISO
2. Accept license agreement
3. Select **Install pfSense**
4. Choose **Auto (ZFS)** or **Auto (UFS)** partitioning
5. Wait for installation to complete
6. Remove ISO and reboot

### Step 5: Initial Configuration

When pfSense boots, configure interfaces:

```
Should VLANs be set up now? n

WAN interface: em0
LAN interface: em1

Confirm: y
```

### Step 6: Set LAN IP Address

From the pfSense menu, select option `2) Set interface(s) IP address`:

```
Select interface: 2 (LAN)
IPv4 address: 192.168.1.1
Subnet mask: 24
Upstream gateway: (leave blank)
IPv6: (skip)
Enable DHCP: y
Start address: 192.168.1.100
End address: 192.168.1.199
Revert to HTTP: n
```

## Final Configuration

After setup, pfSense shows:

```
WAN (wan) -> em0 -> v4/DHCP4: 10.0.2.15/24
LAN (lan) -> em1 -> v4: 192.168.1.1/24
```

## Web Interface Access

The pfSense web interface can be accessed from any VM on the LAN:
- URL: `https://192.168.1.1`
- Default username: `admin`
- Default password: `pfsense`

## Menu Options

```
0) Logout                      9) pfTop
1) Assign Interfaces          10) Filter Logs
2) Set interface(s) IP        11) Restart GUI
3) Reset admin password       12) PHP shell + pfSense tools
4) Reset to factory defaults  13) Update from console
5) Reboot system              14) Enable Secure Shell (sshd)
6) Halt system                15) Restore recent configuration
7) Ping host                  16) Restart PHP-FPM
8) Shell
```

## Troubleshooting

### No WAN IP Address
- Check Adapter 1 is set to NAT in VirtualBox
- Reboot pfSense

### LAN Devices Not Getting DHCP
- Verify DHCP is enabled on LAN interface
- Check other VMs are on "LabNetwork" internal network
- Restart DHCP service from pfSense menu

### Can't Access Web Interface
- Ping 192.168.1.1 from a LAN device
- Check firewall rules haven't been modified
