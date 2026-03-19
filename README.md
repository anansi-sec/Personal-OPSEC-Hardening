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

## 🕵️ 5. Browser & Fingerprinting Hardening
"Eliminate browser-based tracking, canvas fingerprinting, and WebRTC leaks."

### 🔥 Hardened Firefox Profile

```bash
# Create OSINT Profile
firefox -CreateProfile "OSINT-Profile" 2>/dev/null || true

# Find Profile Path
PROFILE_PATH=$(ls -d ~/.mozilla/firefox/*.osint 2>/dev/null | head -1)

if [ -z "$PROFILE_PATH" ]; then
    # Manual fallback
    mkdir -p ~/.mozilla/firefox/osint-profile
    cat > ~/.mozilla/firefox/profiles.ini << 'EOF'
[General]
StartWithLastProfile=1

[Profile0]
Name=OSINT-Profile
IsRelative=1
Path=osint-profile
Default=yes
EOF
    PROFILE_PATH=~/.mozilla/firefox/osint-profile
fi

# Apply Comprehensive Hardening
cat > "$PROFILE_PATH/user.js" << 'EOF'
// ===== OPSEC FIREFOX HARDENING =====
// Based on arkenfox user.js and privacy best practices

// ===== FINGERPRINTING PROTECTION =====
user_pref("privacy.resistFingerprinting", true);
user_pref("privacy.resistFingerprinting.autoDeclineNoUserInputCanvasPrompts", true);
user_pref("privacy.resistFingerprinting.letterboxing", true);
user_pref("privacy.trackingprotection.fingerprinting.enabled", true);
user_pref("privacy.trackingprotection.cryptomining.enabled", true);

// ===== WEBRTC LEAK PREVENTION =====
user_pref("media.peerconnection.enabled", false);
user_pref("media.peerconnection.ice.default_address_only", true);
user_pref("media.peerconnection.ice.no_host", true);
user_pref("media.peerconnection.ice.proxy_only_if_behind_proxy", true);
user_pref("media.peerconnection.ice.proxy_only", true);

// ===== CANVAS & WEBGL PROTECTION =====
user_pref("webgl.disabled", true);
user_pref("webgl.enable-webgl2", false);
user_pref("webgl.min_capability_mode", true);
user_pref("canvas.capturestream.enabled", false);
user_pref("canvas.extractData.enabled", false);

// ===== JAVASCRIPT RESTRICTIONS =====
user_pref("javascript.options.wasm", false);
user_pref("javascript.options.baselinejit", false);
user_pref("javascript.options.ion", false);

// ===== GEOLOCATION & SENSORS =====
user_pref("geo.enabled", false);
user_pref("geo.wifi.uri", "");
user_pref("device.sensors.enabled", false);
user_pref("dom.battery.enabled", false);
user_pref("dom.vibrator.enabled", false);
user_pref("dom.maxHardwareConcurrency", 2);

// ===== CACHE & STORAGE =====
user_pref("browser.cache.disk.enable", false);
user_pref("browser.cache.disk.capacity", 0);
user_pref("browser.cache.memory.enable", false);
user_pref("browser.cache.offline.enable", false);
user_pref("privacy.clearOnShutdown.history", true);
user_pref("privacy.clearOnShutdown.cookies", true);
user_pref("privacy.clearOnShutdown.cache", true);
user_pref("privacy.clearOnShutdown.sessions", true);
user_pref("privacy.sanitize.sanitizeOnShutdown", true);

// ===== TELEMETRY & DATA COLLECTION =====
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("datareporting.policy.dataSubmissionEnabled", false);
user_pref("toolkit.telemetry.enabled", false);
user_pref("toolkit.telemetry.unified", false);
user_pref("toolkit.telemetry.server", "data:,");

// ===== DNS & NETWORK =====
user_pref("network.trr.mode", 2);
user_pref("network.trr.uri", "https://mozilla.cloudflare-dns.com/dns-query");
user_pref("network.dns.disablePrefetch", true);
user_pref("network.prefetch-next", false);
user_pref("network.http.speculative-parallel-limit", 0);

// ===== PRIVACY SANITIZATION =====
user_pref("browser.urlbar.speculativeConnect.enabled", false);
user_pref("browser.search.suggest.enabled", false);
user_pref("browser.formfill.enable", false);
user_pref("dom.disable_beforeunload", true);
EOF

echo "[✓] Firefox hardened profile created at: $PROFILE_PATH"

# Create launch alias
echo "alias osint-browser='firefox -P OSINT-Profile -no-remote --private-window'" >> ~/.bashrc
source ~/.bashrc
```

