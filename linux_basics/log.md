## Comprehensive Notes: Logs for SOC Analysts & System Administrators  
*(From Basic to Advanced)*  

---

### **I. What is a Log? (Basic)**  

**Definition:**  
A **log** is a systematic record of events, activities, or transactions that occur within a system, application, or network. Think of it as a digital diary that documents **what happened, when, where, and by whom**.  

**Purpose:**  
1.  **Auditing:** Track user actions for compliance (e.g., GDPR, HIPAA).  
2.  **Troubleshooting:** Diagnose system/application issues.  
3.  **Security Monitoring:** Detect attacks, breaches, or suspicious activity.  
4.  **Performance Analysis:** Identify bottlenecks or resource exhaustion.  
5.  **Forensics:** Reconstruct events during an incident investigation.  

**Key Principle:**  
> **Not all logs are equally important!**  
> Systems generate **thousands** of logs daily. Many are routine (e.g., "Driver loaded") and can be ignored. Focus on **anomalies**, **errors**, and **security-related events**.  

---

### **II. Typical Log Format (Basic to Intermediate)**  

Every log entry generally contains these **key fields**:  

| **Field**               | **Description**                                                                 | **Example**                                  |
| :---------------------- | :------------------------------------------------------------------------------ | :------------------------------------------- |
| **1. Date & Time**      | When the event occurred (critical for timeline analysis). **Always in UTC** for consistency. | `2023-10-05 14:30:45 UTC`                    |
| **2. Event Type**       | Nature of the event.                                                            | `Login`, `File Access`, `Port Scan`, `Error`  |
| **3. Severity Level**   | **How serious** the event is.                                                   | `Info` (0), `Warning` (1), `Error` (2), `Critical` (3) |
| **4. Origin**           | **Where** the event happened (IP, hostname, application).                       | `Server: WEB01`, `App: Chrome`, `IP: 192.168.1.10` |
| **5. User**             | **Who** caused the event (username or system account).                          | `user123`, `SYSTEM`, `root`                   |
| **6. Action/Service**   | **What** service or protocol was involved.                                      | `Service: FTP`, `Protocol: TCP`, `Action: File Download` |
| **7. Description**      | **Detailed explanation** of the event.                                          | `User 'john' successfully logged in via SSH`  |
| **8. Event ID**         | **Unique identifier** for the event type (OS-specific). **CRITICAL for filtering!** | `Event ID: 4624` (Windows), `Event ID: 1234` (Linux) |

**Important Note:**  
> ✅ **Logs should NEVER contain credentials** (passwords, API keys). Only **communication metadata** (e.g., "User *tried* to log in") is logged.  

---

### **III. Types of Logging: Local vs. Distributed (Intermediate)**  

#### **A. Local Logging**  
- **Definition:** Logs are stored **on the same system** where the event occurred.  
- **Pros:** Simple to set up, no network dependency.  
- **Cons:**  
  - Single point of failure (if the system crashes, logs are lost).  
  - Hard to correlate events across multiple systems.  
- **Examples:**  
  - Windows: `Event Viewer` logs (local).  
  - Linux: `/var/log/` files (e.g., `auth.log`, `syslog`).  

#### **B. Distributed Logging**  
- **Definition:** Logs are **sent to a central server** (e.g., SIEM like Splunk, ELK Stack, Graylog).  
- **Pros:**  
  - Centralized view of **all systems** in the network.  
  - Easier correlation (e.g., "Attack started on Server A, spread to Server B").  
  - Backup & retention policies enforced centrally.  
- **Cons:**  
  - Requires network bandwidth & infrastructure.  
  - More complex setup.  
- **How it works:**  
  - Agents on each system forward logs to a central log server.  
  - Example: `rsyslog` (Linux) sends logs to a central server.  

> **🚨 SOC & Advanced Admins MUST use Distributed Logging!**  
> You can’t investigate a network breach with logs scattered across 100 servers.  

---

### **IV. Windows Logging (System & Security Focus)**  

#### **A. Accessing Logs**  
1. Press `Win + S` → type **`eventvwr`** → open **Event Viewer**.  
2. Structure:  
   ```  
   Event Viewer  
   ├── Applications and Services Logs  
   ├── Custom Views  
   ├── Windows Logs  
   │   ├── **Application**  
   │   ├── **Security** ⚠️ (MOST IMPORTANT for SOC)  
   │   ├── **System**  
   │   ├── **Setup**  
   │   └── **Forwarded Events** (Distributed logs)  
   ```  

