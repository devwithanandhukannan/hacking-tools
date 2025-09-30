```markdown
# VPN Setup Guide for Kali Linux - Pentesting Edition

## Table of Contents
- [Understanding VPN Layers](#understanding-vpn-layers)
- [Free VPNs for Pentesting](#free-vpns-for-pentesting)
- [Installation Methods](#installation-methods)
  - [Method 1: ProtonVPN](#method-1-protonvpn)
  - [Method 2: VPNBook](#method-2-vpnbook)
- [Verification](#verification)

---

## Understanding VPN Layers

### Network Layer VPNs (Layer 3)
- **Protocols**: IPSec, WireGuard, PPTP
- **How it works**: Operates at the IP layer, encrypts entire packets
- **Characteristics**:
  - Encrypts all traffic at network level
  - More transparent to applications
  - Better performance
  - Works with all protocols (TCP/UDP/ICMP)

**IPSec Example:**
```
[Your Device] → IPSec Tunnel → [VPN Server] → [Internet]
     ↓
Encrypts at IP layer (Layer 3)
```

### Application Layer VPNs (Layer 7)
- **Protocols**: OpenVPN, SSL/TLS VPN, SOCKS proxy
- **How it works**: Operates at application layer using SSL/TLS
- **Characteristics**:
  - More flexible and configurable
  - Can bypass restrictive firewalls
  - Works on standard ports (443, 80)
  - Slightly more overhead

**OpenVPN Example:**
```
[Your Device] → SSL/TLS Tunnel → [VPN Server] → [Internet]
     ↓
Encrypts at application layer (Layer 7)
```

---

## Free VPNs for Pentesting

### ✅ Recommended Free Options

| VPN Service | Protocol | Data Limit | Speed | Logs | Pentesting Friendly |
|-------------|----------|------------|-------|------|---------------------|
| **ProtonVPN** | OpenVPN, IKEv2 | Unlimited | Medium | No logs | ✅ Yes |
| **VPNBook** | OpenVPN | Unlimited | Good | Minimal | ✅ Yes |
| **Hide.me** | OpenVPN, IKEv2 | 10GB/month | Good | No logs | ✅ Yes |
| **Windscribe** | OpenVPN, IKEv2 | 10GB/month | Good | Minimal | ⚠️ Limited |

### ⚠️ Important Notes for Pentesting:
- Always get **written permission** before pentesting
- Free VPNs may keep some logs
- Consider using multiple VPN chains for anonymity
- Test for DNS leaks after connection

---

## Installation Methods

### Prerequisites
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install OpenVPN
sudo apt install openvpn -y

# Install network-manager-openvpn for GUI
sudo apt install network-manager-openvpn network-manager-openvpn-gnome -y

# Restart NetworkManager
sudo systemctl restart NetworkManager
```

---

## Method 1: ProtonVPN

### Step-by-Step Setup

#### 1. Download Configuration Files
```bash
# Visit ProtonVPN website
https://account.protonvpn.com/downloads

# Or use CLI
wget https://account.protonvpn.com/api/vpn/config?platform=linux
```

#### 2. GUI Method (Easy)
1. **Click** on the network icon in the top bar
2. Navigate to: **VPN Connections** → **Add a VPN connection**
3. Select: **Import a saved VPN configuration**
4. Browse to your downloaded `.ovpn` file
5. **Enter credentials**:
   - Username: Your ProtonVPN username
   - Password: Your ProtonVPN password
6. Click **Add**

#### 3. Terminal Method (Advanced)
```bash
# Navigate to config directory
cd ~/Downloads/protonvpn-configs

# Connect using OpenVPN
sudo openvpn --config <server-name>.ovpn

# Example:
sudo openvpn --config us-free-01.protonvpn.udp.ovpn
```

#### 4. Save Credentials (Optional)
Create an auth file to avoid typing credentials:
```bash
# Create auth file
nano ~/vpn-auth.txt

# Add credentials (2 lines):
your-username
your-password

# Secure the file
chmod 600 ~/vpn-auth.txt

# Connect with saved auth
sudo openvpn --config server.ovpn --auth-user-pass ~/vpn-auth.txt
```

---

## Method 2: VPNBook

### Step-by-Step Setup

#### 1. Download Configuration Files
```bash
# Visit VPNBook
https://www.vpnbook.com/

# Navigate to OpenVPN section
# Download the OpenVPN configuration bundle
wget https://www.vpnbook.com/free-openvpn-account/VPNBook.com-OpenVPN-US1.zip

# Unzip the files
unzip VPNBook.com-OpenVPN-US1.zip
```

#### 2. Check Credentials on Website
- Go to https://www.vpnbook.com/
- **Username** and **Password** are displayed on the homepage
- **Note**: Passwords change periodically (check website)

#### 3. Import Configuration
```bash
# List available configs
ls *.ovpn

# Example output:
# vpnbook-us1-tcp443.ovpn
# vpnbook-us1-tcp80.ovpn
# vpnbook-us1-udp53.ovpn
```

#### 4. GUI Import
1. **Top bar** → **Network icon** → **VPN Connections**
2. Click **Add a VPN connection** → **Import from file**
3. Select the `.ovpn` file (e.g., `vpnbook-us1-tcp443.ovpn`)
4. Enter credentials from VPNBook website:
   - Username: `vpnbook`
   - Password: (from website)
5. Click **Save**

#### 5. Terminal Method
```bash
# Connect directly
sudo openvpn --config vpnbook-us1-tcp443.ovpn

# When prompted, enter:
# Username: vpnbook
# Password: (from website)
```

---

## Method 3: Hide.me (Bonus)