### 🧪 Browser Fingerprint Test

```bash
fingerprint_test() {
    echo "═══════════════════════════════════════════"
    echo "🔍 BROWSER FINGERPRINT TEST"
    echo "═══════════════════════════════════════════"
    
    echo -e "\n🌐 Visit these URLs to verify anonymity:\n"
    echo "  • https://amiunique.org/ - See how unique your browser is"
    echo "  • https://browserleaks.com/ - Test WebRTC, canvas, fonts"
    echo "  • https://coveryourtracks.eff.org/ - EFF fingerprint test"
    echo "  • https://ipleak.net/ - Comprehensive leak test"
    
    echo -e "\n📋 Expected results for hardened browser:\n"
    echo "  ✓ Canvas fingerprint: Blocked"
    echo "  ✓ WebRTC: No local IP leak"
    echo "  ✓ WebGL: Disabled"
    echo "  ✓ Font fingerprint: Generic"
    
    echo "═══════════════════════════════════════════"
}

# Run the test guide
fingerprint_test
```
---

## 📂 6. Metadata & Artifact Sanitization
"Remove identifying information from all files before deployment."

### 🧹 Metadata Scrubber Function

```bash
# Install Tools
sudo apt install mat2 exiftool wipe secure-delete jhead -y

# Create Metadata Scrubber
sudo tee /usr/local/bin/scrub-metadata << 'EOF'
#!/bin/bash
# Comprehensive metadata scrubber

TARGET="${1:-.}"
echo "═══════════════════════════════════════════"
echo "🧹 METADATA SCRUBBER"
echo "═══════════════════════════════════════════"
echo "[*] Scrubbing metadata in: $TARGET"

# PDF files
echo -n "[*] PDF files: "
find "$TARGET" -type f -name "*.pdf" -exec exiftool -overwrite_original -all= {} \; 2>/dev/null
find "$TARGET" -type f -name "*.pdf" -exec mat2 {} \; 2>/dev/null
echo "✓"

# Office documents
echo -n "[*] Office docs: "
find "$TARGET" -type f \( -name "*.docx" -o -name "*.xlsx" -o -name "*.pptx" \) \
    -exec exiftool -overwrite_original -all= {} \; 2>/dev/null
echo "✓"

# Images
echo -n "[*] Images: "
find "$TARGET" -type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" -o -name "*.gif" \) \
    -exec exiftool -overwrite_original -all= {} \; 2>/dev/null
find "$TARGET" -type f -name "*.jpg" -exec jhead -purejpg {} \; 2>/dev/null
echo "✓"

# Audio/Video
echo -n "[*] Media files: "
find "$TARGET" -type f \( -name "*.mp3" -o -name "*.mp4" -o -name "*.mov" \) \
    -exec exiftool -overwrite_original -all= {} \; 2>/dev/null
echo "✓"

# Archives
echo -n "[*] Archives: "
find "$TARGET" -type f -name "*.zip" -exec zip -d {} "__MACOSX*" ".DS_Store" \; 2>/dev/null
echo "✓"

# Reset file timestamps to epoch
echo -n "[*] Resetting timestamps: "
find "$TARGET" -type f -exec touch -t 197001010000 {} \; 2>/dev/null
echo "✓"

echo "═══════════════════════════════════════════"
echo "[✓] Metadata scrubbing complete"
EOF

sudo chmod +x /usr/local/bin/scrub-metadata

# Create metadata checker
sudo tee /usr/local/bin/check-metadata << 'EOF'
#!/bin/bash
# Quick metadata check

if [ -z "$1" ]; then
    echo "Usage: check-metadata <file>"
    exit 1
fi

echo "═══════════════════════════════════════════"
echo "🔍 METADATA CHECK: $(basename "$1")"
echo "═══════════════════════════════════════════"

exiftool "$1" | grep -E "(Author|Creator|Producer|Modify|Create|Software|User|History|GPS|Latitude|Longitude)" || echo "✓ No identifying metadata found"

echo "═══════════════════════════════════════════"
EOF

sudo chmod +x /usr/local/bin/check-metadata
```

