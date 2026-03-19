🥷 OPSEC Hardening & Stealth Guide
Version: 2.0 (2026 Enhanced Edition)
Target OS: Kali Linux / Debian-based
Author: Professional Red Team Reference

This comprehensive guide transforms your penetration testing environment into an invisible, forensically-clean operation platform. Every command is battle-tested for real-world engagements.

📚 Table of Contents
System Hardening

Network Stealth

Protocol & DNS Hardening

MAC & Identity Management

Browser Fingerprinting Hardening

Metadata & Artifact Sanitization

Containerization & Isolation

Time-Based OPSEC

Automated Hardening Script

Quick Reference Cards

Complete Verification

🛠️ 1. System Hardening (The Foundation)
Before connecting to any network, ensure the OS is silent and secure.

bash
# Run Kernel Security Check
sudo apt update && sudo apt install linux-kernel-hardening-checker -y
kernel-hardening-checker

# Apply Debian Hardening Framework
git clone https://github.com/konstruktoid/hardening
cd hardening && sudo ./debian.sh  # Use debian.sh for Kali

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
🌐 2. Network Stealth (Speed + Invisibility)
Balance anonymity with the stability required for active operations.

The "Speed" Setup (Exploitation Phase)
bash
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
The "Ghost" Setup (Recon/OSINT Phase)
bash
# Install Nipe for Tor Routing
git clone https://github.com/htrgouvea/nipe && cd nipe
sudo cpan install Try::Tiny Config::Simple JSON
sudo perl nipe.pl install
sudo perl nipe.pl start
sudo perl nipe.pl status
Quick Network Toggle Commands
Mode	Command	Use Case
VPN	sudo wg-quick up mullvad-us1	Exploitation, scanning
Tor	sudo perl nipe.pl start	Recon, OSINT, anonymity
Proxy	proxychains4 nmap -sT target	Tool-specific routing
🌍 3. Protocol & DNS Hardening
Prevent leakage of identifying information through network protocols.

bash
# Comprehensive Network Hardening
sudo tee -a /etc/sysctl.conf << 'EOF'
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
net.ipv4.conf.all.send_redirects = 0

# Disable Ping Responses
net.ipv4.icmp_echo_ignore_all = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Prevent Uptime Fingerprinting
net.ipv4.tcp_timestamps = 0

# Randomize IP IDs
net.ipv4.ip_id = 1

# SYN Flood Protection
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_syn_retries = 2
EOF

sudo sysctl -p

# DNS Leak Prevention (DNS over TLS)
sudo apt install stubby dnsmasq -y

# Configure Stubby (DoT)
sudo tee /etc/stubby/stubby.yml << 'EOF'
resolution_type: GETDNS_RESOLUTION_STUB
dns_transport_list:
  - GETDNS_TRANSPORT_TLS
tls_authentication: GETDNS_AUTHENTICATION_REQUIRED
listen_addresses:
  - 127.0.0.1@5300
upstream_recursive_servers:
  - address_data: 1.1.1.1
    tls_auth_name: "cloudflare-dns.com"
  - address_data: 9.9.9.9
    tls_auth_name: "dns.quad9.net"
EOF

# Configure Dnsmasq
sudo tee /etc/dnsmasq.conf << 'EOF'
port=53
domain-needed
bogus-priv
strict-order
listen-address=127.0.0.1
server=127.0.0.1#5300
cache-size=1000
EOF

# Lock DNS Configuration
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
sudo chattr +i /etc/resolv.conf

# Start Services
sudo systemctl enable --now stubby dnsmasq
DNS Leak Test Function
bash
dnsleaktest() {
    echo "═══════════════════════════════════════════"
    echo "🔍 DNS LEAK TEST"
    echo "═══════════════════════════════════════════"
    echo "[*] Current DNS: $(cat /etc/resolv.conf | grep nameserver)"
    echo "[*] External IP: $(curl -s ifconfig.me)"
    echo "[*] Testing DNS Leak:"
    dig +short whoami.akamai.net @resolver1.opendns.com
    echo "[*] Monitoring DNS traffic (5 sec):"
    sudo timeout 5 tcpdump -i any port 53 -c 5 2>/dev/null
}
🔒 4. MAC Address & Identity Management
Spoof hardware identifiers and randomize system identity.

MAC Spoofing Service
bash
# Install Tools
sudo apt install macchanger net-tools ethtool -y

# Create Persistent MAC Spoofing Service
sudo tee /etc/systemd/system/macspoof@.service << 'EOF'
[Unit]
Description=MAC Spoofing for %I
Before=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/ip link set dev %I down
ExecStart=/usr/bin/macchanger -r %I
ExecStart=/usr/bin/ip link set dev %I up
ExecStartPost=/usr/bin/ethtool -K %I tx off rx off

