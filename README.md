# 🥷 OPSEC Hardening & Stealth Guide
> **Version:** 1.0 (2026 Reference)  
> **Target OS:** Kali Linux / Debian-based

This repository serves as a personal reference for hardening a penetration testing environment to remain invisible and maintain high-speed connectivity during engagements.

## 🛠️ 1. System Hardening (The Foundation)

Before connecting to any network, ensure the OS is silent and secure.

- **Run Kernel Check:** Use [kernel-hardening-checker](https://github.com/a13xp0p0v/kernel-hardening-checker) to verify KSPP alignment.
- **Apply Debian Hardening:** Utilize [Debian-Hardening](https://github.com/ovh/debian-hardening) scripts for AppArmor profiles.
- **Silence Broadcasts:** Disable `Avahi` and `mDNS` to prevent local discovery.

## 🌐 2. Network Stealth (Speed + Invisibility)

Balance anonymity with the stability required for active exploitation.

### The "Speed" Setup (Exploitation Phase)
- **Protocol:** Use **WireGuard** (via Mullvad or ProtonVPN) for low latency.
- **Tool:** [ProxyChains-ng](https://github.com/rofl0r/proxychains-ng).
- **Config:** Set to `dynamic_chain` in `/etc/proxychains4.conf` to avoid connection drops.

### The "Ghost" Setup (Recon/OSINT Phase)
- **Tool:** [Nipe](https://github.com/htrgouvea/nipe).
- **Function:** Routes all traffic through Tor.
- **Note:** Switch back to VPN before launching heavy exploits.

## 📂 3. Operational Security (OPSEC) Repos

- **Infrastructure:** [Awesome-Red-Teaming](https://github.com/yeyintminthuhtut/Awesome-Red-Teaming) - Focus on *Cloud Redirectors*.
- **Evasion:** [CheckPlease](https://github.com/p3nt3st/CheckPlease) - Use for sandbox-aware payloads.

## 🚀 4. "Invisible Operator" Quick-Start

Run these commands immediately upon booting your Kali instance:

### A. Spoof MAC Address
```bash
# Replace eth0 with your active interface (e.g., wlan0)
sudo macchanger -r eth0
```

### B. Change Hostname
```bash
sudo hostnamectl set-hostname Workstation-01
```

### C. Disable IPv6
```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
```
---

## 🖥️ Automated Hardening Script
Take the guesswork out of your OPSEC routine with the included `harden.sh` script.

```bash
# Make the script executable
chmod +x harden.sh

# Run the complete hardening process
sudo ./harden.sh
```
---

## 💾 Persistence Guide
The `sysctl -w` commands reset after reboot. Make your stealth configurations permanent by adding the following to `/etc/sysctl.conf`:

```bash
# Add to /etc/sysctl.conf for permanent IPv6 disabling
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf

# Apply changes
sudo sysctl -p
```
---

## 🧹 Artifact Cleaning

A key part of being an "Invisible Operator" is knowing how to clear your tracks before logging off:

```bash
# Clear bash history
history -c && history -w

# Securely delete temporary files
shred -zvu /tmp/* 2>/dev/null
shred -zvu ~/.bash_history

# Wipe system logs (use with caution)
sudo journalctl --rotate
sudo journalctl --vacuum-time=1s
```
---

## ✅ Verification Section

Confirm your "Invisible" status before touching the target network:

```bash
# Verify MAC address change
ip a | grep -A 1 ether

# Verify hostname change
hostname

# Verify IPv6 is disabled
sysctl net.ipv6.conf.all.disable_ipv6
ip a | grep inet6

# Verify network interfaces
ip a

# Check active connections
ss -tuln
```
---

## 📋 Quick Reference Card

| Action | Command | Verification |
|--------|---------|--------------|
| **Spoof MAC** | `sudo macchanger -r eth0` | `ip a \| grep -A 1 ether` |
| **Change Hostname** | `sudo hostnamectl set-hostname NEW` | `hostname` |
| **Disable IPv6** | `sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1` | `sysctl net.ipv6.conf.all.disable_ipv6` |
| **Clear History** | `history -c && history -w` | `history` |
| **Run Full Hardening** | `sudo ./harden.sh` | Run all verification commands above |

> **💡 Pro Tip:** Run verification commands **before** connecting to the target network to ensure you're truly invisible.

> **⚠️ Legal Disclaimer**
> 
> This guide is intended for **educational purposes** and **authorized security testing only**. Always ensure you have proper written permission before testing or hardening any system you do not own. The authors are not responsible for misuse, illegal activities, or any damages resulting from the use of this information.
> 
> *By using this guide, you acknowledge that you are solely responsible for complying with all applicable local, state, and federal laws.*