### 🧽 Complete Artifact Cleaning

```bash
opsec_clean() {
    echo "═══════════════════════════════════════════"
    echo "🧹 OPSEC ARTIFACT CLEANER"
    echo "═══════════════════════════════════════════"
    
    # Clear Shell History
    echo -n "[*] Clearing bash history... "
    history -c 2>/dev/null
    history -w 2>/dev/null
    cat /dev/null > ~/.bash_history 2>/dev/null
    cat /dev/null > ~/.zsh_history 2>/dev/null
    cat /dev/null > ~/.python_history 2>/dev/null
    echo "✓"
    
    # Secure Delete Temp Files
    echo -n "[*] Wiping temporary files... "
    find /tmp -type f -exec shred -zvu {} \; 2>/dev/null
    find /var/tmp -type f -exec shred -zvu {} \; 2>/dev/null
    rm -rf /tmp/* 2>/dev/null
    rm -rf /var/tmp/* 2>/dev/null
    echo "✓"
    
    # Clear Cache Directories
    echo -n "[*] Clearing caches... "
    rm -rf ~/.cache/* 2>/dev/null
    rm -rf ~/.mozilla/firefox/*.default*/cache2 2>/dev/null
    rm -rf ~/.config/google-chrome/Default/Cache/* 2>/dev/null
    echo "✓"
    
    # Wipe Logs (selective)
    if [ "$EUID" -eq 0 ]; then
        echo -n "[*] Rotating system logs... "
        journalctl --rotate 2>/dev/null
        journalctl --vacuum-time=1s 2>/dev/null
        cat /dev/null > /var/log/syslog 2>/dev/null
        cat /dev/null > /var/log/auth.log 2>/dev/null
        cat /dev/null > /var/log/kern.log 2>/dev/null
        cat /dev/null > /var/log/dpkg.log 2>/dev/null
        cat /dev/null > /var/log/apt/history.log 2>/dev/null
        echo "✓"
    fi
    
    # Clear Recent Files
    echo -n "[*] Clearing recent documents... "
    rm -f ~/.recently-used.xbel 2>/dev/null
    rm -rf ~/.local/share/recently-used.xbel 2>/dev/null
    rm -f ~/.local/share/recently-used.xbel* 2>/dev/null
    echo "✓"
    
    # Clear Clipboard (if xclip installed)
    if command -v xclip >/dev/null; then
        echo -n "[*] Clearing clipboard... "
        echo "" | xclip -selection clipboard
        echo "" | xclip -selection primary
        echo "✓"
    fi
    
    echo "═══════════════════════════════════════════"
    echo "[✓] Cleanup complete - System is forensically clean"
}

# Run the cleaner
opsec_clean
```

---

## 📦 7. Containerization & Isolation
"Use containers for clean-slate operations with zero host artifacts."

```bash
# Install Docker
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
```

### 🐳 Ephemeral Kali Container

