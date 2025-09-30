# IDS/IPS Quick Reference (Snort + Honeypot)

A concise, standalone `.md` note describing steps to set up Snort rules on Kali, start services, and a basic honeypot test using KFSensor (Windows). Update IP addresses to match your environment where noted.

---

## Environment & network setup

1. In your host virtualization software, set the VM network to **Bridge Adapter** (so the Kali VM gets an IP on the local network).
2. Start Kali Linux.
3. Verify the Kali IP address:

```bash
ifconfig
# or
ip addr show
```

Make a note of your Kali IP (example below uses `192.168.0.106` — replace with your actual IP).

---

## Install Snort

```bash
sudo apt update
sudo apt install -y snort
```

During installation you may be prompted for network interface selection — choose the interface that matches your bridge adapter (for example `eth0` or `ens33`).

---

## Create local rules file

Use a simple text editor (e.g., `mousepad`, `nano`, or `vim`) to create a local rules file. Example uses `/root/local.rules`.

```bash
mousepad /root/local.rules
```

Add the following rules. Replace IP addresses with your environment's addresses as needed.

```snort
# 1) Alert when Kali user accesses any unsecured HTTP site (replace Kali_IP)
alert tcp 192.168.0.106 any -> any 80 (msg:"Accessing unsecured website"; sid:1000001; rev:1;)

# 2) Alert when Kali user accesses testfire.net (example IP 65.61.137.117)
alert tcp 192.168.0.106 any -> 65.61.137.117 80 (msg:"testfire website"; sid:1000002; rev:1;)

# 3) Alert when Windows user accesses Apache on Kali (replace Windows_IP)
alert tcp 192.168.0.149 any -> 192.168.0.106 80 (msg:"apache web server"; sid:1000003; rev:1;)

# 4) Alert when Windows user attempts SSH to Kali
alert tcp 192.168.0.149 any -> 192.168.0.106 22 (msg:"windows ssh access"; sid:1000004; rev:1;)

# 5) Alert when Windows system pings the Kali machine (ICMP)
alert icmp 192.168.0.149 any -> 192.168.0.106 any (msg:"ping request from windows"; sid:1000005; rev:1;)

# 6) Alert when Windows user accesses FTP on Kali
alert tcp 192.168.0.149 any -> 192.168.0.106 21 (msg:"ftp server access"; sid:1000006; rev:1;)
```

Save the file (`Ctrl+S`) and close the editor.

> Note: `sid` (rule ID) must be unique and typically custom rules use IDs above `1000000` to avoid conflicts with official rules. `rev` indicates revision.

---

## Run Snort with the rules

Example command (adjust paths and interface):

```bash
sudo snort -c /etc/snort/snort.lua -R /root/local.rules -i eth0 -A alert_fast -s 65535 -k none
```

Flags explained:

* `-c` = path to the Snort configuration file (often `/etc/snort/snort.lua` for Snort 3.x or `/etc/snort/snort.conf` for Snort 2.x).
* `-R` = read additional rules from the specified file (legacy; some Snort versions accept `-S` or include rules via config — confirm for your Snort version).
* `-i` = network interface to listen on (e.g., `eth0`).
* `-A` = alert mode (e.g., `alert_fast` prints alerts to console quickly).
* `-s` = snaplen (maximum packet capture size) — `65535` to capture full packets.
* `-k none` = disable checksum checks (useful when capturing on some virtual interfaces).

If you encounter errors, check Snort logs and ensure the correct config file and rule syntax for your Snort version (Snort 2.x vs 3.x differ).

---

## Example: Start Apache, SSH, vsftpd on Kali (to test rules)

```bash
# Apache
sudo service apache2 start

# SSH
sudo service ssh start

# FTP (vsftpd)
sudo apt update
sudo apt install -y vsftpd
sudo service vsftpd start
```

Trigger tests from a Windows host (or another VM) using browser visits, `ftp`, `ssh`, or `ping` to the Kali IP.

---

## Host-based vs Network-based IDS/IPS

* **Host-based IDS/IPS (HIDS/HIPS):** Monitors activity on an individual host — file integrity, logs, local process behavior. Examples: OSSEC, Wazuh, Tripwire.
* **Network-based IDS/IPS (NIDS/NIPS):** Monitors network traffic on one or more links; Snort and Suricata are common NIDS solutions.

Choose based on requirements: HIDS offers visibility into host internals; NIDS provides broad network traffic monitoring.

---

## Honeypot: KFSensor (Windows)

KFSensor is a Windows-based honeypot. Basic steps (KFSensor runs on Windows):

1. On the Windows host, install **Npcap** (packet capture driver, similar to WinPcap) — choose the "WinPcap compatible" option if prompted.
2. Install **KFSensor** (follow vendor instructions and license terms).
3. If test purposes require, temporarily disable the Windows firewall (or allow KFSensor ports) so the honeypot can receive connections.
4. Start KFSensor and configure the services/ports you want to emulate (HTTP, FTP, SSH, etc.).

---

## Test by scanning from Kali

From Kali, run an aggressive nmap scan against the Windows honeypot IP (example `192.168.0.111`):

```bash
nmap -A 192.168.0.111
```

Observe KFSensor logs and Snort alerts on the Kali host (if the traffic passes through the Kali-monitored network link).

---

## Notes & best practices

* Ensure rule syntax matches the Snort version installed (Snort 2.x rules differ slightly from Snort 3.x configuration mechanisms). For Snort 3, rule inclusion is typically done via the main `.conf`/`.lua` configuration rather than `-R`.
* Always test rules in a controlled lab before deploying on production networks.
* Use unique SIDs for local rules (commonly `1_000_000` and above) to avoid collisions with community or official rulesets.
* Keep services and OS patched; honeypots can be targeted and exploited if misconfigured.

---