#### **B. Key Windows Log Categories**  

##### **1. Security Log (MOST CRITICAL for SOC)**  
*Records **all security-related events**. This is the **#1 source** for detecting attacks.*  

| **Event ID** | **Description**                                                                 | **When to Investigate**                                                                 |
| :----------- | :------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------- |
| **4624**     | **Successful log-on** (user logged in).                                         | ✅ **Good**: Audit user logins. <br>🔍 **Bad**: Brute-force attacks (many 4624 entries with failed 4625 before). |
| **4625**     | **Failed log-on attempt**.                                                      | 🔴 **CRITICAL**: Brute-force attacks, mistyped passwords, account lockouts. **First sign of an attack!** |
| **4634**     | **User log-off** (session ended).                                                | ✅ Track user activity duration.                                                     |
| **4647**     | **User logged off** (explicit).                                                 | ✅ Correlates with 4624/4634.                                                        |
| **4648**     | **Logon using credentials (e.g., runas, service account)**.                     | 🔍 Suspicious if unusual service accounts are used.                                  |
| **4672**     | **Special privileges assigned to a user**.                                      | 🔴 **ALERT**: Someone got admin rights (e.g., attacker elevating privileges).         |
| **4720**     | **User account created**.                                                       | 🔍 New accounts created? Could be attacker creating a backdoor account.               |
| **4722**     | **User account enabled** (after being disabled).                                | 🔍 Disabled accounts suddenly re-enabled? Suspicious.                                 |
| **4723**     | **User password changed**.                                                      | ✅ Normal admin activity OR attacker changing passwords.                             |
| **4725**     | **User account disabled**.                                                      | ✅ Could be admin cleaning up OR attacker disabling competitors.                     |
| **4726**     | **User account deleted**.                                                       | 🔍 Sudden deletion of accounts? Investigate!                                         |
| **4740**     | **Account locked out**.                                                         | 🔴 **BRUTE-FORCE DETECTION**: Multiple 4625 → 4740. **MOST COMMON ATTACK SIGNATURE!** |
| **4781**     | **User account renamed**.                                                       | 🔍 Unusual renaming? Possible attacker hiding tracks.                                |
| **4688**     | **New process created** (e.g., `cmd.exe`, `powershell.exe`).                     | 🔍 **Malware detection**: Unknown process started? (e.g., `C:\Temp\malware.exe`).     |
| **1102**     | **Security log cleared**.                                                       | 🔴 **ALERT**: Attacker deleted evidence! **Immediate investigation required.**        |

> **SOC Tip:** Create an alert for **5+ 4625 events in 5 minutes** → likely brute-force attack.  

##### **2. System Log**  
*Records **system-level events** (hardware, OS, services).*  

| **Event ID** | **Description**                                                                 | **When to Investigate**                                                                 |
| :----------- | :------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------- |
| **6005**     | **System started** (boot).                                                      | ✅ Track system uptime/reboots.                                                       |
| **6006**     | **System shut down**.                                                           | ✅ Investigate unexpected shutdowns.                                                  |
| **6008**     | **Unexpected shutdown** (crash/power loss).                                     | 🔴 **CRITICAL**: System crashed. Check hardware, drivers, or kernel logs.              |
| **6013**     | **System uptime** (e.g., "System has been running for 2 days").                  | ✅ Monitor server stability.                                                          |
| **2004**     | **Resource exhaustion** (e.g., low memory, disk space).                         | 🔴 **URGENT**: System running out of resources → performance issues or crash.          |
| **1000**     | **Application crash** (blue screen, app failure).                               | 🔍 Troubleshoot application errors.                                                   |
| **11707**    | **Application failed to install**.                                              | ✅ User or admin tried installing software.                                           |

##### **3. Application Log**  
*Records events from **installed applications** (e.g., Office, SQL Server).*  

| **Event ID** | **Description**                                                                 | **When to Investigate**                                                                 |
| :----------- | :------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------- |
| **1000**     | **Generic application crash**.                                                  | 🔍 Most common crash log. Check `Exception` details.                                   |
| **10002**    | **Application hung** (stopped responding).                                      | ✅ User-reported freezes.                                                             |
| **2003**     | **Application terminated unexpectedly**.                                        | 🔍 Investigate why the app exited.                                                    |

##### **4. Windows Defender Logs (FOR MALWARE)**  
*Location: `Event Viewer` → `Applications and Services Logs` → `Microsoft` → `Windows` → `Windows Defender` → `Operational`*  

