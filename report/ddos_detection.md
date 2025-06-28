# DDoS Attack Detection & Mitigation

---

## Introduction

Distributed Denial of Service (DDoS) attacks are malicious attempts to disrupt the normal traffic of a targeted server, service, or network by overwhelming it with a flood of internet traffic. In Software Defined Networks (SDN), due to the centralized control architecture, there is a greater opportunity to detect and mitigate such attacks dynamically.

---

## Objective

This project aims to implement a *two-stage approach* for real-time detection and mitigation of DDoS attacks in an SDN-like environment.

- *Stage 1 – Detection*: Use wavelet transformation for feature extraction and a CNN-based model for accurate attack classification.
- *Stage 2 – Mitigation*: Perform source tracing through graph traversal and apply flow-based rules using iptables or SDN controller.

---

## Tools & Technologies

- *Apache2* – Local HTTP web server
- *hping3* – DDoS simulation using SYN Flood
- *netstat, **awk, **grep, **iptables* – Network monitoring and IP blocking
- *Fail2Ban* – Log-based automated banning
- *Linux (Ubuntu)* – Host environment for implementation

---

## Proposed Architecture

1. *Web Server Simulation*:
   - Apache2 used to host a local site.
   - Simulates a real-world service that can be targeted.

2. *Attack Simulation*:
   - hping3 used to flood the server with TCP SYN packets from random spoofed sources.

3. *Detection*:
   - Monitor network using netstat, awk, and grep.
   - Identify top source IPs and connection counts.

4. *Mitigation*:
   - Use iptables to block individual or multiple IPs.
   - Enable automatic blocking with Fail2Ban.
   - Apply traffic rate limiting.

---

## Experimental Steps

### Step 1: Start Local Server

sudo apt install apache2 -y
sudo systemctl start apache2

## Step 2: Simulate DDoS
sudo apt install hping3 -y
sudo hping3 -S -p 80 --flood --rand-source 127.0.0.1

## Step 3: Detect Attack
netstat -an | grep :80 | wc -l
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head

## Step 4: Block Attacker IPs
sudo iptables -A INPUT -s <ATTACKER_IP> -j DROP

## Step 5: Auto Mitigation with Fail2Ban
sudo apt install fail2ban -y
sudo systemctl start fail2ban
sudo nano /etc/fail2ban/jail.local

## Step 6: Apply Rate Limiting
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 10/s --limit-burst 20 -j ACCEPT
