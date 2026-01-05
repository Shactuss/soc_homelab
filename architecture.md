# Network Architecture

## Overview

This SOC homelab uses a segmented network architecture with pfSense acting as the central firewall and router. All VMs communicate through an internal virtual network called "LabNetwork."

## Network Diagram

```
                         INTERNET
                            │
                            │ NAT (VirtualBox)
                            │
                    ┌───────▼───────┐
                    │   pfSense     │
                    │   Firewall    │
                    ├───────────────┤
                    │ WAN: em0      │◄── NAT (10.0.2.15)
                    │ LAN: em1      │◄── Internal (192.168.1.1)
                    └───────┬───────┘
                            │
                            │ LabNetwork (Internal Network)
                            │ Subnet: 192.168.1.0/24
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
│    Wazuh      │   │ Metasploitable│   │  (Future)     │
│    SIEM       │   │    Target     │   │ Kali Linux    │
├───────────────┤   ├───────────────┤   ├───────────────┤
│192.168.1.100  │   │192.168.1.101  │   │192.168.1.x    │
│ enp0s3: LAN   │   │  eth0: LAN    │   │               │
│ enp0s8: NAT   │   │               │   │               │
└───────────────┘   └───────────────┘   └───────────────┘
```

## IP Address Allocation

| Device | Interface | IP Address | Purpose |
|--------|-----------|------------|---------|
| pfSense | em0 (WAN) | 10.0.2.15 (DHCP) | Internet access via NAT |
| pfSense | em1 (LAN) | 192.168.1.1 | Gateway for lab network |
| Wazuh | enp0s3 | 192.168.1.100 | Lab network communication |
| Wazuh | enp0s8 | 10.0.3.x (NAT) | Dashboard access from host |
| Metasploitable | eth0 | 192.168.1.101 | Lab network (target) |

## DHCP Configuration

pfSense provides DHCP services on the LAN interface:
- **Range:** 192.168.1.100 - 192.168.1.199
- **Gateway:** 192.168.1.1
- **DNS:** Provided by pfSense

## VirtualBox Network Configuration

### pfSense VM
- **Adapter 1:** NAT (WAN interface)
- **Adapter 2:** Internal Network "LabNetwork" (LAN interface)

### Wazuh VM
- **Adapter 1:** Internal Network "LabNetwork"
- **Adapter 2:** NAT (for dashboard access via port forwarding)

### Metasploitable VM
- **Adapter 1:** Internal Network "LabNetwork"

## Port Forwarding (Wazuh Dashboard Access)

To access the Wazuh dashboard from the host machine:

| Name | Protocol | Host IP | Host Port | Guest Port |
|------|----------|---------|-----------|------------|
| Wazuh-Dashboard | TCP | 127.0.0.1 | 9443 | 443 |

Access URL: `https://127.0.0.1:9443`

## Firewall Rules

pfSense default rules:
- **WAN:** Block all inbound (default deny)
- **LAN:** Allow all outbound to WAN
- **LAN:** Allow inter-LAN communication

## Security Considerations

1. **Isolation:** The lab network is isolated from the host network
2. **NAT Protection:** WAN interface uses NAT, no direct internet exposure
3. **Controlled Access:** Dashboard access only through port forwarding
4. **Vulnerable Systems:** Metasploitable is intentionally vulnerable - keep isolated
