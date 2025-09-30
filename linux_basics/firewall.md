
# Kali Linux IPTables Tutorial

## 1. Start Kali and Login
1. Boot Kali Linux.
2. Login as **root**.
3. Check network interfaces:
```bash
ifconfig
````

---

## 2. Basic IPTables Commands

* **List rules**

```bash
iptables -L
```

* **Add a rule**

```bash
iptables -A <CHAIN> -p <PROTOCOL> --dport <PORT> -j <ACTION>
```

* **Delete a rule**

```bash
iptables -D <CHAIN> -p <PROTOCOL> --dport <PORT> -j <ACTION>
```

**Key Flags:**

| Flag                           | Meaning                          |
| ------------------------------ | -------------------------------- |
| `-L`                           | List rules                       |
| `-A`                           | Append (add) a rule              |
| `-D`                           | Delete a rule                    |
| `-p`                           | Protocol (`tcp`, `udp`, `icmp`)  |
| `--dport`                      | Destination port                 |
| `-j`                           | Action (`DROP`, `ACCEPT`, `LOG`) |
| `-m multiport --dport <ports>` | Match multiple ports             |

---

## 3. Blocking Outgoing Traffic

**Block HTTP traffic (port 80)**

```bash
iptables -A OUTPUT -p tcp --dport 80 -j DROP
iptables -L
iptables -D OUTPUT -p tcp --dport 80 -j DROP
```

**Block traffic to specific IP**

```bash
iptables -A OUTPUT -p tcp -s 192.168.0.106 -d 65.61.137.117 --dport 80 -j DROP
iptables -L
iptables -D OUTPUT -p tcp -s 192.168.0.106 -d 65.61.137.117 --dport 80 -j DROP
```

**Verify** access using:

* [testfire.net](http://testfire.net)
* [testphp.vulnweb.com](http://testphp.vulnweb.com)

---

## 4. Blocking Incoming Traffic from Windows to Kali

### 4.1 Block HTTP from Windows

```bash
service apache2 start
iptables -A INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 --dport 80 -j DROP
iptables -D INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 --dport 80 -j DROP
```

### 4.2 Block SSH from Windows

```bash
service ssh start
# From Windows PowerShell: ssh root@192.168.0.106
iptables -A INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 --dport 22 -j DROP
iptables -D INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 --dport 22 -j DROP
```

### 4.3 Block Multiple Ports

```bash
iptables -A INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 -m multiport --dport 80,22 -j DROP
iptables -D INPUT -p tcp -s 192.168.0.149 -d 192.168.0.106 -m multiport --dport 80,22 -j DROP
```

### 4.4 Block Ping (ICMP)

```bash
iptables -A INPUT -p icmp -s 192.168.0.149 -d 192.168.0.106 -j DROP
iptables -D INPUT -p icmp -s 192.168.0.149 -d 192.168.0.106 -j DROP
```

**Alternative (echo-request)**

```bash
iptables -D INPUT -p icmp --icmp-type echo-request -s 192.168.1.2 -j DROP
```

---

## 5. Block Specific Websites from Kali

**Example: Hackerschool website**

```bash
iptables -A OUTPUT -p tcp -s 192.168.0.106 -d 172.67.189.184 --dport 443 -j DROP
iptables -A OUTPUT -p tcp -s 192.168.0.106 -d 104.21.65.93 --dport 443 -j DROP

ip6tables -A OUTPUT -p tcp -d 2606:4700:3032::6815:415d --dport 443 -j DROP
ip6tables -A OUTPUT -p tcp -d 2606:4700:3033::ac43:bdb8 --dport 443 -j DROP
```

---

## 6. Make IPTables Rules Persistent

**Install persistence package**

```bash
apt update
apt install iptables-persistent
```

**Save rules**

```bash
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6

cat /etc/iptables/rules.v4
cat /etc/iptables/rules.v6
```

**Enable auto-load on boot**

```bash
update-rc.d netfilter-persistent enable
```

* `update-rc.d`: manage services
* `netfilter-persistent`: load iptables rules on boot

**Edit rules manually**

```bash
mousepad /etc/iptables/rules.v4
```

---

## ✅ Tips

* Use `iptables -L -v -n` to verify packet counts and rule effectiveness.
* Use `service <name> start` to start services before applying rules.
* Always test rules with websites or IPs before final deployment.

```

---

I can also create a **version with diagrams showing INPUT/OUTPUT/FORWARD chains visually**, which makes this much easier to understand for beginners.  

Do you want me to do that next?
```
