# 🛡️ Backup-Server Trap Honeypot 🗄️

## 📖 Summary
I’m running a Rocky Linux honeypot that simulates a corporate backup server. My goal is to **observe attacker behavior**, study how malicious activity appears in logs and packet captures, and **learn how to produce actionable artifacts** for incident response (IR) and stakeholder reporting.

The honeypot runs in an isolated environment to prevent unauthorized access or exfiltration, while providing realistic bait files and banners to attract potential threats. This setup allows me to **practice detection, analysis, and containment** in a safe, controlled environment.

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

# 🖥️ Bash Shell — What Is It Doing?
- **`mkdir -p /var/honeypot/captures`** → I create directories safely; **`-p`** prevents errors if they already exist.  
- **`tcpdump -i any -w /var/honeypot/captures/test.pcap -c 1`** → I capture 1 packet to verify packet capture works.  
- **`logger "honeypot rsyslog test"`** → I send a test message to syslog to verify logging.

### 4️⃣ Create Honeypot User & Directories
### I create a non-root honeypot user and set up directories for logs and captures. I make sure permissions are configured correctly so all artifacts are safe and isolated.

# 🖥️ Bash Shell — What Is It Doing?
- **`useradd -m -s /sbin/nologin honeypot`** → I create a restricted user to run honeypot processes safely.  
- **`chown -R honeypot:honeypot /var/log/honeypot`** → I give ownership of logs to the honeypot user.  
- **`chmod 750 /var/log/honeypot`** → I set directory permissions to restrict access.

### 5️⃣ Lock Down the Network
### I block outbound traffic using the firewall to ensure the honeypot cannot be used for pivoting or data exfiltration.

# 🖥️ Bash Shell — What Is It Doing?
- **`ip link show`** → I find the correct interface name for firewall rules.  
- **`firewall-cmd --permanent --zone=drop --add-interface=ens33`** → I block outbound traffic to isolate the honeypot.  
- **`firewall-cmd --reload`** → I apply the firewall changes immediately.

### 6️⃣ Test Logging & Captures
### I capture a few packets and log test messages to verify that all data collection mechanisms are functioning correctly.

# 🖥️ Bash Shell — What Is It Doing?
- **`tcpdump -i any -w /var/honeypot/captures/test.pcap -c 10`** → I capture multiple packets to confirm monitoring works.  
- **`logger "Test honeypot log message"`** → I generate a test log entry for verification.  
- **`ls -lh /var/honeypot/captures/test.pcap`** → I confirm the capture file exists and has size.
- 
### 7️⃣ Post-Setup Verification
### I deploy realistic backup files and banners to attract attacker interaction, generating richer logs and packet captures for analysis.

# 🖥️ Bash Shell — What Is It Doing?
- **`hostname`** → I double-check the hostname is correct.  
- **`ls -ld /var/log/honeypot /var/honeypot/captures`** → I verify directories, ownership, and permissions.  
- **`systemctl status --no-pager rsyslog firewalld`** → I confirm critical services are running properly.

---

## 📈 Next Steps
- I plan to forward selected logs and captures to a SIEM for visualization and correlation.  
- I will create a short IR playbook covering detection, triage, containment, and evidence collection.  
- I run exercises to detect attacks and produce concise, professional reports for stakeholders.
