Here’s a clean, structured note version of what you wrote — easier to follow if you’re using it as a local install guide or checklist:

---

## 🧭 QRadar Local Installation & Configuration Notes

### **1. VM & Network Setup**

* Set **network adapter** to **Host-Only Adapter**.
* During setup, create a **password** that includes **8+ characters**, **a symbol**, **a capital letter**, and **a number**.
* Login as **root**.

---

### **2. System Updates & Basic Config**

```bash
yum update -y
```

After update completes:

```bash
hostnamectl set-hostname soc.qradar
timedatectl set-timezone Asia/Kolkata
ip a
ip r
```

If IP can’t be changed directly, use `nmtui`.

---

### **3. Set Static IP**

Run:

```bash
nmtui
```

Then:

* Edit → Wired connection
* **Profile name:** same as device name (e.g. `enp0s17`)
* **IPv4 Address:** `192.168.10.150`
* **Gateway:** `192.168.10.100`
* **DNS:** `192.168.10.100`
* **IPv6:** Ignore or disable

Save and exit.

Disable IPv6 permanently:

```bash
echo "net.ipv6.conf.all.disable_ipv6=1" >> /etc/sysctl.conf
```

---

### **4. OS Version Change**

Edit version file:

```bash
vi /etc/redhat-release
```

* Press `i` → change version to:

  ```
  Red Hat Enterprise Linux Server release 7.5.1804
  ```
* Press `Esc` → type `:wq!` → hit Enter.

---

### **5. Confirm Host & Time**

```bash
hostname
timedatectl
date
```

---

### **6. Access Info (Reference)**

> ⚠️ Use only for lab/demo — not production.

```
URL: https://hs733.001193.xyz/
Username: test@gmail.com
Password: Pa$$w0rd
```

---

### **7. Final Steps**

* Save all network and version changes.
* Reboot the system:

  ```bash
  reboot
  ```
* Or power off:

  ```bash
  poweroff
  ```

---

### **8. Notes**

* Ensure the **default gateway** and **DNS** use the same IP.
* When assigning IP manually, **only modify the last octet** — keep the first three the same.
* Disable IPv6 both in `nmtui` and via terminal.
* Once the system boots cleanly, proceed with QRadar installation and rule/offense configuration.

