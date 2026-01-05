# Lessons Learned

## Overview

This document captures key lessons learned during the build and configuration of this SOC homelab.

---

## Installation Challenges

### pfSense Installation Issues

**Problem:** First pfSense installation failed with "No /boot/kernel/kernel" error.

**Cause:** ZFS filesystem installation didn't complete properly in VirtualBox.

**Solution:** 
- Reinstalled using the newer Netgate installer
- Let the installer complete fully before rebooting
- Removed ISO before reboot to prevent boot loop

**Lesson:** Always verify installation completes successfully before rebooting. Remove installation media promptly.

---

### Wazuh Agent Conflict

**Problem:** Couldn't install Wazuh agent on the Wazuh server.

**Error:** `wazuh-agent conflicts with wazuh-manager`

**Cause:** The Wazuh manager and agent cannot run on the same system.

**Solution:** The Wazuh manager already monitors itself - no separate agent needed.

**Lesson:** Understand component relationships before installation. The manager includes self-monitoring capabilities.

---

### Metasploitable SSL Issues

**Problem:** Couldn't download Wazuh agent on Metasploitable.

**Error:** `OpenSSL error: SSL routines: SSL23_GET_SERVER_HELLO: sslv3 alert handshake failure`

**Cause:** Metasploitable 2 uses outdated SSL libraries that can't negotiate with modern HTTPS servers.

**Solution:** 
- Metasploitable is too old for modern agent installation
- Use it as an agentless target for attack simulation instead
- Focus agent deployment on modern systems

**Lesson:** Legacy systems may have compatibility issues with modern security tools. Plan accordingly.

---

### Network Connectivity Issues

**Problem:** Metasploitable couldn't reach the internet initially.

**Cause:** Missing default gateway and DNS configuration.

**Solution:**
```bash
sudo route add default gw 192.168.1.1
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

**Lesson:** Internal network VMs need explicit gateway and DNS configuration if not provided by DHCP options.

---

## Configuration Lessons

### Rule Syntax Errors

**Problem:** Wazuh manager failed to start after adding custom rules.

**Error:** `Invalid option 'if_matched_side' for rule '100003'`

**Cause:** Typo in rule XML - used `if_matched_side` instead of `if_matched_sid`.

**Solution:** 
1. Check logs: `sudo cat /var/ossec/logs/ossec.log | tail -30`
2. Fix typo in `/var/ossec/etc/rules/local_rules.xml`
3. Restart manager

**Lesson:** 
- Always check logs when services fail to start
- Use `wazuh-logtest` to validate rules before restarting
- XML syntax must be exact

---

### NAT Adapter Persistence

**Problem:** Wazuh's NAT adapter (enp0s8) doesn't automatically get an IP after reboot.

**Cause:** The second adapter isn't configured in netplan by default.

**Solution:** Run after each boot:
```bash
sudo ip link set enp0s8 up && sudo dhcpcd enp0s8
```

**Future improvement:** Add persistent network configuration to netplan.

**Lesson:** Test configurations after reboot to catch persistence issues.

---

## Best Practices Identified

### 1. Start VMs in Order
Always start pfSense first - it provides DHCP and routing for all other VMs.

### 2. Document Credentials
Keep track of all passwords:
- pfSense admin password
- Wazuh dashboard admin password
- VM user credentials

### 3. Save VM States
Use VirtualBox "Save State" feature to preserve work:
- Faster than full shutdown/startup
- Preserves running processes
- Useful for long installations

### 4. Test Before Production
Always test rules with `wazuh-logtest` before restarting the manager.

### 5. Check Logs First
When something fails:
1. Check service status: `systemctl status <service>`
2. Check logs: `journalctl -u <service>` or application-specific logs
3. Search for specific errors

---

## Future Improvements

### Short Term
- [ ] Add Kali Linux for penetration testing
- [ ] Configure email alerts for critical events
- [ ] Add more custom detection rules

### Medium Term
- [ ] Set up automated VM snapshots
- [ ] Create incident response playbooks
- [ ] Add Windows VM with Sysmon

### Long Term
- [ ] Integrate threat intelligence feeds
- [ ] Add network traffic analysis (Zeek/Suricata)
- [ ] Implement SOAR automation

---

## Resources Used

### Documentation
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [VirtualBox Manual](https://www.virtualbox.org/manual/)

### Troubleshooting
- Wazuh logs: `/var/ossec/logs/ossec.log`
- System logs: `journalctl -xe`
- Network debugging: `ip a`, `ping`, `traceroute`

---

## Time Investment

| Task | Estimated Time |
|------|----------------|
| VirtualBox setup | 30 minutes |
| pfSense installation | 1 hour |
| Wazuh installation | 1-2 hours |
| Network configuration | 1 hour |
| Custom rules creation | 1 hour |
| Testing and validation | 1 hour |
| Documentation | 1-2 hours |
| **Total** | **6-9 hours** |

---

## Conclusion

Building this SOC homelab provided hands-on experience with:
- Network architecture and segmentation
- Firewall configuration
- SIEM deployment and configuration
- Detection engineering
- Troubleshooting complex systems

The skills gained are directly applicable to real-world security operations roles.