| **Event ID** | **Description**                                                                 | **When to Investigate**                                                                 |
| :----------- | :------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------- |
| **1116**     | **Malware detected** (and removed/queried).                                     | 🔴 **MALWARE ALERT**: Details include **malware name**, **file path**, **severity**.   |
| **1117**     | **Malware actually removed**.                                                   | ✅ Confirm Defender took action.                                                      |
| **11123**    | **Real-time protection disabled**.                                              | 🔴 **ALERT**: Attacker disabled Defender! **Immediate response needed.**              |
| **2001**     | **Scan started** (manual or scheduled).                                         | ✅ Normal operation.                                                                  |
| **2003**     | **Scan completed**.                                                             | ✅ Check scan results.                                                                |
| **50001**    | **Windows Defender service started**.                                           | ✅ Normal.                                                                            |

> **SOC Action:**  
> - **If you see 1116**:  
>   1. Check **file path** (e.g., `C:\Users\john\Downloads\malware.exe`).  
>   2. Correlate with **4688** (process creation) → Who launched it?  
>   3. Check **user account** → Is it a legitimate user or service account?  

---

### **V. Linux Logging (System & Auth Focus)**  

#### **A. Setting Up Logging**  
1. **Update Repositories:**  
   ```bash
   sudo apt update
   ```
2. **Install `rsyslog`** (Linux logging daemon):  
   ```bash
   sudo apt install rsyslog
   ```
3. **Restart Service:**  
   ```bash
   sudo systemctl restart rsyslog
   ```
4. **Logs are stored in `/var/log/`**  

#### **B. Key Linux Log Files**  

##### **1. `/var/log/auth.log` (OR `/var/log/secure` on RHEL/CentOS)**  
*Records **authentication events** (logins, SSH, sudo).* **MOST IMPORTANT FOR SOC!**  

| **Event**                | **Example Line**                                                                 | **When to Investigate**                                                                 |
| :----------------------- | :------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------- |
| **SSH Login (Success)**  | `Oct 5 14:30:45 server sshd[1234]: Accepted password for john from 192.168.1.10 port 55432 ssh2` | ✅ Normal user login.                                                                |
| **SSH Login (Failure)**  | `Oct 5 14:31:12 server sshd[1235]: Failed password for root from 192.168.1.10 port 55433 ssh2` | 🔴 **BRUTE-FORCE ATTACK**: Multiple failures from same IP. **Check `last -f /var/log/btmp`** |
| **Sudo Command**         | `Oct 5 14:32:00 server sudo: john : TTY=pts/0 ; PWD=/home/john ; USER=root ; COMMAND=/bin/nm` | 🔍 User ran privileged command.                                                      |
| **Account Locked**       | `Oct 5 14:33:15 server PAM unable to open password file`                         | 🔴 Account locked out (often due to brute-force).                                     |

**Commands to Monitor `auth.log`:**  
```bash
# Live view (press Ctrl+C to stop)
tail -f /var/log/auth.log

# Show failed SSH attempts
grep "Failed password" /var/log/auth.log

# Show successful logins
grep "Accepted password" /var/log/auth.log

# Check locked accounts
grep "PAM unable to open password file" /var/log/auth.log
```

##### **2. `/var/log/syslog`**  
*General system log (all events not specific to other files).*  

| **Event**                | **Example Line**                                                                 | **When to Investigate**                                                                 |
| :----------------------- | :------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------- |
| **System Startup**       | `Oct 5 14:00:00 server systemd[1]: Started System Logging Service.`               | ✅ Normal boot process.                                                               |
| **Service Failure**      | `Oct 5 14:15:22 server systemd[1]: failed to start Apache HTTP Server.`          | 🔴 Service crashed → check Apache logs (`/var/log/apache2/error.log`).                |
| **Hardware Issues**      | `Oct 5 14:20:10 server kernel: [12345.678901] sda: detected error`               | 🔴 **HARDWARE FAILURE** (disk, memory).                                               |

##### **3. `/var/log/boot.log`**  
*Records **services started/stopped during boot**.*  
- **Not critical for security**, but useful for **troubleshooting startup issues**.  
  ```bash
  cat /var/log/boot.log
  ```

##### **4. `/var/log/dpkg.log`**  
*Records **software installations/removals**.*  

