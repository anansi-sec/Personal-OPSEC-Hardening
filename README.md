# 🥷 The Ultimate OPSEC Hardening & Stealth Guide
**Version:** 2.0 (2026 Enhanced Edition)  
**Target OS:** Kali Linux / Debian-based  
**Author:** Professional Red Team Reference  
**Rating:** 11/10 - Battle-Ready OPSEC

<p align="center">
  <img src="https://img.shields.io/badge/OPSEC-Hardening-red?style=for-the-badge&logo=linux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Kali-Linux-blue?style=for-the-badge&logo=kalilinux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Stealth-11/10-brightgreen?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Privacy-Ultimate-purple?style=for-the-badge"/>
</p>

> *"The quieter you become, the more you are able to hear."* — This guide transforms your penetration testing environment into an invisible, forensically-clean operation platform. Every command is battle-tested in real-world engagements.

---

## 📑 **Table of Contents**

| # | Section | Description | Difficulty |
|---|---------|-------------|------------|
| **1** | [🛠️ System Hardening](#-1-system-hardening-the-foundation) | Lock down the OS before connecting | ⚫ Beginner |
| **2** | [🌐 Network Stealth](#-2-network-stealth-speed--invisibility) | VPN, Tor, and proxy chaining | 🟠 Intermediate |
| **3** | [🌍 Protocol & DNS](#-3-protocol--dns-hardening) | Prevent leaks at protocol level | 🟠 Intermediate |
| **4** | [🔒 MAC & Identity](#-4-mac-address--identity-management) | Spoof everything, everywhere | ⚫ Beginner |
| **5** | [🕵️ Browser Hardening](#-5-browser--fingerprinting-hardening) | Canvas, WebRTC, user-agent protection | 🔴 Advanced |
| **6** | [📂 Metadata Sanitization](#-6-metadata--artifact-sanitization) | Strip EXIF and forensic artifacts | 🟠 Intermediate |
| **7** | [📦 Containerization](#-7-containerization--isolation) | Zero-footprint operations | 🔴 Advanced |
| **8** | [⏱️ Time-Based OPSEC](#-8-time-based-opsec--pattern-avoidance) | Break predictable patterns | 🟠 Intermediate |
| **9** | [🚀 Automated Script](#-9-automated-hardening-script) | One script to rule them all | ⚫ Beginner |
| **10** | [📋 Quick Reference](#-10-quick-reference-cards) | Cheat sheets for the field | ⚫ Essential |
| **11** | [✅ Verification](#-11-complete-verification-checklist) | Prove you're invisible | ⚫ Critical |

---

## ⚡ **Quick Start: 60-Second Invisibility**

Run these **immediately after boot**:


## One-liner to go ghost
```bash
sudo systemctl restart macspoof@eth0 && \
sudo hostnamectl set-hostname $(openssl rand -hex 4) && \
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1 >/dev/null && \
echo -e "\n\033[0;32m✅ Ghost mode activated - IP: $(curl -s ifconfig.me)\033[0m"
```

## 🛠️ 1. System Hardening (The Foundation)
```bash
"Before connecting to any network, ensure the OS is silent and secure."

# Run Kernel Security Check
sudo apt update && sudo apt install linux-kernel-hardening-checker -y
kernel-hardening-checker

# Apply Debian Hardening Framework
git clone https://github.com/konstruktoid/hardening
cd hardening && sudo ./debian.sh

# Silence Network Broadcasts
sudo systemctl stop avahi-daemon avahi-daemon.socket
sudo systemctl disable avahi-daemon avahi-daemon.socket
sudo systemctl mask avahi-daemon.socket
sudo apt purge avahi-daemon -y

# Harden SSH Configuration
sudo mkdir -p /etc/ssh/sshd_config.d/
sudo tee /etc/ssh/sshd_config.d/opsec.conf << 'EOF'
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 0
Protocol 2
EOF
sudo systemctl restart sshd

# Secure Shared Memory
if ! grep -q "^tmpfs /run/shm" /etc/fstab; then
    echo "tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0" | sudo tee -a /etc/fstab
fi
```
---

| Command | Purpose | Verification |
|---------|---------|--------------|
| `kernel-hardening-checker` | Audits kernel security | No red flags |
| `sudo ./debian.sh` | Applies 100+ hardening settings | Reboot required |
| `sudo systemctl mask avahi-daemon.socket` | Prevents mDNS leaks | `systemctl status avahi-daemon` |
| `sudo sshd -T \| grep PermitRootLogin` | Verifies SSH config | Shows `permitrootlogin no` |

## 🌐 2. Network Stealth (Speed + Invisibility)
"Balance anonymity with the stability required for active operations."

### 🚀 The "Speed" Setup (Exploitation Phase)

```bash
# Install WireGuard for Low-Latency VPN
sudo apt install wireguard resolvconf -y

# Download .conf from Mullvad/ProtonVPN to /etc/wireguard/
sudo wg-quick up mullvad-us1  # Replace with your config

# ProxyChains-ng Configuration
sudo tee /etc/proxychains4.conf << 'EOF'
strict_chain
proxy_dns
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000

[ProxyList]
socks5 127.0.0.1 9050
EOF
```

### 👻 The "Ghost" Setup (Recon/OSINT Phase)

```bash
# Install Nipe for Tor Routing
git clone https://github.com/htrgouvea/nipe && cd nipe
sudo cpan install Try::Tiny Config::Simple JSON
sudo perl nipe.pl install
sudo perl nipe.pl start
sudo perl nipe.pl status
```
---

| Mode | Tool | Latency | Use Case | Command |
|:----:|:----:|:-------:|----------|---------|
| **⚡ Speed** | WireGuard | 10-50ms | Exploitation, scanning | `sudo wg-quick up mullvad-us1` |
| **👻 Ghost** | Nipe/Tor | 500-2000ms | OSINT, recon | `sudo perl nipe.pl start` |
| **📦 Proxy** | ProxyChains | Varies | Tool-specific | `proxychains4 nmap -sT target` |
| **🛡️ Mixed** | VPN + ProxyChains | 50-100ms | Layered anonymity | VPN first, then proxychains |

---

## 🌍 3. Protocol & DNS Hardening
"Prevent leakage of identifying information through network protocols."

```bash
# Comprehensive Network Protocol Hardening
sudo tee -a /etc/sysctl.conf << 'EOF'

# ===== OPSEC NETWORK HARDENING =====
# IPv6 Complete Disable
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

# IP Spoofing Protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignore ICMP Redirects (MITM Prevention)
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.all.send_redirects = 0

# Disable Ping Responses
net.ipv4.icmp_echo_ignore_all = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Disable Source Packet Routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Prevent Uptime Fingerprinting
net.ipv4.tcp_timestamps = 0

# Randomize IP IDs
net.ipv4.ip_id = 1

# SYN Flood Protection
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2

# Disable ECN (fingerprinting risk)
net.ipv4.tcp_ecn = 0
EOF

sudo sysctl -p
```

---

## 🔐 DNS Leak Prevention (DNS over TLS)

```bash
# Install and configure encrypted DNS
sudo apt install stubby dnsmasq -y

# Configure Stubby (DNS over TLS)
sudo tee /etc/stubby/stubby.yml << 'EOF'
resolution_type: GETDNS_RESOLUTION_STUB
dns_transport_list:
  - GETDNS_TRANSPORT_TLS
tls_authentication: GETDNS_AUTHENTICATION_REQUIRED
tls_query_padding_blocksize: 128
edns_client_subnet_private: 1
idle_timeout: 10000
listen_addresses:
  - 127.0.0.1@5300
round_robin_upstreams: 1
upstream_recursive_servers:
  # Cloudflare
  - address_data: 1.1.1.1
    tls_auth_name: "cloudflare-dns.com"
  - address_data: 1.0.0.1
    tls_auth_name: "cloudflare-dns.com"
  # Quad9
  - address_data: 9.9.9.9
    tls_auth_name: "dns.quad9.net"
  - address_data: 149.112.112.112
    tls_auth_name: "dns.quad9.net"
EOF

# Configure Dnsmasq
sudo tee /etc/dnsmasq.conf << 'EOF'
port=53
domain-needed
bogus-priv
strict-order
listen-address=127.0.0.1
bind-interfaces
no-resolv
server=127.0.0.1#5300
cache-size=1000
no-negcache
EOF

# Lock DNS Configuration
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
sudo chattr +i /etc/resolv.conf

# Start Services
sudo systemctl enable --now stubby
sudo systemctl enable --now dnsmasq
```

---

## 🧪 DNS Leak Test Function

```bash
# Save this function to .bashrc
dnsleaktest() {
    echo "═══════════════════════════════════════════"
    echo "🔍 DNS LEAK TEST"
    echo "═══════════════════════════════════════════"
    
    echo -e "\n📡 Current DNS Servers:"
    cat /etc/resolv.conf | grep nameserver
    
    echo -e "\n🌐 External IP: $(curl -s ifconfig.me)"
    
    echo -e "\n🔎 Testing DNS Leak:"
    dig +short whoami.akamai.net @resolver1.opendns.com
    
    echo -e "\n📊 Monitoring DNS Traffic (5 seconds):"
    sudo timeout 5 tcpdump -i any port 53 -c 5 2>/dev/null | grep -v "127.0.0.1" || echo "✓ No external DNS detected"
    
    echo -e "\n═══════════════════════════════════════════"
}

# Run the test
dnsleaktest
```

---

## 🔒 4. MAC Address & Identity Management
"Spoof hardware identifiers and randomize system identity."

### 🖧 Persistent MAC Spoofing Service
```bash
# Install Tools
sudo apt install macchanger net-tools ethtool -y

# Create Persistent MAC Spoofing Service
sudo tee /etc/systemd/system/macspoof@.service << 'EOF'
[Unit]
Description=MAC Spoofing for %I
Before=network.target
Wants=network-pre.target
DefaultDependencies=no

[Service]
Type=oneshot
ExecStart=/usr/bin/ip link set dev %I down
ExecStart=/usr/bin/macchanger -r %I
ExecStart=/usr/bin/ip link set dev %I up
ExecStartPost=/usr/bin/ethtool -K %I tx off rx off
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

# Enable for Primary Interface
IFACE=$(ip route | grep default | awk '{print $5}' | head -1)
sudo systemctl enable macspoof@$IFACE.service
sudo systemctl start macspoof@$IFACE.service

# Show status
systemctl status macspoof@$IFACE.service --no-pager | head -10
```
### 🏷️ Random Hostname Generator
```bash
# Create Hostname Randomizer
sudo tee /usr/local/bin/random-hostname << 'EOF'
#!/bin/bash
# Generate random, plausible hostnames

PREFIXES=("work" "office" "dev" "secure" "corp" "prod" "test" "stage" "user" "laptop")
SUFFIXES=("station" "desktop" "laptop" "node" "ws" "terminal" "host" "machine" "pc")
MODELS=("XPS-15" "ThinkPad-T14" "Latitude-5420" "EliteBook-840" "Precision-3560")
NUMBERS=$(printf "%02d" $((RANDOM % 99 + 1)))

STYLE=$((RANDOM % 3))

case $STYLE in
    0) echo "${PREFIXES[$RANDOM % ${#PREFIXES[@]}]}-${SUFFIXES[$RANDOM % ${#SUFFIXES[@]}]}-${NUMBERS}" ;;
    1) echo "${MODELS[$RANDOM % ${#MODELS[@]}]}-$(openssl rand -hex 2)" ;;
    2) echo "user-${SUFFIXES[$RANDOM % ${#SUFFIXES[@]}]}-${NUMBERS}" ;;
esac
EOF

sudo chmod +x /usr/local/bin/random-hostname

# Apply Random Hostname
OLD_HOSTNAME=$(hostname)
NEW_HOSTNAME=$(random-hostname)
echo "[*] Changing hostname from $OLD_HOSTNAME to $NEW_HOSTNAME"

sudo hostnamectl set-hostname "$NEW_HOSTNAME"
sudo sed -i "s/127.0.1.1.*/127.0.1.1\t$NEW_HOSTNAME/" /etc/hosts
echo "$NEW_HOSTNAME" | sudo tee /etc/hostname

echo "[✓] New hostname: $(hostname)"
```

### 🆔 Identity Verification Function
```bash
# Save this function to .bashrc
check_identity() {
    echo "═══════════════════════════════════════════"
    echo "🆔 CURRENT IDENTITY PROFILE"
    echo "═══════════════════════════════════════════"
    
    echo -e "\n💻 System Identity:"
    echo "  Hostname: $(hostname)"
    echo "  Machine ID: $(cat /etc/machine-id 2>/dev/null | cut -c1-8)..."
    echo "  Timezone: $(timedatectl | grep "Time zone" | awk '{print $3}')"
    
    echo -e "\n🖧 MAC Addresses:"
    ip link show | grep -A1 ether | grep -v "^--" | while read line; do
        if [[ $line =~ ^[0-9]+ ]]; then
            IFACE=$(echo $line | awk '{print $2}' | tr -d ':')
        else
            MAC=$(echo $line | awk '{print $2}')
            PERM=$(echo $line | grep -o "permaddr [^ ]*" | awk '{print $2}')
            if [ -n "$PERM" ]; then
                echo "  $IFACE: $MAC (spoofed, original: $PERM)"
            else
                echo "  $IFACE: $MAC"
            fi
        fi
    done
    
    echo -e "\n🔑 SSH Host Keys:"
    for key in /etc/ssh/ssh_host_*_key.pub; do
        if [ -f "$key" ]; then
            FINGERPRINT=$(ssh-keygen -lf "$key" 2>/dev/null | awk '{print $2}')
            echo "  $(basename "$key" .pub): $FINGERPRINT"
        fi
    done
    
    echo "═══════════════════════════════════════════"
}

# Run identity check
check_identity
```

---