### Quick Setup
```bash
# Download config
wget https://hide.me/downloads/hide.me-OpenVPN-configs.zip

# Extract
unzip hide.me-OpenVPN-configs.zip

# Import via GUI or connect via terminal
sudo openvpn --config hide.me-<location>.ovpn
```

---

## Additional Free VPN Options

### 1. **Windscribe**
```bash
# Install Windscribe CLI
wget https://windscribe.com/install/desktop/linux_deb_x64
sudo dpkg -i windscribe*.deb

# Login and connect
windscribe login
windscribe connect
```

### 2. **TunnelBear** (Limited but reliable)
- 500MB/month free
- Good for quick tests
- Available via website config download

### 3. **Riseup VPN** (Privacy-focused)
```bash
# Install
sudo apt install riseup-vpn

# Run
riseup-vpn
```

---

## Network Layer Protocols Explained

### IPSec (Internet Protocol Security)
- **Layer**: Network Layer (Layer 3)
- **Components**:
  - **AH (Authentication Header)**: Provides integrity and authentication
  - **ESP (Encapsulating Security Payload)**: Provides encryption
  
```bash
# Check IPSec status
sudo ipsec status

# Install strongSwan for IPSec
sudo apt install strongswan -y
```

### WireGuard (Modern Network Layer VPN)
```bash
# Install WireGuard
sudo apt install wireguard -y

# Example configuration
sudo nano /etc/wireguard/wg0.conf

# Connect
sudo wg-quick up wg0
```

---

## Verification & Testing

### Check Your IP
```bash
# Before VPN
curl ifconfig.me

# After VPN (should show different IP)
curl ifconfig.me
```

### Check DNS Leaks
```bash
# Check DNS servers being used
nmcli dev show | grep DNS

# Online test
curl https://www.dnsleaktest.com/
```

### Verify VPN Connection
```bash
# Check active connections
ip a | grep tun

# Check routing table
ip route

# Monitor OpenVPN
sudo journalctl -u openvpn -f
```

### Kill Switch (Prevent IP Leaks)
```bash
# Create simple iptables kill switch
sudo iptables -A OUTPUT -o tun0 -j ACCEPT
sudo iptables -A OUTPUT -d <VPN_SERVER_IP> -j ACCEPT
sudo iptables -A OUTPUT -j DROP
```

---

## Pentesting Considerations

### Best Practices
1. **Chain VPNs**: Use multiple VPNs for enhanced anonymity
2. **Use Tor + VPN**: Extra layer (VPN → Tor or Tor → VPN)
3. **Check for leaks**: DNS, WebRTC, IPv6 leaks
4. **Rotating IPs**: Switch servers frequently
5. **Use kill switch**: Prevent accidental exposure

### Testing VPN Stability
```bash
# Continuous ping to check stability
ping -c 100 8.8.8.8

# MTU testing
ping -M do -s 1472 google.com
```

### Pentesting Tools with VPN
```bash
# All traffic through VPN
nmap -sV <target>
nikto -h <target>
sqlmap -u <url>

# ProxyChains for additional anonymity
proxychains nmap <target>
```

---

## Troubleshooting

### Common Issues

**Problem**: VPN won't connect
```bash
# Check logs
sudo journalctl -xe | grep -i vpn

# Restart NetworkManager
sudo systemctl restart NetworkManager
```

**Problem**: DNS not resolving
```bash
# Edit resolv.conf
sudo nano /etc/resolv.conf

# Add OpenDNS
nameserver 208.67.222.222
nameserver 208.67.220.220
```

**Problem**: Slow speeds
- Try UDP instead of TCP protocols
- Switch to closer servers
- Check MTU settings

---

## Quick Reference Commands

```bash
# Connect OpenVPN
sudo openvpn --config <file>.ovpn

# Disconnect (Ctrl+C or)
sudo killall openvpn

# List active VPN connections
nmcli connection show --active

# Connect via NetworkManager
nmcli connection up <vpn-name>

# Disconnect via NetworkManager
nmcli connection down <vpn-name>

# Check public IP
curl ifconfig.me

# Full leak test
curl -s https://ipleak.net/json/
```

---

## Security Warning ⚠️

**Remember:**
- Free VPNs have limitations
- Not all VPNs allow pentesting
- Always read Terms of Service
- Get proper authorization before testing
- Consider paid VPN for serious work
- Free VPNs may log data

---

## Summary

| Feature | ProtonVPN | VPNBook | Hide.me |
|---------|-----------|---------|---------|
| **Data Limit** | Unlimited | Unlimited | 10GB/month |
| **Speed** | Medium | Good | Good |
| **Servers** | 3 free | 6 servers | Limited |
| **Logs** | No logs | Minimal | No logs |
| **Setup Difficulty** | Easy | Easy | Easy |
| **Best For** | Daily use | Testing | Quick tasks |

---

**Happy & Ethical Pentesting! 🔒🐧**
```

---

## How to Save This as a Markdown File

### Option 1: Using Terminal
```bash
# Create the file
nano vpn-setup-guide.md

# Paste the content above
# Save: Ctrl + O, Enter
# Exit: Ctrl + X
```

### Option 2: Using Echo Command
```bash
cat > vpn-setup-guide.md << 'EOF'
# Paste the entire markdown content here
EOF
```

### Option 3: Direct Download
Save the above content to a file named `vpn-setup-guide.md` or `README.md` and view it with:

```bash
# View in terminal
cat vpn-setup-guide.md

# View with formatting (if installed)
mdless vpn-setup-guide.md

# Convert to PDF (if pandoc installed)
pandoc vpn-setup-guide.md -o vpn-setup-guide.pdf

# Open in default markdown viewer
xdg-open vpn-setup-guide.md
```