[Install]
WantedBy=multi-user.target
EOF

# Enable for Primary Interface
IFACE=$(ip route | grep default | awk '{print $5}' | head -1)
sudo systemctl enable macspoof@$IFACE.service
sudo systemctl start macspoof@$IFACE.service
Random Hostname Generator
bash
# Create Hostname Randomizer
sudo tee /usr/local/bin/random-hostname << 'EOF'
#!/bin/bash
PREFIXES=("work" "office" "dev" "secure" "corp")
SUFFIXES=("station" "desktop" "laptop" "node" "ws")
echo "${PREFIXES[$RANDOM % ${#PREFIXES[@]}]}-${SUFFIXES[$RANDOM % ${#SUFFIXES[@]}]}-$((RANDOM % 99 + 1))"
EOF
sudo chmod +x /usr/local/bin/random-hostname

# Apply Random Hostname
sudo hostnamectl set-hostname $(random-hostname)
sudo sed -i "s/127.0.1.1.*/127.0.1.1\t$(hostname)/" /etc/hosts
Identity Verification
bash
check_identity() {
    echo "═══════════════════════════════════════════"
    echo "🆔 CURRENT IDENTITY PROFILE"
    echo "═══════════════════════════════════════════"
    echo "Hostname: $(hostname)"
    echo "MAC: $(ip link show | grep ether | head -1 | awk '{print $2}')"
    echo "Machine ID: $(cat /etc/machine-id 2>/dev/null | cut -c1-8)..."
    echo "Timezone: $(timedatectl | grep "Time zone" | awk '{print $3}')"
}
🕵️ 5. Browser & Fingerprinting Hardening
Eliminate browser-based tracking and canvas fingerprinting.

Hardened Firefox Profile
bash
# Create OSINT Profile
firefox -CreateProfile "OSINT-Profile" 2>/dev/null || true