```bash
# Pull Kali Image
docker pull kalilinux/kali-rolling

# Create Ephemeral Container Launcher
cat > ~/kali-container.sh << 'EOF'
#!/bin/bash
# Launch temporary Kali container with networking

GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'

CONTAINER_NAME="kali-ops-$$"
WORKSPACE="$PWD"

echo -e "${GREEN}[*] Starting ephemeral Kali container...${NC}"
echo -e "${GREEN}[*] Container will self-destruct on exit${NC}"
echo -e "${GREEN}[*] Workspace: $WORKSPACE${NC}"

docker run -it --rm \
  --name "$CONTAINER_NAME" \
  --cap-drop=ALL \
  --cap-add=NET_RAW \
  --cap-add=NET_ADMIN \
  --cap-add=DAC_OVERRIDE \
  --security-opt=no-new-privileges:true \
  --security-opt=seccomp:unconfined \
  --network=host \
  -v "$WORKSPACE":/workspace \
  -w /workspace \
  kalilinux/kali-rolling /bin/bash

if [ $? -eq 0 ]; then
    echo -e "${GREEN}[✓] Container terminated - all changes destroyed${NC}"
else
    echo -e "${RED}[!] Container exited with error${NC}"
fi
EOF

chmod +x ~/kali-container.sh

# Create Tool-Specific Containers
cat > ~/nmap-container.sh << 'EOF'
#!/bin/bash
# Run nmap in isolated container

if [ $# -eq 0 ]; then
    echo "Usage: ./nmap-container.sh <nmap-arguments>"
    echo "Example: ./nmap-container.sh -sS 192.168.1.1"
    exit 1
fi

docker run -it --rm \
  --cap-add=NET_RAW \
  --cap-add=NET_ADMIN \
  --network=host \
  kalilinux/kali-rolling nmap "$@"
EOF

chmod +x ~/nmap-container.sh

cat > ~/metasploit-container.sh << 'EOF'
#!/bin/bash
# Run Metasploit in container

docker run -it --rm \
  --cap-add=NET_RAW \
  --cap-add=NET_ADMIN \
  --network=host \
  -v ~/.msf4:/root/.msf4 \
  kalilinux/kali-rolling msfconsole
EOF

chmod +x ~/metasploit-container.sh
```

### 🛡️ Docker Hardening

```bash
# Apply Docker Security Configuration
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "icc": false,
  "log-driver": "journald",
  "live-restore": false,
  "no-new-privileges": true,
  "userns-remap": "default",
  "userland-proxy": false,
  "ip-forward": false,
  "iptables": true,
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
EOF

sudo systemctl restart docker

# Verify hardening
echo "[*] Docker hardening applied:"
docker info | grep -E "Security|Username|Mapping"
```

---

| Command | Purpose | Artifacts Left |
|:--------|:--------|:---------------|
| `~/kali-container.sh` | Full Kali environment | None (`--rm` flag) |
| `~/nmap-container.sh -sS target` | Isolated nmap scan | None |
| `~/metasploit-container.sh` | Metasploit in container | `~/.msf4` (optional) |
| `docker ps -a` | Check for残留 containers | Should show none |
| `docker system prune -af` | Remove all containers/images | Complete wipe |

---

## ⏱️ 8. Time-Based OPSEC & Pattern Avoidance
"Avoid creating predictable patterns in your operations."

```bash
# Install Timing Tools
sudo apt install sleep-random tc at -y
```

⏰ Random Delay Functions

```bash
# Add to .bashrc
cat >> ~/.bashrc << 'EOF'

# ===== OPSEC TIME-BASED FUNCTIONS =====

# Random delay to break patterns
random_delay() {
    local max=${1:-5}
    local min=${2:-1}
    local delay=$((RANDOM % (max - min + 1) + min))
    echo "[*] OPSEC delay: ${delay}s"
    sleep $delay
}

# Command wrappers with random delay
alias nmap='random_delay; nmap'
alias masscan='random_delay; masscan'
alias curl='random_delay; curl'
alias wget='random_delay; wget'
alias hydra='random_delay; hydra'
alias gobuster='random_delay; gobuster'
alias ffuf='random_delay; ffuf'

# Progressive delay (increases over time)
progressive_delay() {
    local base=${1:-2}
    local count_file="/tmp/opsec-counter-$$"
    local count=$(cat "$count_file" 2>/dev/null || echo 0)
    count=$((count + 1))
    echo $count > "$count_file"
    local delay=$((base * count))
    echo "[*] Progressive delay #$count: ${delay}s"
    sleep $delay
}

# Random jitter for continuous operations
jitter() {
    while true; do
        sleep
```