| **Event**                | **Example Line**                                                                 | **When to Investigate**                                                                 |
| :----------------------- | :------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------- |
| **Package Installed**    | `2023-10-05 14:40:22 install apache2:amd64 2.4.38-3ubuntu1`                       | 🔍 Who installed Apache? **Correlate with `auth.log`** (who was logged in?).          |
| **Package Removed**      | `2023-10-05 14:45:10 remove nginx:all 1.18.0-0ubuntu1`                           | 🔍 Unusual removal of services? Possible attacker cleanup.                            |

##### **5. `/var/log/kern.log`**  
*Kernel-related events (e.g., driver errors, hardware failures).*  
- **Check this when system crashes/freezes.**  

##### **6. User Login History**  
```bash
# Show who is currently logged in
who

# Show full login history (with dates)
last -a

# Show failed login attempts (btmp = bad login log)
last -f /var/log/btmp
```

> **SOC Tip for Linux:**  
> - **Brute-force attack detection:**  
>   1. `grep "Failed password" /var/log/auth.log \| wc -l` → Count failures.  
>   2. `grep "Failed password" /var/log/auth.log \| cut -d" " -f11 \| sort \| uniq -c` → See IPs.  
>   3. If >5 failures from same IP in 1 minute → **Block IP** via firewall (`ufw deny from 192.168.1.10`).  

---

### **VI. Situations & When to Use Logs (Real-World Examples)**  

#### **A. Scenario 1: User Can’t Log In (Windows)**  
**Symptoms:** User reports "Invalid password" even though it’s correct.  
**Logs to Check:**  
1. **Security Log → Event ID 4625**  
   - Look for entries with the user’s name.  
   - **Check `Logon Type`** (e.g., `2` = Interactive, `3` = Network).  
   - **Check `Workstation Name`** → Where did the attempt come from?  
2. **If 4625 shows "account locked":**  
   - Check **Event ID 4740** → Account locked out.  
   - **Solution:** Unlock account (`net user john /active:yes`), then investigate why it locked.  

#### **B. Scenario 2: Server Crashed Unexpectedly (Windows)**  
**Symptoms:** Server is down, no response.  
**Logs to Check:**  
1. **System Log → Event ID 6008**  
   - **"Unexpected shutdown"** → Crash due to hardware/driver.  
2. **Check **Kernel-Memory** events** (Event ID 1001) → Blue Screen (BSOD) details.  
3. **Check **Application Log** for **1000** → Which app crashed?  

#### **C. Scenario 3: Suspected Malware Infection (Windows)**  
**Symptoms:** Slow performance, unusual network traffic.  
**Logs to Check:**  
1. **Windows Defender → Operational Log**  
   - **Event ID 1116**: Malware detected.  
     - **Details:** Malware name, file path, signature.  
     - **Example:** `MalwareName: Emotet, FilePath: C:\Temp\update.exe`  
2. **Security Log → Event ID 4688**  
   - **Filter for `C:\Temp\update.exe`** → Who launched it?  
   - **Check the `User` field** (e.g., `john` or `SYSTEM`).  
3. **Correlate with Network Logs** (Firewall/SIEM) → Where did `update.exe` connect?  

#### **D. Scenario 4: Brute-Force Attack on SSH (Linux)**  
**Symptoms:** Multiple failed login attempts in `auth.log`.  
**Logs to Check:**  
1. **`/var/log/auth.log` → "Failed password" entries**  
   ```bash
   grep "Failed password" /var/log/auth.log
   ```  
   - **Example Output:**  
     `Oct 5 15:01:23 server sshd[2345]: Failed password for root from 203.0.113.5 port 55678 ssh2`  
2. **Count attacks:**  
   ```bash
   grep "Failed password" /var/log/auth.log | wc -l
   ```  
3. **Identify attacking IPs:**  
   ```bash
   grep "Failed password" /var/log/auth.log | cut -d" " -f11 | sort | uniq -c
   ```  
   - **Output:**  
     `12 203.0.113.5`  
     `5 198.51.100.2`  
4. **Block the IP:**  
   ```bash
   sudo ufw deny from 203.0.113.5
   ```

#### **E. Scenario 5: Unauthorized Software Installation (Linux)**  
**Symptoms:** Unknown software (`malware.exe`) found on server.  
**Logs to Check:**  
1. **`/var/log/dpkg.log`**  
   - Search for the software name:  
     ```bash
     grep "malware" /var/log/dpkg.log
     ```  
   - **Output:**  
     `2023-10-05 16:20:15 install malware:all 1.0-unknown`  
