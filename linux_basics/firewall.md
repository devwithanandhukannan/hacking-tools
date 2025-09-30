# Kali Linux IPTables Beginner Tutorial

## 1. Introduction
IPTables is a **firewall utility** in Linux that controls network traffic.  
It decides which packets are **allowed**, **blocked**, or **logged**.  

### Key Concepts
- **Packet**: A small piece of data sent over the network.
- **Chain**: A set of rules that a packet passes through.
- **Rule**: A condition to allow or block packets.

The main chains:
- **INPUT** → Incoming traffic to your computer
- **OUTPUT** → Outgoing traffic from your computer
- **FORWARD** → Traffic passing through your machine to another device

---

## 2. Start Kali and Login
1. Boot Kali Linux.
2. Login as **root**.
3. Check network interfaces:
```bash
ifconfig
````

---

## 3. Basic IPTables Commands

| Command                        | Description                      |
| ------------------------------ | -------------------------------- |
| `iptables -L`                  | List all rules                   |
| `iptables -A <CHAIN>`          | Add a rule to a chain            |
| `iptables -D <CHAIN>`          | Delete a rule from a chain       |
| `-p <protocol>`                | Specify protocol: tcp, udp, icmp |
| `--dport <port>`               | Destination port                 |
| `-j <action>`                  | Action: DROP, ACCEPT, LOG        |
| `-m multiport --dport <ports>` | Match multiple ports             |

**Example:**

```bash
iptables -A INPUT -p tcp --dport 80 -j DROP
```

---

## 4. INPUT Chain

* Controls **incoming traffic** to your computer.
* Example: Block HTTP requests from another computer:

```bash
iptables -A INPUT -p tcp --dport 80 -j DROP
```

* Allows you to **protect your services** (web server, SSH, etc.)

**Analogy:** INPUT = Visitors trying to enter your house.

**Check rules**

```bash
iptables -L -v -n
```

**Delete a rule**

```bash
iptables -D INPUT -p tcp --dport 80 -j DROP
```

---

## 5. OUTPUT Chain

* Controls **outgoing traffic** from your computer.
* Example: Block your computer from visiting websites on port 80:

```bash
iptables -A OUTPUT -p tcp --dport 80 -j DROP
```

* Protects your computer from **sending data to the internet**.

**Analogy:** OUTPUT = You sending letters or going out of your house.

**Delete a rule**

```bash
iptables -D OUTPUT -p tcp --dport 80 -j DROP
```

---

## 6. Blocking Traffic Examples

### 6.1 Block HTTP Traffic

```bash
iptables -A OUTPUT -p tcp --dport 80 -j DROP
iptables -D OUTPUT -p tcp --dport 80 -j DROP
```

### 6.2 Block Specific IP

```bash
iptables -A OUTPUT -p tcp -s 192.168.0.106 -d 65.61.137.117 --dport 80 -j DROP
iptables -D OUTPUT -p tcp -s 192.168.0.106 -d 65.61.137.117 --dport 80 -j DROP
```

### 6.3 Block Incoming HTTP from Windows

```bash
service apache2 start
iptables -A INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 --dport 80 -j DROP
iptables -D INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 --dport 80 -j DROP
```

### 6.4 Block SSH from Windows

```bash
service ssh start
# From Windows PowerShell: ssh root@192.168.0.106
iptables -A INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 --dport 22 -j DROP
iptables -D INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 --dport 22 -j DROP
```

### 6.5 Block Multiple Ports

```bash
iptables -A INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 -m multiport --dport 80,22 -j DROP
iptables -D INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 -m multiport --dport 80,22 -j DROP
```

### 6.6 Block Ping (ICMP)

```bash
iptables -A INPUT -p icmp -s 192.168.0.149 -d 192.168.0.106 -j DROP
iptables -D INPUT -p icmp -s 192.168.0.149 -d 192.168.0.106 -j DROP
```

Alternative (echo-request):

```bash
iptables -D INPUT -p icmp --icmp-type echo-request -s 192.168.1.2 -j DROP
```

### 6.7 Block Specific Website

```bash
iptables -A OUTPUT -p tcp -s 192.168.0.106 -d 172.67.189.184 --dport 443 -j DROP
iptables -A OUTPUT -p tcp -s 192.168.0.106 -d 104.21.65.93 --dport 443 -j DROP

ip6tables -A OUTPUT -p tcp -d 2606:4700:3032::6815:415d --dport 443 -j DROP
ip6tables -A OUTPUT -p tcp -d 2606:4700:3033::ac43:bdb8 --dport 443 -j DROP
```

---

## 7. Make IPTables Rules Persistent

**Install persistence package**

```bash
apt update
apt install iptables-persistent
```

**Save rules**

```bash
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6
```

**Enable auto-load on boot**

```bash
update-rc.d netfilter-persistent enable
```

**Edit rules manually**

```bash
mousepad /etc/iptables/rules.v4
```

---

## 8. Quick Tips

* Always test rules before deployment.
* Use `iptables -L -v -n` to see which rules are active and how many packets matched.
* INPUT = incoming traffic, OUTPUT = outgoing traffic.
* Learn multiport and ICMP filtering for advanced scenarios.

---

## 9. Summary

* **INPUT** → Controls incoming traffic.
* **OUTPUT** → Controls outgoing traffic.
* **FORWARD** → Controls traffic passing through your machine.
* IPTables is your **first step to securing Linux** with a firewall.

```


