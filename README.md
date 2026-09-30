# Snort-IDS-Attack-Simulation
Network security lab using Snort IDS on Ubuntu Server to detect ICMP ping and SSH brute-force attacks simulated from Kali Linux (Nmap, Hydra).

A small-scale network security lab where **Snort IDS** running on an Ubuntu Server detects attacks simulated from **Kali Linux**. The lab covers ICMP ping detection and SSH login/brute-force detection.

>  **Disclaimer:** This project was built in an isolated lab for educational purposes only. Never run these attacks against systems you do not own or have permission to test.
 Objectives

- Set up two virtual machines: Kali Linux (attacker) and Ubuntu Server (target with Snort)
- Install OpenSSH on the Ubuntu Server
- Install and configure Snort IDS
- Write custom Snort rules to detect ICMP and SSH traffic
- Simulate attacks using Nmap and Hydra
- Analyse Snort's alerts

---

##  Tools & Technologies

| Tool | Purpose |
|------|---------|
| VirtualBox | Virtualization platform |
| Kali Linux 2024.3 | Attacker machine |
| Ubuntu Server 24.04.1 LTS | Target machine running Snort |
| Snort 2.9.20 | Intrusion Detection System |
| OpenSSH | SSH service on the target |
| Nmap | Port scanning |
| Hydra | SSH brute-force attack |
| Snorpy | Web-based Snort rule creator (explored) |

---

##  Lab Setup

| Machine | Role | IP Address |
|---------|------|------------|
| Ubuntu Server | Target + Snort IDS | 192.168.1.10 |
| Kali Linux | Attacker | 192.168.1.11 |

**Network configuration**
- Both VMs use a **Bridged Adapter** so they are on the same network (192.168.1.0/24)
- **Promiscuous Mode** is set to **Allow All** on the Ubuntu Server VM so Snort can see all traffic
- Connectivity was verified with `ifconfig` and `ping`

---

##  Snort Configuration

1. Installed Snort:
```bash
   sudo apt-get update
   sudo apt-get install snort -y
   snort --version
```
2. Edited `/etc/snort/snort.conf` and set the protected network:
```
   ipvar HOME_NET 192.168.1.0/24
   ipvar EXTERNAL_NET any
```
3. Added custom rules in `/etc/snort/rules/local.rules`

---

##  Custom Snort Rules

```
alert icmp any any -> $HOME_NET any (msg:"ICMP Ping Detected"; sid:100001; rev:1;)
alert tcp any any -> $HOME_NET 22 (msg:"SSH Authentication Attempt"; sid:100002; rev:1;)
```

| Rule | SID | Detects |
|------|-----|---------|
| ICMP rule | 100001 | Ping requests to the network |
| SSH rule | 100002 | Any TCP connection to port 22 |


##  Running Snort

```bash
sudo snort -q -l /var/log/snort -i enp0s3 -A console -c /etc/snort/snort.conf
```

| Option | Meaning |
|--------|---------|
| `-q` | Quiet mode |
| `-l /var/log/snort` | Log directory |
| `-i enp0s3` | Network interface to monitor |
| `-A console` | Print alerts to the console |
| `-c /etc/snort/snort.conf` | Configuration file |


##  Attack Simulations

### 1. ICMP Ping
```bash
ping 192.168.1.10
```
**Result:** Snort raised `ICMP Ping Detected` alerts (SID 100001).

### 2. SSH Brute-Force (Hydra)
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 192.168.1.10 -t 5 ssh
```
**Result:** Snort generated a large number of `SSH Authentication Attempt` alerts (SID 100002) from 192.168.1.11 to 192.168.1.10:22.

### 3. Port Scan (Nmap)
```bash
nmap -p 22 192.168.1.10
```
**Result:** Port 22 was found open, and Snort detected the connection attempts to SSH.

---

##  Results

- Snort successfully detected ICMP ping requests in real time
- Snort successfully detected the Hydra SSH brute-force attempts
- Snort also detected Nmap probing of the SSH port
- Alerts showed the timestamp, rule ID, message, protocol, source and destination IP/port

---

##  Future Improvements

- Add threshold rules to separate real brute-force activity from a single SSH connection
- Add a rule to detect Nmap port scans specifically
- Build a dashboard to visualise Snort alerts (for example with Splunk or an ELK stack)
- Send alerts to log files and set up email notifications

---

##  Full Report

The complete step-by-step documentation with screenshots is available here:
[Project_IDS_snort.pdf](ProjectIDSsnort.pdf)

---

