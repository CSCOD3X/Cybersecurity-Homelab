# Cybersecurity-Homelab
Security Monitoring Platform


## Overview

The SOC Homelab project implements a complete enterprise-grade Security Operations Center platform for security monitoring, threat detection, and incident response. The lab simulates a multi-site enterprise environment with network segmentation, centralized logging, intrusion detection, and endpoint security, providing hands-on experience with professional security tools and workflows.

---

## Components and Their Purposes

### pfSense Firewalls (Virtual Machines - Site A & Site B)

Serve as enterprise-grade network security appliances providing advanced firewalling, routing, and site-to-site VPN capabilities. Allow creation of network segments, firewall rules with default deny policies, and management access restrictions. Facilitate practice of network security, traffic monitoring, and firewall logging to SIEM.

### Suricata IDS/IPS (Raspberry Pi 5)

Runs as a dedicated network security sensor on Raspberry Pi 5 hardware. Loads 65,796 detection rules including malware C2, port scanning, exploit detection, and web attack signatures. Provides real-time intrusion detection and traffic analysis on the network edge.

### Wazuh XDR (Ubuntu VM - SOC Controller)

Acts as a centralized Extended Detection and Response platform for endpoint security. Collects telemetry from Windows 10, Windows Domain Controller, Red Hat Linux, and K8s Node agents. Provides file integrity monitoring, malware detection, rootkit detection, and MITRE ATT&CK mapping for security events.

### ELK Stack (Ubuntu VM - SOC Controller)

Elasticsearch, Logstash, and Kibana work together as a complete SIEM solution. Logstash receives syslog from pfSense firewalls on port 5140 and parses CSV fields. Elasticsearch indexes 300,000+ Suricata events and 140,000+ pfSense logs. Kibana provides real-time security dashboards for threat visualization and incident investigation.

### Uptime Kuma (Docker Container on Ubuntu VM)

Provides service health monitoring for all SOC infrastructure components. Tracks uptime, response times, and sends alerts when services become unavailable.

### Tailscale VPN

Enables secure remote access to the entire lab environment from outside locations. Provides encrypted tunnel without complex firewall configuration.

### Kali Linux (Attack Machine)

Serves as the dedicated attack machine for penetration testing and detection validation. Used to simulate real-world attacks including port scans, SSH brute force, web directory enumeration, and DoS attacks to validate SOC detection capabilities.

### Internal Endpoints

**Windows 10 VM:** Provides Windows-based endpoint for monitoring testing and client-side attack simulation.

**Windows Domain Controller:** Acts as central authority for authentication and Active Directory services, enabling practice with domain security monitoring.

**Red Hat Linux:** Serves as Linux production server for SSH brute force testing and Linux security monitoring.

**K8s Node:** Provides container host environment for Kubernetes security monitoring practice.

---

## Architecture Overview

The lab consists of two pfSense firewalls connected via IPsec site-to-site VPN. Site A manages internal network 10.0.2.0/24 hosting Windows workloads. Site B manages internal network 10.0.3.0/24 hosting Linux workloads. A Raspberry Pi 5 runs Suricata IDS monitoring network traffic. An Ubuntu VM serves as the SOC controller running Elasticsearch, Kibana, Logstash, Wazuh Manager, and Uptime Kuma. All pfSense logs are sent via syslog to Logstash on port 5140. Suricata logs are shipped via Filebeat to Elasticsearch. Wazuh agents send telemetry to the Wazuh Manager, which forwards alerts to Elasticsearch. Kibana provides unified visualization across all data sources.

---

## Detection Capabilities

**Network Layer:** Port scanning detection via threshold monitoring, malware C2 via signature matching, suspicious protocols via protocol anomaly detection, DDoS patterns via rate-based monitoring, known exploits via vulnerability signatures.

**Endpoint Layer:** Malware execution via file integrity monitoring, privilege escalation via auditd, persistence via registry monitoring, lateral movement via authentication failure correlation.

**Firewall Layer:** Unauthorized access via blocked connection logs, reconnaissance via multiple blocked ports from same source, policy violations via rule match logging, VPN issues via authentication failure logs.

---

## Attack Simulations Validated

**Port Scan:** Nmap scan against pfSense WAN interface triggered Suricata "ET SCAN Nmap" alert with 4,180+ hits in Kibana.

**SSH Brute Force:** Hydra and repeated failed SSH attempts triggered Wazuh multiple failed login alerts and pfSense NAT access logs.

**Web Directory Scan:** Gobuster enumeration triggered Suricata suspicious User-Agent alerts.

**EICAR Malware Simulation:** EICAR test file transfer triggered Wazuh malware signature detection.

**Ping Flood:** ICMP flood triggered pfSense DoS pattern detection logs.

---

## Service Access

- **Kibana Dashboard:** http://192.168.1.55:5601
- **Uptime Kuma:** http://192.168.1.55:3001
- **pfSense-1 GUI:** https://192.168.1.40:8443
- **pfSense-2 GUI:** https://192.168.1.41:8443

Management access restricted to MANAGE_NETS alias (192.168.1.0/24).

---

## Key Achievements

- Deployed dual pfSense firewalls with site-to-site IPsec VPN and network segmentation
- Implemented Suricata IDS/IPS on dedicated Raspberry Pi 5 with 65,796 detection rules
- Deployed Wazuh XDR across Windows and Linux endpoints with FIM and malware detection
- Built complete ELK Stack processing 300,000+ Suricata events and 140,000+ pfSense logs
- Created three professional Kibana dashboards for network, endpoint, and executive monitoring
- Validated five attack types with real-time detection in Kibana

---

## Technologies Used

- **Firewall:** pfSense CE 2.7.2
- **IDS/IPS:** Suricata 7.0.5
- **SIEM/XDR:** Wazuh 4.7.0
- **Log Storage:** Elasticsearch 7.17.15
- **Log Processing:** Logstash 7.17.15
- **Visualization:** Kibana 7.17.15
- **Log Shipping:** Filebeat 7.17.15
- **Service Monitoring:** Uptime Kuma 1.23.0
- **VPN:** Tailscale
- **Attack Machine:** Kali Linux
- **Sensor Hardware:** Raspberry Pi 5
- **Host OS:** Ubuntu 20.04

---

By setting up this comprehensive SOC environment, users gain hands-on experience in network security monitoring, intrusion detection, endpoint security, SIEM operations, log management, security dashboarding, attack simulation, and incident response. The lab provides a safe, controlled environment to practice professional security monitoring skills in a realistic multi-site enterprise context.
