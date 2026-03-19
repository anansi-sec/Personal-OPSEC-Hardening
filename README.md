# 🥷 OPSEC Hardening & Stealth Guide
> **Version:** 1.0 (2026 Reference)  
> **Target OS:** Kali Linux / Debian-based

This repository serves as a personal reference for hardening a penetration testing environment to remain invisible and maintain high-speed connectivity during engagements.

---

## 🛠️ 1. System Hardening (The Foundation)
Before connecting to any network, ensure the OS is silent and secure.

- [ ] **Run Kernel Check:** Use [kernel-hardening-checker](https://github.com/a13xp0p0v/kernel-hardening-checker) to verify KSPP alignment.
- [ ] **Apply Debian Hardening:** Utilize [Debian-Hardening](https://github.com/ovh/debian-hardening) scripts for AppArmor profiles.
- [ ] **Silence Broadcasts:** Disable `Avahi` and `mDNS` to prevent local discovery.

## 🌐 2. Network Stealth (Speed + Invisibility)
Balance anonymity with the stability required for active exploitation.

### The "Speed" Setup (Exploitation Phase)
*   **Protocol:** Use **WireGuard** (via Mullvad or ProtonVPN) for low latency.
*   **Tool:** [ProxyChains-ng](https://github.com/rofl0r/proxychains-ng).
*   **Config:** Set to `dynamic_chain` in `/etc/proxychains4.conf` to avoid connection drops.

### The "Ghost" Setup (Recon/OSINT Phase)
*   **Tool:** [Nipe](https://github.com/htrgouvea/nipe).
*   **Function:** Routes all traffic through Tor. 
*   *Note: Switch back to VPN before launching heavy exploits.*

## 📂 3. Operational Security (OPSEC) Repos
- **Infrastructure:** [Awesome-Red-Teaming](https://github.com/yeyintminthuhtut/Awesome-Red-Teaming) - Focus on *Cloud Redirectors*.
- **Evasion:** [CheckPlease](https://github.com/p3nt3st/CheckPlease) - Use for sandbox-aware payloads.

---

## 🚀 4. "Invisible Operator" Quick-Start
Run these commands immediately upon booting your Kali instance:

### A. Spoof MAC Address
```bash
# Replace eth0 with your active interface (e.g., wlan0)
sudo macchanger -r eth0

sudo hostnamectl set-hostname Workstation-01

sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
