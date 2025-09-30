# How to install VPN in Kali

> Complete notes: application-layer vs network-layer VPNs, step-by-step installation for ProtonVPN and VPNBook (OpenVPN), quick notes on IPSec, Tor, and proxychains on Kali Linux.

---

## Overview: Application layer vs Network layer VPN

* **Application-layer VPN** (also called proxy/VPN per-application): encrypts traffic for a specific application only (for example, a browser or an SSH client). You configure the app to use a local proxy or a VPN client that supports selective routing. This is useful when you only want certain apps to go through the VPN.

* **Network-layer VPN** (system-wide): routes traffic at the OS/network stack level so every application and service uses the VPN tunnel (unless split-tunnelling rules are used). Common implementations: OpenVPN, WireGuard, IPSec. This is ideal for full-machine anonymity or connecting to remote networks.

---

## Prerequisites (Kali Linux)

* A working internet connection.
* Root or sudo privileges.
* `apt` package manager (Kali base).
* Recommended: update system before starting:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 1) ProtonVPN (general steps)

**Install ProtonVPN client (official) or use OpenVPN configs from Proton to import**

### A. Using ProtonVPN Linux CLI (recommended for convenience)

1. Install dependencies and the ProtonVPN repository (follow Proton's latest instructions). Example common commands:

```bash
sudo apt install -y gnupg2 wget
# add Proton's repo (example; always check ProtonVPN docs for latest repo and commands)
wget -q -O - https://repo.protonvpn.com/debian/public_key.asc | sudo apt-key add -
sudo add-apt-repository 'deb https://repo.protonvpn.com/debian stable main'
sudo apt update
sudo apt install -y protonvpn
```

2. Initialize and login:

```bash
sudo protonvpn-cli login <your-email>
sudo protonvpn-cli connect # or use 'protonvpn-cli connect --sc' for secure core etc.
```

> **Note:** The ProtonVPN GUI or CLI may show a **Download** or **Configuration** section on the web dashboard where you can export OpenVPN configuration files if you prefer `openvpn`.

### B. Using ProtonVPN OpenVPN configs (manual import)

1. On the ProtonVPN web dashboard click **Download** (left menu) and generate OpenVPN configuration files for the server you want.
2. Save the generated `.ovpn` file.
3. In Kali's topbar: **Network** (or Ethernet) → **VPN Connections** → **Configure VPN** → **Add** → **Import a saved VPN configuration** and select the `.ovpn` file.
4. Or use the terminal:

```bash
sudo apt install -y openvpn
sudo openvpn --config /path/to/protonvpn-server.ovpn
```

5. GUI import will create a NetworkManager connection. Use the topbar network menu to connect/disconnect.

---

## 2) VPNBook (OpenVPN)

VPNBook provides free OpenVPN configs (username/password-based).

### Steps:

1. Download the OpenVPN ZIP from VPNBook website and unzip it:

```bash
wget https://www.vpnbook.com/free-openvpn-account/VPNBook.com-OpenVPN-Euro1.zip -O vpnbook-euro.zip
unzip vpnbook-euro.zip -d vpnbook-configs
```

2. Inside the extracted folder you will find `.ovpn` files, e.g. `vpnbjlok-ca196-tcp80.ovpn` (filename will vary).

3. Bring up the VPN using OpenVPN in terminal:

```bash
sudo apt install -y openvpn
sudo openvpn vpnbook-configs/vpnbjlok-ca196-tcp80.ovpn
```

4. OpenVPN will prompt for **username** and **password** — use the credentials listed on VPNBook's web page (they rotate/change frequently). Keep these credentials handy.

5. To run in the background use systemd or run `openvpn --daemon --config <file>`.

6. To use NetworkManager GUI: import the `.ovpn` file as described in ProtonVPN step.

---

## 3) IPSec (network layer)

* IPSec operates at the network layer (L3) and is commonly used for site-to-site VPNs and some client-to-server VPNs (e.g., strongSwan, libreswan).

### Quick strongSwan example (install):

```bash
sudo apt update
sudo apt install -y strongswan strongswan-plugin-eap-mschapv2
```

Configuration for IPSec (client) typically lives in `/etc/ipsec.conf` and secrets in `/etc/ipsec.secrets`. IPSec setups can be more involved (certificates, PSK, IKEv2 settings).

---

## 4) Tor and proxychains (anonymity layer, not a VPN)

Tor is a circuit-based anonymity network. It is not a VPN but can provide application-level anonymity when used with proxychains.

### Install Tor service

```bash
sudo apt update
sudo apt install -y tor
sudo systemctl enable --now tor    # starts tor and enables at boot
sudo systemctl start tor           # start
sudo systemctl stop tor            # stop
```

Tor's default SOCKS proxy listens on `127.0.0.1:9050`.

### Install proxychains4

```bash
sudo apt install -y proxychains4
```

### proxychains4: config and usage

* Configuration file: `/etc/proxychains4.conf` (or `~/.proxychains/proxychains.conf`).
* Example excerpt (tail) that ensures Tor is the default proxy:

```text
# proxy types: http, socks4, socks5, raw
# * raw: The traffic is simply forwarded to the proxy without additional framing

[ProxyList]
# add proxy here ...
# defaults set to "tor"
socks4 127.0.0.1 9050
# (auth types supported: "basic" - http, "user/pass" - socks)
```

* To run a command through proxychains (prefix the command):

```bash
proxychains4 curl http://ifconfig.me
# or
proxychains4 nmap -sT <target>
```

> **Note:** Not all traffic/ports/protocols work reliably through proxychains/Tor. DNS leaks and UDP are common caveats.

---

## What is a proxy and why use one?

* A **proxy** is an intermediary between your application and the destination server. Types: HTTP proxy, SOCKS4/SOCKS5.

* **Why use a proxy?**

  * Route traffic through another host (location masking).
  * Add an anonymity layer (when combined with Tor or other proxies).
  * Apply per-application routing (instead of full-machine VPN).

* Example: `socks4 127.0.0.1 9050` means: use SOCKS4 proxy on localhost port 9050 (Tor's default).

---

## Handy commands & tips

* Install OpenVPN and network-manager plugin for GUI support:

```bash
sudo apt install -y openvpn network-manager-openvpn network-manager-openvpn-gnome
sudo systemctl restart NetworkManager
```

* To import a `.ovpn` via GUI: Topbar → Network → VPN Connections → Configure VPN → Add → Import from file → select `.ovpn`.

* Run OpenVPN from terminal (better for logs):

```bash
sudo openvpn --config /path/to/file.ovpn
```

* If OpenVPN asks for username/password, provide as prompted or create a `credentials.txt` with two lines (username on first, password on second) and reference it in the `.ovpn` with:

```text
auth-user-pass credentials.txt
```

* To test IP after connecting:

```bash
curl ifconfig.me
# or
curl https://ipinfo.io/ip
```

* To see active network-manager VPN connections:

```bash
nmcli connection show --active
```

---

## Security and privacy notes

* Free VPNs (like VPNBook) may log or throttle traffic. Read their privacy policy.
* ProtonVPN has a no-logs policy (verify current policy on Proton's site).
* Tor provides strong anonymity but is not a drop-in replacement for a VPN — it has different threat models and performance characteristics.
* Always verify the authenticity of downloaded config files (signature or HTTPS), and avoid using untrusted VPN providers for sensitive activities.

---

## Quick troubleshooting

* **OpenVPN: TLS handshake error** → Check credentials, server status, and that ports are not blocked (TCP/UDP as required).
* **No internet after connect** → check routing table (`ip route`), DNS settings, and `resolv.conf`.
* **DNS leaks** → use DNS over VPN or configure `/etc/resolv.conf` via NetworkManager to use secure DNS.

---

## Further reading & next steps

* ProtonVPN docs (for CLI and OpenVPN exports).
* OpenVPN manual and `man openvpn`.
* strongSwan documentation for IPSec/IKEv2.
* Tor Project documentation and `proxychains4` usage notes.

---

