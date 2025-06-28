# DDoS Attack Detection & Mitigation – Linux Commands Guide

This file contains a complete set of Linux commands and explanations to simulate, detect, and mitigate DDoS attacks in a **controlled environment** using tools like Apache2, hping3, netstat, iptables, and Fail2Ban.

---

## Step 1: Set Up a Local Apache Web Server

```bash
# Install Apache
sudo apt update && sudo apt install apache2 -y

# Start the Apache server
sudo systemctl start apache2

# Enable Apache to start on boot
sudo systemctl enable apache2

# Test if server is running
curl http://localhost
```

*This sets up a local HTTP server on port 80.*

---

## Step 2: Simulate a DDoS Attack using hping3

```bash
# Install hping3 (packet generator and DDoS simulation tool)
sudo apt install hping3 -y

# Launch a SYN flood attack on port 80 from spoofed IPs
sudo hping3 -S -p 80 --flood --rand-source 127.0.0.1
```

*Simulates a TCP SYN flood attack with random source IPs.*

*Use only in isolated test environments!*

---

##  Step 3: Detect the DDoS Attack

### A. Count Total Active Connections on Port 80

```bash
netstat -an | grep :80 | wc -l
```

### B. Identify the Top Source IPs Hitting the Server

```bash
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head
```

*These commands help detect abnormal connection counts and highlight attacker IPs.*

---

## Step 4: Mitigate the Attack – Block Attacker IPs

### A. Block a Single Attacker IP

```bash
sudo iptables -A INPUT -s <ATTACKER_IP> -j DROP
```

> Replace `<ATTACKER_IP>` with a real IP from the detection step.

### B. Block Multiple IPs from a File

```bash
# Create a file with each IP on a new line
echo -e "ip1\nip2\nip3" > blocked_ips.txt

# Read and block each IP
while read ip; do sudo iptables -A INPUT -s "$ip" -j REJECT; done < blocked_ips.txt
```

---

## Step 5: Automatic IP Banning with Fail2Ban

### A. Install and Start Fail2Ban

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### B. Configure Jail Settings

```bash
sudo nano /etc/fail2ban/jail.local
```

> Inside the file, define rules for services (e.g., `[apache-auth]`) and set:
> - `maxretry`
> - `findtime`
> - `bantime`

 *Fail2Ban will automatically ban IPs based on logs and retry thresholds.*

---

##  Step 6: Apply Rate Limiting using iptables

```bash
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 10/s --limit-burst 20 -j ACCEPT
```

 *This allows a maximum of 10 connections per second with a burst of 20 before limiting new ones.*

---

## Step 7: Recheck Connections After Mitigation

```bash
netstat -an | grep :80 | wc -l
```

 *Use this to verify that the number of connections has decreased after applying blocking or rate limits.*

---

## Summary of Tools

| Tool       | Purpose                                      |
|------------|----------------------------------------------|
| `apache2`  | To run a web server                          |
| `hping3`   | To simulate TCP SYN DDoS attacks             |
| `netstat`  | To monitor network connections               |
| `iptables` | To block/limit IPs or apply rate limiting    |
| `Fail2Ban` | To automate IP banning using log analysis    |

---

## Disclaimer

This guide is for **educational and research** purposes **only**.  
Never simulate DDoS attacks on public or unauthorized networks.

Use all commands **only in controlled lab/test environments**.

---