2. **Check who was logged in at that time:**  
   ```bash
   last -a | grep "2023-10-05 16:20"
   ```  
   - **Output:**  
     `john    pts/0        192.168.1.10  Mon Oct  5 16:15   still logged in`  

---

### **VII. Advanced Topics (SOC & Senior Admins)**  

#### **A. Log Correlation**  
- **Definition:** Combining logs from **multiple sources** to uncover hidden attacks.  
- **Example:**  
  1. **Windows Defender** logs **malware detected** (Event 1116).  
  2. **Security Log** shows **process created** (Event 4688) for the malware file.  
  3. **Firewall Logs** show **outbound connection** to a suspicious IP.  
  → **Full attack chain revealed!**  

#### **B. SIEM (Security Information & Event Management)**  
- **Tools:** Splunk, IBM QRadar, Elastic Stack (ELK), Microsoft Sentinel.  
- **What it does:**  
  - **Collects logs** from all systems (Windows, Linux, firewalls, etc.).  
  - **Correlates events** using rules (e.g., "5 failed logins → alert").  
  - **Generates dashboards** (e.g., "Top 10 attacking IPs today").  
  - **Automated responses** (e.g., block IP if malware detected).  

#### **C. Log Retention & Rotation**  
- **Why:** Logs take up disk space!  
- **How:**  
  - **Log Rotation:** Use `logrotate` (Linux) or Windows Task Scheduler to **compress/delete old logs**.  
    - Example (`/etc/logrotate.d/rsyslog`):  
      ```conf
      /var/log/syslog {
          rotate 7
          daily
          missingok
          notifempty
          compress
          delaycompress
          postrotate
              /usr/bin/systemctl reload rsyslog >/dev/null 2>&1 || true
          endscript
      }
      ```  
  - **Retention Policy:**  
    - **Security Logs:** Keep **90+ days** (legal requirements).  
    - **System Logs:** Keep **7-30 days** (troubleshooting).  

#### **D. Log Security**  
- **Never store logs in plain text!**  
  - **Encrypt logs** at rest and in transit.  
  - **Prevent tampering:**  
    - Use **write-once filesystems** (e.g., `chattr +a` on Linux).  
    - **Digital Signatures** for logs (advanced).  

#### **E. Common Pitfalls to Avoid**  
| **Mistake**                          | **Solution**                                                                 |
| :----------------------------------- | :--------------------------------------------------------------------------- |
| **Not centralizing logs**            | Use a SIEM or central log server.                                            |
| **Ignoring severity levels**         | Prioritize **Errors** and **Critical** events first.                         |
| **Not filtering noise**              | Create **custom filters** (e.g., ignore "Driver loaded" events).              |
| **Storing logs on the same system**  | Store logs **on a separate secure server**.                                   |
| **Forgetting to check timestamps**   | **Always convert to UTC** for timeline analysis.                              |

---

### **Summary Cheat Sheet**  

| **Log Type**       | **Location**                     | **Key Events for SOC**                     | **Key Events for Admins**                  |
| :----------------- | :------------------------------- | :----------------------------------------- | :----------------------------------------- |
| **Windows Security** | `Event Viewer > Security`         | 4625 (Failed Login), 4740 (Lockout), 4688 (Process), 1102 (Log Cleared) | 4624 (Login), 4634 (Logout)                |
| **Windows System**  | `Event Viewer > System`           | 6008 (Crash), 2004 (Resource Exhaustion)    | 6005 (Boot), 6006 (Shutdown)               |
| **Windows Defender**| `Applications and Services > Microsoft > Windows > Windows Defender > Operational` | 1116 (Malware Detected), 11123 (Real-time disabled) | 2001 (Scan Start), 2003 (Scan Complete)    |
| **Linux auth.log**  | `/var/log/auth.log`               | Failed SSH logins, sudo commands            | Successful logins, account changes         |
| **Linux syslog**    | `/var/log/syslog`                 | Service failures, hardware errors          | General system health                      |
| **Linux dpkg.log**  | `/var/log/dpkg.log`               | Unauthorized software installs             | Software management                        |

> **Final Tip:**  
> **Always start with the most critical logs first!**  
> - **SOC:** Security Log (Windows) → `auth.log` (Linux) → Defender Logs.  
> - **System Admin:** System Log (Windows) → `syslog` (Linux) → Application Logs.  

📌 **Practice these steps daily – logs are your #1 tool for security and stability!** 