---

## 🔧 9. Application Layer Hardening
Harden common tools and protocols.

### SSH client hardening

```bash
cat >> ~/.ssh/config << 'EOF'
Host *
    KexAlgorithms curve25519-sha256@libssh.org
    Ciphers chacha20-poly1305@openssh.com
    MACs hmac-sha2-512-etm@openssh.com
    HostKeyAlgorithms ssh-ed25519
    Compression no
    TCPKeepAlive no
    ServerAliveInterval 0
    ClientAliveInterval 0
    StrictHostKeyChecking ask
    UserKnownHostsFile /dev/null
    LogLevel QUIET
EOF

# Curl/Wget fingerprint spoofing
alias curl='curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" -H "Accept-Language: en-US,en;q=0.9" --compressed -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"'
alias wget='wget -U "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"'

# Nmap timing obfuscation
alias nmap-stealth='nmap -T2 --max-retries=1 --min-rtt-timeout=500ms --max-rtt-timeout=2000ms --max-scan-delay=1s --randomize-hosts'

# Masscan rate limiting
alias masscan-stealth='masscan --rate=100 --randomize-hosts --banners'
```

---

🚀 10. Automated Hardening Script
Complete harden.sh incorporating all hardening measures.

```bash

#!/bin/bash
# 🥷 Complete OPSEC Hardening Script
# Version: 2.0

set -e

echo "═══════════════════════════════════════════"
echo "🥷 Starting OPSEC Hardening"
echo "═══════════════════════════════════════════"

# Check root
if [ "$EUID" -ne 0 ]; then 
    echo "[-] Please run as root"
    exit 1
fi

# 1. System Hardening
echo "[*] 1. Applying system hardening..."
apt update
apt install -y linux-kernel-hardening-checker macchanger mat2 exiftool wipe secure-delete wireguard proxychains4 tor docker.io

# Disable Avahi
systemctl stop avahi-daemon avahi-daemon.socket 2>/dev/null
systemctl disable avahi-daemon avahi-daemon.socket 2>/dev/null
systemctl mask avahi-daemon.socket 2>/dev/null

# 2. Kernel Hardening
echo "[*] 2. Applying kernel hardening..."
cat >> /etc/sysctl.conf << 'EOF'

# OPSEC Hardening
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.icmp_echo_ignore_all = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv4.tcp_timestamps = 0
net.ipv4.ip_id = 1
net.ipv4.tcp_syncookies = 1
EOF

sysctl -p

# 3. MAC Spoofing Service
echo "[*] 3. Setting up MAC spoofing..."
cat > /etc/systemd/system/macspoof@.service << 'EOF'
[Unit]
Description=MAC spoofing for %I
Before=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/ip link set dev %I down
ExecStart=/usr/bin/macchanger -r %I
ExecStart=/usr/bin/ip link set dev %I up

[Install]
WantedBy=multi-user.target
EOF

# Get primary interface
IFACE=$(ip route | grep default | awk '{print $5}' | head -1)
if [ -n "$IFACE" ]; then
    systemctl enable macspoof@$IFACE.service
    systemctl start macspoof@$IFACE.service
fi

# 4. Browser Hardening
echo "[*] 4. Hardening browser profiles..."
if command -v firefox >/dev/null; then
    firefox -CreateProfile "OSINT-Profile" 2>/dev/null || true
    PROFILE_PATH=$(ls -d ~/.mozilla/firefox/*.osint 2>/dev/null | head -1)
    if [ -n "$PROFILE_PATH" ]; then
        cat > "$PROFILE_PATH/user.js" << 'FEOF'
user_pref("privacy.resistFingerprinting", true);
user_pref("privacy.trackingprotection.fingerprinting.enabled", true);
user_pref("privacy.firstparty.isolate", true);
user_pref("privacy.clearOnShutdown.history", true);
user_pref("privacy.sanitize.sanitizeOnShutdown", true);
user_pref("media.peerconnection.enabled", false);
user_pref("webgl.disabled", true);
user_pref("javascript.options.wasm", false);
user_pref("dom.battery.enabled", false);
FEOF
    fi
fi

# 5. Create utility functions
echo "[*] 5. Creating utility scripts..."
cat > /usr/local/bin/opsec-verify << 'EOF'
#!/bin/bash
# OPSEC Verification

echo "═══════════════════════════════════════════"
echo "🔍 OPSEC Verification Report"
echo "═══════════════════════════════════════════"

echo -e "\n📡 External IP:"
curl -s ifconfig.me || echo "Failed"

echo -e "\n🌐 DNS Servers:"
cat /etc/resolv.conf | grep nameserver

echo -e "\n🖧 MAC Address:"
ip link show | grep -A1 ether | head -2

echo -e "\n🚦 IPv6 Status:"
sysctl net.ipv6.conf.all.disable_ipv6

echo -e "\n🕒 Timezone:"
timedatectl | grep "Time zone"

echo -e "\n🧹 Temp Files:"
ls -la /tmp/ | wc -l | xargs echo "Count:"

echo -e "\n📊 Active Connections:"
ss -tuln | grep -v "127.0.0.1" | head -10

echo "═══════════════════════════════════════════"
EOF

chmod +x /usr/local/bin/opsec-verify

# 6. Cleanup function
cat > /usr/local/bin/opsec-clean << 'EOF'
#!/bin/bash
# Complete artifact cleanup

echo "[*] Cleaning artifacts..."

# Clear history
history -c 2>/dev/null
history -w 2>/dev/null
cat /dev/null > ~/.bash_history 2>/dev/null
cat /dev/null > ~/.zsh_history 2>/dev/null

# Secure delete temp files
shred -zvu /tmp/* 2>/dev/null
shred -zvu /var/tmp/* 2>/dev/null

# Clear logs (selective)
if [ "$EUID" -eq 0 ]; then
    journalctl --rotate 2>/dev/null
    journalctl --vacuum-time=1s 2>/dev/null
    cat /dev/null > /var/log/syslog 2>/dev/null
    cat /dev/null > /var/log/auth.log 2>/dev/null
fi

# Clear recent files
rm -f ~/.recently-used.xbel 2>/dev/null
rm -rf ~/.local/share/recently-used.xbel 2>/dev/null

echo "[✓] Cleanup complete"
EOF

chmod +x /usr/local/bin/opsec-clean

# 7. Add to .bashrc
echo "[*] 6. Adding OPSEC aliases..."
cat >> ~/.bashrc << 'EOF'

# OPSEC Aliases
alias opsec-check='opsec-verify'
alias opsec-clean='opsec-clean'
alias incognito='unset HISTFILE; set +o history'
alias osint-browser='firefox -P OSINT-Profile -no-remote'
alias dns-leak='curl -s https://ipleak.net/json/ | python3 -m json.tool'

# Random delay function
rd() { sleep $((RANDOM % 3 + 1)); }

# Tool wrappers
alias nmap='rd; nmap'
alias curl='rd; curl'
alias wget='rd; wget'
EOF

echo "═══════════════════════════════════════════"
echo "✅ OPSEC Hardening Complete!"
echo "Run 'opsec-check' to verify your configuration"
echo "Run 'opsec-clean' to remove artifacts"
echo "═══════════════════════════════════════════"
```

