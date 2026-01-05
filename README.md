# SOC Homelab Project

A fully functional Security Operations Center (SOC) homelab built for learning threat detection, log analysis, and incident response.

## 🏗️ Architecture

```
                    ┌─────────────────┐
                    │    Internet     │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │    pfSense      │
                    │   (Firewall)    │
                    │  192.168.1.1    │
                    └────────┬────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
   ┌────────▼────────┐ ┌─────▼──────┐ ┌──────▼───────┐
   │     Wazuh       │ │Metasploitable│ │   (Future)   │
   │     (SIEM)      │ │  (Target)    │ │ Kali Linux   │
   │ 192.168.1.100   │ │192.168.1.101 │ │  (Attacker)  │
   └─────────────────┘ └──────────────┘ └──────────────┘
```

## ✅ Project Progress

- [x] Install VirtualBox
- [x] Set up pfSense firewall
- [x] Install Wazuh SIEM
- [x] Add vulnerable target machines (Metasploitable)
- [x] Configure log forwarding
- [x] Create detection rules
- [ ] Add Kali Linux for penetration testing
- [ ] Create incident response playbooks

## 🛠️ Components

| Component | Role | IP Address | Platform |
|-----------|------|------------|----------|
| pfSense | Firewall/Router | 192.168.1.1 | FreeBSD |
| Wazuh | SIEM & Log Analysis | 192.168.1.100 | Ubuntu 24.04 |
| Metasploitable | Vulnerable Target | 192.168.1.101 | Ubuntu (Legacy) |

## 📁 Documentation

- [Network Architecture](docs/architecture.md)
- [pfSense Setup](docs/pfsense-setup.md)
- [Wazuh SIEM Setup](docs/wazuh-setup.md)
- [Detection Rules](docs/detection-rules.md)
- [Lessons Learned](docs/lessons-learned.md)

## 🔍 Custom Detection Rules

| Rule ID | Description | Severity Level |
|---------|-------------|----------------|
| 100002 | Failed SSH login attempt for root user | 10 (High) |
| 100003 | Possible SSH brute force attack (5 failures in 2 min) | 12 (Critical) |
| 100004 | New user account created on system | 8 (Medium) |

## 🚀 Quick Start

1. **Start VMs in order:**
   - pfSense first (provides network/DHCP)
   - Wazuh second (SIEM)
   - Metasploitable (target)

2. **Access Wazuh Dashboard:**
   ```
   https://127.0.0.1:9443
   Username: admin
   ```

3. **Enable NAT adapter on Wazuh (if needed):**
   ```bash
   sudo ip link set enp0s8 up && sudo dhcpcd enp0s8
   ```

## 📊 Skills Demonstrated

- Network segmentation and firewall configuration
- SIEM deployment and configuration
- Custom detection rule creation
- Log analysis and threat hunting
- Security monitoring and alerting
- File integrity monitoring
- Virtual lab environment setup

## 🔧 Technologies Used

- **Virtualization:** VirtualBox
- **Firewall:** pfSense 2.8.1
- **SIEM:** Wazuh 4.9.2
- **Target System:** Metasploitable 2
- **Operating Systems:** Ubuntu Server 24.04, FreeBSD

## 📝 License

This project is for educational purposes only.

## 🙏 Acknowledgments

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [Metasploitable Project](https://sourceforge.net/projects/metasploitable/)
