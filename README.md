# 🛡️ Backup-Server Trap Honeypot 🗄️

### I’m running a Rocky Linux honeypot that simulates a corporate backup server. My goal is to **observe attacker behavior**, study how malicious activity appears in logs and packet captures, and **learn how to produce actionable artifacts** for incident response (IR) and stakeholder reporting. The honeypot runs in an isolated environment to prevent unauthorized access or exfiltration, while providing realistic bait files and banners to attract potential threats. This setup allows me to **practice detection, analysis, and containment** in a safe, controlled environment.

![uqWrrH0](https://github.com/user-attachments/assets/18b30c27-448c-4111-8209-598f4dece0ef)

## 🎯 What I’m Learning
- I can see how attacker behavior appears in logs and PCAPs (scans, brute-force attempts, reconnaissance).  
- I can generate IR artifacts like timelines, suspicious command logs, and network traces.  
- I practice safe honeypot operation while collecting actionable intelligence.

## ⚠️ Safety Considerations
- I make sure the VM is fully isolated from production networks.  
- I use non-root accounts to handle all honeypot artifacts.  
- I monitor SELinux for denials and adjust policies as needed.  
- I deploy honeypots only on systems where I have proper authorization.

## 🔧 Utilities & Technologies I’m Using

| Tool / Technology | Purpose | Notes |
|------------------|---------|-------|
| 🖥️ Rocky Linux | Base operating system | Latest stable version |
| 📡 tcpdump | Network packet capture | I save PCAP files to analyze traffic |
| 📜 rsyslog | System logging | I monitor and store log events |
| 🛡️ firewalld | Network isolation & egress control | I block outbound traffic to prevent misuse |
| 🛠️ vim, git, wget, curl, net-tools | System management & troubleshooting | Core tools I use for setup and monitoring |
| 🔐 SELinux tools (policycoreutils) | Security enforcement | I monitor and adjust SELinux policies as needed |
| 👤 Non-root honeypot user | Artifact ownership | I ensure logs and captures remain isolated |
| 📊 Optional SIEM (Splunk, ELK) | Log ingestion & visualization | I forward logs for correlation and visualization |

---

### 1️⃣ Update & Install Tools
I update the system and install all required utilities. I enable logging and firewall services so I can **capture activity and control network traffic** from the start.

<img width="1009" height="685" alt="2V0K7Bu" src="https://github.com/user-attachments/assets/9a1a4664-5aa4-4ef0-890f-0b2208bd7ad0" />

# 🖥️ Bash Shell — What Is It Doing?
- **`sudo`** → I run commands as root because system changes need admin privileges.  
- **`dnf update -y`** → I update all packages automatically to keep the system patched.  
- **`dnf install -y <pkg>`** → I install critical tools (**tcpdump**, **rsyslog**, **firewalld**) to monitor and secure the system.

---

### 2️⃣ Configure Hostname
### I assign a realistic hostname (`backup-HP-01`) and update the hosts file to make the honeypot appear like a corporate backup server. This improves the chances that attackers will interact with it.

<img width="813" height="657" alt="TkPMROL" src="https://github.com/user-attachments/assets/babaaf46-841c-47e0-800e-a7a5fa734ea8" />

# 🖥️ Bash Shell — What Is It Doing?
- **`hostnamectl set-hostname backup-hp-01`** → I assign a clear system name for logs and identification.  
- **`cat >> /etc/hosts <<'EOF' ... EOF`** → I append host entries in one step using a here-doc.  
- **`hostname`** → I verify the hostname change took effect.
  
---

### 3️⃣ Verify Functionality
### I perform test captures and log entries to ensure that packet capturing and logging are working properly before adding bait content.

<img width="1013" height="362" alt="XIbYKIw" src="https://github.com/user-attachments/assets/2aa7ee29-71ba-4c67-9d48-f5fe7c5f3cea" />

<img width="952" height="39" alt="y1OuTbz" src="https://github.com/user-attachments/assets/2637effd-db58-4d5e-b2e8-6353377bffa4" />

# 🖥️ Bash Shell — What Is It Doing?
- **`mkdir -p /var/honeypot/captures`** → I create directories safely; **`-p`** prevents errors if they already exist.  
- **`tcpdump -i any -w /var/honeypot/captures/test.pcap -c 1`** → I capture 1 packet to verify packet capture works.  
- **`logger "honeypot rsyslog test"`** → I send a test message to syslog to verify logging.

---

### 4️⃣ Create Honeypot User & Directories
### I create a non-root honeypot user and set up directories for logs and captures. I make sure permissions are configured correctly so all artifacts are safe and isolated.

<img width="1006" height="443" alt="68kgIxq" src="https://github.com/user-attachments/assets/b15f8c54-094e-404c-a806-408c69b5403f" />

# 🖥️ Bash Shell — What Is It Doing?
- **`useradd -m -s /sbin/nologin honeypot`** → I create a restricted user to run honeypot processes safely.  
- **`chown -R honeypot:honeypot /var/log/honeypot`** → I give ownership of logs to the honeypot user.  
- **`chmod 750 /var/log/honeypot`** → I set directory permissions to restrict access.

---

### 5️⃣ Lock Down the Network
### I block outbound traffic using the firewall to ensure the honeypot cannot be used for pivoting or data exfiltration.

<img width="1014" height="146" alt="9baG0rl" src="https://github.com/user-attachments/assets/0575ad1d-3cf9-4811-9ce7-055fb513c06d" />

<img width="1002" height="548" alt="ARrUwbg" src="https://github.com/user-attachments/assets/832e6ccf-ece1-4d18-8fd8-b5d6a565e0c6" />

# 🖥️ Bash Shell — What Is It Doing?
- **`ip link show`** → I find the correct interface name for firewall rules.  
- **`firewall-cmd --permanent --zone=drop --add-interface=ens33`** → I block outbound traffic to isolate the honeypot.  
- **`firewall-cmd --reload`** → I apply the firewall changes immediately.

---

### 6️⃣ Test Logging & Captures
### I capture a few packets and log test messages to verify that all data collection mechanisms are functioning correctly.

<img width="1006" height="340" alt="avWcwK5" src="https://github.com/user-attachments/assets/ef16361a-d7a3-4087-98e1-b46163f6e26f" />

<img width="1003" height="144" alt="USsEO7R" src="https://github.com/user-attachments/assets/797e123b-ac61-4932-b225-b4aec64cc68e" />

# 🖥️ Bash Shell — What Is It Doing?
- **`tcpdump -i any -w /var/honeypot/captures/test.pcap -c 10`** → I capture multiple packets to confirm monitoring works.  
- **`logger "Test honeypot log message"`** → I generate a test log entry for verification.  
- **`ls -lh /var/honeypot/captures/test.pcap`** → I confirm the capture file exists and has size.

---

### 7️⃣ Post-Setup Verification
### I deploy realistic backup files and banners to attract attacker interaction, generating richer logs and packet captures for analysis.

<img width="1017" height="268" alt="Im2LvKL" src="https://github.com/user-attachments/assets/49cc1638-8603-46da-86f3-63a105244355" />

<img width="1012" height="300" alt="IACeGNk" src="https://github.com/user-attachments/assets/b338126e-f184-46cd-b774-beeeb2aaae00" />

# 🖥️ Bash Shell — What Is It Doing?
- **`hostname`** → I double-check the hostname is correct.  
- **`ls -ld /var/log/honeypot /var/honeypot/captures`** → I verify directories, ownership, and permissions.  
- **`systemctl status --no-pager rsyslog firewalld`** → I confirm critical services are running properly.

---
### 8️⃣ Cron Job Setup & Validation

#### Cron Job Creation Using Nano
### I create and configure a cron job that runs my honeypot capture script automatically at regular intervals.

<img width="613" height="49" alt="NDaDvfi" src="https://github.com/user-attachments/assets/d365a0cd-5f13-4189-9254-c012675c69c8" />

<img width="920" height="148" alt="6BP4ioO" src="https://github.com/user-attachments/assets/70fc52cc-dbf2-4a5a-bffd-cd4005cd6f10" />

> # 🖥️ Bash Shell — What Is It Doing?
- I create `/etc/cron.d/honeypot-capture-user` using nano.
- Cron runs `/usr/local/bin/honeypot_capture_rotate_user.sh` every 30 minutes.
- Cron execution is logged to `/var/log/honeypot/honeypot-cron.log`.
- Root owns the cron file while the honeypot user executes the script.

---

#### Ensure Honeypot User Owns the Directories
### I set ownership and permissions so my honeypot user can safely write to the captures directory.

<img width="709" height="112" alt="ahfIOVN" src="https://github.com/user-attachments/assets/d72538a9-8fa2-4a1f-916f-44b3278732d2" />

> # 🖥️ Bash Shell — What Is It Doing?
- Ownership: `honeypot:honeypot`.
- Permissions: 644 to allow the owner to read and write the file while giving other users read-only access.
- Ensures captured PCAPs and logs remain isolated and secure.

---

#### Run Commands as Honeypot User
### All honeypot commands and validation tests are executed under the honeypot account to prevent permission issues.

<img width="1018" height="167" alt="cAgYyuQ" src="https://github.com/user-attachments/assets/9abd6e3c-3594-4f4d-803c-8d95014d9b96" />

> # 🖥️ Bash Shell — What Is It Doing?
- Commands are run with `sudo -u honeypot`.
- Includes running the capture script and listing captured files.
- Ensures cron simulation matches scheduled execution.

---

#### Ensure tcpdump Can Be Used by Honeypot User
### I grant Linux capabilities so the honeypot user can capture network packets without full root privileges.

<img width="738" height="90" alt="tbyJkc6" src="https://github.com/user-attachments/assets/9c8c0e14-4e7b-45c8-b778-492759b661ac" />

> # 🖥️ Bash Shell — What Is It Doing?
- `tcpdump` requires `cap_net_raw` and `cap_net_admin` capabilities.
- Verifying capabilities ensures non-root packet captures succeed.
- Prevents permission errors during automated captures.

---

#### Handle SELinux Write Permissions
### I adjust SELinux contexts to allow honeypot to write logs and captures without denials.

<img width="738" height="90" alt="tbyJkc6" src="https://github.com/user-attachments/assets/79f0eaaa-c802-4133-92b9-e9321ccab2c9" />

> # 🖥️ Bash Shell — What Is It Doing?
- Applies the proper security context to `/var/honeypot`.
- `restorecon` ensures changes take effect.
- Prevents AVC denials and cron job failures.

---

## 🔍 Using Captured Files for Threat Hunting
The PCAPs, logs, and metadata generated by the honeypot will be analyzed to identify attacker behavior, suspicious patterns, and potential indicators of compromise (IOCs). These artifacts form the foundation for threat hunting exercises and security investigations.

### Notes
- Packet captures (`.pcap.gz`) are examined to detect network scanning, brute-force attempts, and other reconnaissance activity.  
- Log archives (`hp_logs_*.tar.gz`) contain system events, authentication attempts, and honeypot interactions for timeline analysis.  
- Metadata files (`hp_meta_*.txt`) store contextual information about each capture, such as timestamps, script version, and system state.  
- Checksums (`.sha256`) verify the integrity of artifacts for analysis and reporting.  
- Screenshots can demonstrate analysis in tools such as Wireshark, Zeek, or Splunk.  
- These artifacts will be used to create **playbooks, threat intelligence reports, and actionable insights** for security operations and SOC exercises.