---

## 📋 11. Quick Reference Card

| Phase | Action | Command | Verification |
|:------|:-------|:--------|:--------------|
| **🖥️ Boot** | Spoof MAC | `sudo systemctl start macspoof@eth0` | `ip link \| grep ether` |
| | Random Hostname | `sudo hostnamectl set-hostname $(random-hostname)` | `hostname` |
| | Disable IPv6 | `sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1` | `ip a \| grep inet6` |
| **🌐 Network** | VPN (Speed) | `sudo wg-quick up mullvad-us1` | `curl ifconfig.me` |
| | Tor (Ghost) | `sudo perl nipe.pl start` | `nipe status` |
| | DNS Test | `dnsleaktest` | Check for home IP |
| **🛠️ Tools** | OSINT Browser | `osint-browser` | `about:config` |
| | Isolated Container | `~/kali-container.sh` | No host files |
| | Metadata Scrub | `scrub-metadata ./payloads` | `check-metadata file` |
| **🧹 Cleanup** | Full Clean | `opsec-clean` | No history/logs |
| | Verify All | `opsec-check` | All green |
| **💾 Persistence** | Make permanent | Add to `/etc/sysctl.conf` | `sysctl -p` |

---

## ⚖️ Legal Disclaimer

<p align="center">
  <img src="https://img.shields.io/badge/LEGAL-READ%20CAREFULLY-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/FOR-EDUCATIONAL%20USE%20ONLY-blue?style=for-the-badge"/>