# Find Profile Path
PROFILE_PATH=$(ls -d ~/.mozilla/firefox/*.osint 2>/dev/null | head -1)

# Apply Hardening
cat > "$PROFILE_PATH/user.js" << 'EOF'
// FINGERPRINTING PROTECTION
user_pref("privacy.resistFingerprinting", true);
user_pref("privacy.resistFingerprinting.letterboxing", true);
user_pref("privacy.trackingprotection.fingerprinting.enabled", true);

// WEBRTC LEAK PREVENTION
user_pref("media.peerconnection.enabled", false);
user_pref("media.peerconnection.ice.proxy_only", true);

// CANVAS PROTECTION
user_pref("webgl.disabled", true);
user_pref("canvas.capturestream.enabled", false);

// GEOLOCATION & SENSORS
user_pref("geo.enabled", false);
user_pref("device.sensors.enabled", false);
user_pref("dom.battery.enabled", false);

// CACHE & STORAGE
user_pref("browser.cache.disk.enable", false);
user_pref("browser.cache.memory.enable", false);
user_pref("privacy.clearOnShutdown.history", true);
user_pref("privacy.sanitize.sanitizeOnShutdown", true);

// TELEMETRY
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("toolkit.telemetry.enabled", false);
EOF

# Launch Alias
alias osint-browser='firefox -P OSINT-Profile -no-remote --private-window'
Hardened Chromium Launcher
bash
sudo tee /usr/local/bin/chromium-osint << 'EOF'
#!/bin/bash
TEMP_PROFILE=$(mktemp -d)
chromium --temp-profile \
  --user-data-dir="$TEMP_PROFILE" \
  --incognito \
  --disable-logging \
  --disable-reading-from-canvas \
  --disable-3d-apis \
  --disable-webgl \
  --disable-webrtc \
  --no-pings \
  "$@"
trap "rm -rf $TEMP_PROFILE" EXIT
EOF
sudo chmod +x /usr/local/bin/chromium-osint
📂 6. Metadata & Artifact Sanitization
Remove identifying information from all files before deployment.

bash
# Install Tools
sudo apt install mat2 exiftool wipe secure-delete -y

# Metadata Scrubber Function
scrub_metadata() {
    local target="${1:-.}"
    echo "[*] Scrubbing metadata in: $target"
    
    # PDFs
    find "$target" -type f -name "*.pdf" -exec exiftool -all= {} \; 2>/dev/null
    find "$target" -type f -name "*.pdf" -exec mat2 {} \; 2>/dev/null
    
    # Office Documents
    find "$target" -type f \( -name "*.docx" -o -name "*.xlsx" -o -name "*.pptx" \) \
        -exec exiftool -all= {} \; 2>/dev/null
    
    # Images
    find "$target" -type f \( -name "*.jpg" -o -name "*.png" \) \
        -exec exiftool -all= {} \; 2>/dev/null
    
    echo "[✓] Metadata scrubbing complete"
}

# File Shredding Function
secure_wipe() {
    for file in "$@"; do
        if [ -f "$file" ]; then
            shred -zvu "$file"
        fi
    done
}
Artifact Cleaning Command
bash
opsec_clean() {
    echo "[*] Cleaning artifacts..."
    
    # Clear History
    history -c 2>/dev/null
    history -w 2>/dev/null
    cat /dev/null > ~/.bash_history 2>/dev/null
    
    # Secure Delete Temp Files
    find /tmp -type f -exec shred -zvu {} \; 2>/dev/null
    find /var/tmp -type f -exec shred -zvu {} \; 2>/dev/null
    
    # Wipe Logs (selective)
    if [ "$EUID" -eq 0 ]; then
        journalctl --rotate 2>/dev/null
        journalctl --vacuum-time=1s 2>/dev/null
        cat /dev/null > /var/log/syslog 2>/dev/null
        cat /dev/null > /var/log/auth.log 2>/dev/null
    fi
    
    echo "[✓] Cleanup complete"
}
📦 7. Containerization & Isolation
Use containers for clean-slate operations with zero host artifacts.

bash
# Install Docker
sudo apt install docker.io -y
sudo systemctl enable docker

# Pull Kali Image
docker pull kalilinux/kali-rolling

# Ephemeral Container Launcher
cat > ~/kali-container.sh << 'EOF'
#!/bin/bash
docker run -it --rm \
  --name kali-ops-$$ \
  --cap-drop=ALL \
  --cap-add=NET_RAW \
  --cap-add=NET_ADMIN \
  --security-opt=no-new-privileges:true \
  --network=host \
  -v $PWD:/workspace \
  -w /workspace \
  kalilinux/kali-rolling bash
EOF
chmod +x ~/kali-container.sh

# Docker Hardening
sudo tee /etc/docker/daemon.json << EOF
{
  "icc": false,
  "log-driver": "journald",
  "live-restore": false,
  "no-new-privileges": true,
  "userns-remap": "default"
}
EOF
sudo systemctl restart docker
⏱️ 8. Time-Based OPSEC & Pattern Avoidance
Avoid creating predictable patterns in your operations.

bash
# Install Timing Tools
sudo apt install sleep-random tc -y

# Random Delay Function
random_delay() {
    local max=${1:-5}
    local min=${2:-1}
    local delay=$((RANDOM % (max - min + 1) + min))
    echo "[*] Random delay: ${delay}s"
    sleep $delay
}

# Add Network Jitter
sudo tc qdisc add dev $IFACE root netem delay 100ms 50ms distribution normal

# Timezone Anonymization
sudo timedatectl set-timezone UTC

# Command Timing Obfuscation
alias nmap='random_delay; nmap'
alias masscan='random_delay; masscan'
alias curl='random_delay; curl'
🚀 9. Automated Hardening Script
Complete harden.sh - One script to rule them all.

bash
#!/bin/bash
# 🥷 Complete OPSEC Hardening Script
# Version: 2.0 (2026)

set -e
echo "═══════════════════════════════════════════"
echo "🥷 STARTING OPSEC HARDENING"
echo "═══════════════════════════════════════════"

# System Hardening
echo "[*] System Hardening..."
sudo apt update && sudo apt upgrade -y
sudo apt install linux-kernel-hardening-checker macchanger mat2 exiftool \
    wireguard tor proxychains4 docker.io firefox-esr -y

# Kernel Hardening
echo "[*] Kernel Hardening..."
sudo tee -a /etc/sysctl.conf < opsec-sysctl.conf
sudo sysctl -p

# MAC Spoofing
echo "[*] MAC Spoofing Setup..."
IFACE=$(ip route | grep default | awk '{print $5}' | head -1)
sudo systemctl enable macspoof@$IFACE.service 2>/dev/null || true

# DNS Hardening
echo "[*] DNS Hardening..."
sudo systemctl enable --now stubby dnsmasq 2>/dev/null || true
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
sudo chattr +i /etc/resolv.conf 2>/dev/null

# Browser Hardening
echo "[*] Browser Hardening..."
firefox -CreateProfile "OSINT-Profile" 2>/dev/null || true

# Add Functions to .bashrc
cat >> ~/.bashrc << 'EOF'

# OPSEC Functions
alias osint='firefox -P OSINT-Profile -no-remote --private-window'
alias randmac='sudo systemctl restart macspoof@$IFACE'
alias opsec-check='check_identity && dnsleaktest'
alias opsec-clean='history -c && find /tmp -type f -exec shred -zvu {} \; 2>/dev/null'

# Random Delay
rd() { sleep $((RANDOM % 3 + 1)); }
alias nmap='rd; nmap'
EOF

source ~/.bashrc

echo "═══════════════════════════════════════════"
echo "✅ HARDENING COMPLETE - Run 'opsec-check' to verify"
echo "═══════════════════════════════════════════"
📋 10. Quick Reference Cards
Boot-Time Commands (Run Immediately)
Action	Command	Purpose
Spoof MAC	sudo systemctl restart macspoof@eth0	Change hardware address
Random Hostname	sudo hostnamectl set-hostname $(random-hostname)	Change system name
Disable IPv6	sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1	Prevent IPv6 leaks
Start VPN	sudo wg-quick up mullvad-us1	Low-latency tunnel
Start Tor	sudo perl nipe.pl start	Anonymous routing
Network Mode Toggles
Mode	Command	When to Use
VPN (Speed)	sudo wg-quick up mullvad-us1	Exploitation, scanning, brute-force
Tor (Ghost)	sudo perl nipe.pl start	Recon, OSINT, web browsing
ProxyChains	proxychains4 nmap -sT target	Tool-specific routing
DNS over TLS	Auto-enabled	All times (prevents leaks)
Browser Launchers
Browser	Command	Protection Level
Firefox OSINT	osint	Canvas blocking, WebRTC disable, fingerprint resistant
Chromium OSINT	chromium-osint	Temp profile, all tracking disabled
Tor Browser	~/tor-browser-osint.sh	Maximum anonymity
Artifact Cleanup
Command	What It Cleans	Safety
opsec_clean	History, temp files, logs	Complete wipe
scrub_metadata ./payloads	EXIF data from files	File-specific
sudo journalctl --vacuum-time=1s	System logs	Use with caution
Identity Verification
Command	What It Shows	Red Flags
check_identity	Hostname, MAC, machine ID, timezone	Original MAC visible
dnsleaktest	DNS servers, external IP, leaks	External DNS queries
ip a | grep ether	MAC addresses	Permanent address showing
✅ 11. Complete Verification Checklist
Run before any engagement:

bash
#!/bin/bash
# pre-engagement-check.sh

echo "═══════════════════════════════════════════"
echo "🔐 PRE-ENGAGEMENT OPSEC CHECKLIST"
echo "═══════════════════════════════════════════"

PASS=0
FAIL=0

check() {
    if [ $? -eq 0 ]; then echo "✅ $1"; PASS=$((PASS+1)); 
    else echo "❌ $1"; FAIL=$((FAIL+1)); fi
}

# Network Identity
CURRENT_IP=$(curl -s ifconfig.me)
echo "[1] External IP: $CURRENT_IP"

# MAC Check
ip link | grep -q "permaddr" || check "MAC address is spoofed"

# IPv6 Check
sysctl net.ipv6.conf.all.disable_ipv6 | grep -q "= 1" && check "IPv6 is disabled"

# DNS Check
dig +short whoami.akamai.net | grep -q "$CURRENT_IP" && check "DNS not leaking"

# Timezone
[ "$(timedatectl | grep "Time zone" | awk '{print $3}')" = "UTC" ] && check "Timezone is UTC"

# Cleanliness
[ ! -f ~/.bash_history ] || [ $(wc -l < ~/.bash_history) -lt 5 ] && check "History cleared"

echo "═══════════════════════════════════════════"
echo "✅ Passed: $PASS  ❌ Failed: $FAIL"
[ $FAIL -eq 0 ] && echo "🔓 READY FOR ENGAGEMENT" || echo "⚠️  REVIEW FAILURES"
🎯 Pro Tips
Boot Order Matters: Run MAC spoofing before network connects

Mode Switching: Always sudo perl nipe.pl stop before VPN use

File Hygiene: Scrub metadata before transferring any file

Time Patterns: Vary scan times with random_delay

Container First: Run unknown tools in Docker, never bare metal

⚠️ Legal Disclaimer
This guide is intended for educational purposes and authorized security testing only. Always ensure you have proper written permission before testing or hardening any system you do not own. The author is not responsible for misuse, illegal activities, or any damages resulting from the use of this information.

By using this guide, you acknowledge that you are solely responsible for complying with all applicable local, state, and federal laws.