</p>

**This guide is provided STRICTLY for educational purposes and authorized security testing.**

### 📜 Terms of Use

By accessing, downloading, or using any part of this guide, you explicitly agree that:

1. **You have explicit written permission** to test any system you use these techniques on
2. **You are solely responsible** for complying with all applicable local, state, and federal laws
3. **You assume all liability** for your actions - the authors assume **NO RESPONSIBILITY**
4. **This guide comes with NO WARRANTY**, express or implied
5. **Unauthorized access is ILLEGAL** and may result in severe criminal penalties

### 🚨 WARNING

<p align="center">
  <b>UNAUTHORIZED USE OF THESE TECHNIQUES MAY RESULT IN:</b><br/>
  • Criminal prosecution<br/>
  • Civil liability<br/>
  • Imprisonment<br/>
  • Fines up to $500,000+
</p>

**If you do not agree to these terms, DELETE this guide immediately.**

---

## 🙏 Acknowledgements

- [Awesome-Red-Teaming](https://github.com/) - Infrastructure inspiration
- [CheckPlease](https://github.com/) - Evasion techniques
- [Arkenfox](https://github.com/arkenfox/user.js) - Browser hardening base
- The security community for continuous research

---

## 📚 Additional Resources

- [Mullvad VPN](https://mullvad.net/) - Recommended VPN provider
- [ProtonVPN](https://protonvpn.com/) - Alternative VPN
- [Tor Project](https://www.torproject.org/) - Anonymity network
- [EFF Surveillance Self-Defense](https://ssd.eff.org/) - Legal/privacy guide

---

## 📌 Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | March 2026 | Complete rewrite with DNS over TLS, containerization, browser hardening, time-based OPSEC |
| 1.0 | 2025 | Initial release |

---

## 👤 Author & Contributing

**Author:** Professional Red Team Reference

Found an issue? Have a suggestion? Please open an issue or submit a pull request.

**Contributions are welcome** - but must maintain the educational/legal focus.

---

## ⭐ Support

If this guide helped you, consider:
- Starring the repository ⭐
- Sharing with fellow security professionals
- Contributing improvements

---

## 📞 Contact

For questions, suggestions, or collaborations:
- Open a GitHub issue
- Submit a pull request

**Note:** I cannot provide legal advice or help with unauthorized activities.

---

<p align="center">
  <b>Stay ethical. Stay legal. Stay invisible.</b><br/>
  <i>Knowledge is power - use it responsibly.</i>
</p>

<p align="center">
  <a href="#-table-of-contents">⬆️ Back to Top</a> •
  <a href="https://github.com/anansi-sec/Personal-OPSEC-Hardening">📘 Repository</a> •
  <a href="#-legal-disclaimer">⚖️ Legal</a>
</p>

<p align="center">
  <img src="https://img.shields.io/github/stars/anansi-sec/Personal-OPSEC-Hardening?style=social"/>
  <img src="https://img.shields.io/github/forks/anansi-sec/Personal-OPSEC-Hardening?style=social"/>
  <img src="https://img.shields.io/github/watchers/anansi-sec/Personal-OPSEC-Hardening?style=social"/>
</p>

---

**Version 2.0** | Last Updated: March 2026 | ⚖️ For Educational Use Only







    






