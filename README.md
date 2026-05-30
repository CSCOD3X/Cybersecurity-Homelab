# Cybersecurity-Homelab
![Status](https://img.shields.io/badge/Status-Under%20Development-orange)
![Last Updated](https://img.shields.io/badge/Updated-May%202026-blue)

> This is a continuous learning project
## Overview

The Homelab project implements a complete Security Operations Center platform for security monitoring, threat detection. The lab simulates a multi-site enterprise environment with network segmentation, centralized logging, intrusion detection, and endpoint security, providing hands-on experience with professional security tools and workflows.

---

<img width="7673" height="9394" alt="Cybersecurity Homelab(future)" src="https://github.com/user-attachments/assets/d066672a-4e42-4bc3-b037-ea1f0b7c271b" />


---

## Components and Their Purposes

### pfSense Firewalls (Virtual Machines - Site A & Site B)

Serve as enterprise-grade network security appliances providing advanced firewalling, routing, and site-to-site VPN capabilities. Allow creation of network segments, firewall rules with default deny policies, and management access restrictions. Facilitate practice of network security, traffic monitoring, and firewall logging to SIEM. Also added snort IDS/IPS for internal network traffic detection and blocking suspicous traffic from the attacker.

### Physical Cisco Catalyst Switch Stack (2960-CX & 3560-CX Compact Hardware)

Acts as the high-speed physical network backplane across the desk. The 2960-CX Layer 2 switch aggregates physical PC hosts and trunks VLAN tags over a 2-cable LACP EtherChannel bundle to the 3560-CX Layer 3 Core switch, which processes hardware-level inter-VLAN routing and mirrors packet fragments via a SPAN port.

### TrueNAS Core IP SAN (Virtual Machine - Proxmox on Dell Optiplex)

Operates on a dedicated bare-metal hypervisor node to serve as an enterprise-grade block-storage SAN appliance. Passes through a physical 4TB external hard drive to carve out separate low-latency iSCSI LUN block allocations over a completely isolated, un-routed Storage Fabric (VLAN 88, Jumbo MTU 9000).

### Suricata IDS/IPS (Docker Container on Raspberry Pi 5)

Runs as a dedicated network security sensor on Raspberry Pi 5 hardware. Intercepts raw, uncompressed network traffic directly from the physical Cisco switch's hardware SPAN port. Loads **65,796 detection rules** including malware C2, port scanning, exploit detection, and web attack signatures.

### Wazuh XDR (Docker Container on Ubuntu VM - SOC Controller)

Acts as a centralized Extended Detection and Response platform for endpoint security. Collects telemetry from Windows 10, Windows Domain Controller, Red Hat Linux, and K8s Node agents. Provides file integrity monitoring, malware detection, rootkit detection, and **MITRE ATT&CK mapping** for security events.

### ELK Stack (Docker Container on Ubuntu VM - SOC Controller)

Elasticsearch, Logstash, and Kibana work together as a complete SIEM solution. Logstash centralizes log intake, receiving direct syslog tables from Palo Alto/FortiGate on port 5140 and processing Suricata/Filebeat fields. Elasticsearch indexes **300,000+ Suricata events** and **140,000+ firewall logs**, utilizing a high-speed 2.5TB iSCSI SAN LUN to optimize disk I/O performance. Kibana provides real-time security dashboards for threat visualization and incident investigation.

### Shuffle SOAR Platform (Docker Container on Ubuntu VM)

Provides automated incident response and orchestration. Intercepts high-severity alerts from the Wazuh Manager and executes dynamic playbooks, communicating back with security gateways to automate containment workflows.

### Uptime Kuma (Docker Container on Ubuntu VM)

Provides service health monitoring for all SOC infrastructure components. Tracks uptime, response times, and sends alerts when services become unavailable.

### Tailscale VPN

Enables secure remote access to the entire lab environment from outside locations.

### Pi-hole (DNS Sinkhole)

Deployed on the same Raspberry Pi 5 security sensor, providing DNS-level threat intelligence and query log forwarding to Logstash while acting as a DNS sinkhole to drop outbound malware C2 domains.

### n8n Automation Platform (Docker Container on Ubuntu VM)
Provides workflow automation and integration across SOC services. Acts as a low-code orchestration engine to connect Suricata, Wazuh, ELK, Shuffle SOAR, and external APIs. Enables automated enrichment of alerts (e.g., querying VirusTotal, WHOIS, or threat intel feeds), ticket creation in Jira, and notification routing to email/Slack. Simplifies repetitive SOC tasks by chaining triggers and actions into visual workflows, reducing manual analyst workload and accelerating incident response.

### Linode Cloud VPS (Public Cloud Debian Instance)

Hosts a standalone public cloud **T-Pot Honeypot Framework** out-of-band. Captures real-world internet attacker botnets, brute-force payloads, and malicious IPs safely away from the home infrastructure, shipping raw threat intelligence telemetry back to the local Logstash container over TLS.

### Kali Linux (Attack Machine)

Serves as the dedicated attack machine for penetration testing and detection validation. Used to simulate real-world attacks including port scans, SSH brute force, web directory enumeration, and DoS attacks to validate SOC detection capabilities.

### SANS SIFT Workstation (DFIR vm)
Provides an isolated, professional Digital Forensics and Incident Response (DFIR) workspace to mount raw system images and analyze historical packet captures for advanced threat investigations.

### Internal Endpoints

**Windows 10 VM:** Provides Windows-based endpoint for monitoring testing and client-side attack simulation.

**Windows Domain Controller:** Acts as central authority for authentication and Active Directory services. Runs the **Azure AD (Microsoft Entra) Connect Sync agent** to establish outbound Password Hash Synchronization (PHS) to a cloud tenant, enabling hybrid identity tracking and User-ID firewall integration.

**Red Hat Linux:** Serves as Linux production server for SSH brute force testing and Linux security monitoring.

**K8s Node:** Provides container host environment for Kubernetes security monitoring practice.


---

## Architecture Overview

The lab splits computing workloads across a multi-host distributed topology to prevent performance bottlenecks. PC 1 (64GB ThinkPad Laptop) runs VirtualBox to host the Pfsense-1 VM, Kali Linux, and the heavy central ELK/Wazuh SIEM stack. PC 2 (32GB Desktop Tower) splits resources by running a bare-metal Proxmox hypervisor dedicated strictly to your TrueNAS SAN array, while running VirtualBox on the base desktop OS to host the Pfsense-2 VM, Windows DC, and client endpoints. All physical nodes connect via Cat6 to a Cisco 2960-CX Layer 2 switch, bundling traffic over an LACP EtherChannel trunk to a Cisco 3560-CX Layer 3 switch. Networks are dual-homed across the Cisco backplane: Management and Azure cloud identity syncing route over **VLAN 99 (MTU 1500)**, while TrueNAS shares raw block storage out-of-band over **VLAN 88 (Jumbo MTU 9000)** with blank default gateways directly to the Elasticsearch data directories. Logstash parses incoming local firewall logs, public cloud Linode T-Pot honeypot events, and Pi 5 Suricata sensor traffic, ascending data vertically up a strict security operations hierarchy.

---

## Detection Capabilities

**Network Layer:** Port scanning detection via threshold monitoring, malware C2 via signature matching, suspicious protocols via protocol anomaly detection, DDoS patterns via rate-based monitoring, known exploits via vulnerability signatures.

**Endpoint Layer:** Malware execution via file integrity monitoring, privilege escalation via auditd, persistence via registry monitoring, lateral movement via authentication failure correlation.

**Firewall Layer:** Unauthorized access via blocked connection logs, reconnaissance via multiple blocked ports from same source, policy violations via rule match logging, VPN issues via authentication failure logs.

**Cloud Identity & Ingestion Layer:** Account compromise via anomalous Azure Active Directory sign-in indicators, automated brute-force containment via Shuffle SOAR orchestration, full User-ID mapping to track malicious traffic by on-premises and cloud user profiles rather than static IPs.

---

## Attack Simulations Validated

**Port Scan:** Nmap scan against pfSense WAN interface triggered Suricata "ET SCAN Nmap" alert with **4,180+ hits** in Kibana.

**SSH Brute Force:** Hydra and repeated failed SSH attempts triggered Wazuh multiple failed login alerts and pfSense NAT access logs.

**Ping Flood:** ICMP flood triggered pfSense DoS pattern detection logs.

**SMB enumeration using enum4linux:** Detected scans on nas storage.


---
## Key Achievements

- Deployed dual pfSense firewalls with site-to-site IPsec VPN and network segmentation
- Implemented Suricata IDS/IPS on dedicated Raspberry Pi 5 with **65,796 detection rules**
- Deployed Wazuh XDR across Windows and Linux endpoints with FIM and malware detection
- Built complete ELK Stack processing **300,000+ Suricata events** and **140,000+ pfSense logs**
- Created two professional Kibana dashboards for network, endpoint
- Validated **4 attack types** (port scan, SSH brute force, DoS, SMB enumeration) with full detection coverage

---

## Service Access

**Kibana Dashboard:** https://kibana.duckdns.org:5601

**Uptime Kuma:** https://status.duckdns.org:3001

**Shuffle SOAR:** https://soar.duckdns.org:3002

**Pfsense-1 VM GUI:** https://192.168.1.40:8443

**Pfsesne-2 VM GUI:** https://192.168.1.41:8443

**Pi-hole GUI:** http://192.168.1.39:80

---

## Upcoming Enhancements

- **Next-Generation Firewall (NGFW) Migration:** Transition network boundaries away from legacy pfSense appliances by deploying a multi-vendor **Palo Alto VM-Series** edge gateway and a corporate **FortiGate VM** cluster to practice advanced zone-based filtering, SSL decryption, and enterprise security management.
- **Cross-Vendor Hybrid Mesh VPN:** Establish a production-grade **IPsec Site-to-Site VPN Tunnel** directly linking the Palo Alto core (PC 1) and FortiGate domain (PC 2) to secure traffic routing between distinct physical and virtual compute hosts.
- **Dedicated iSCSI IP SAN Array:** Virtualize **TrueNAS Core** within a bare-metal **Proxmox hypervisor** on the secondary Dell Optiplex to instantiate a high-performance block-storage fabric. Allocate separate **iSCSI LUN block drives** over an isolated, un-routed **Storage Network (VLAN 88)** running Jumbo Frames (**MTU 9000**) to optimize Elasticsearch disk IOPS and host native VirtualBox `.vdi` disk cores.
- **Three-Tier Distributed Storage Pool:** Segment hardware assets into distinct operational layers by maintaining the Raspberry Pi 5 **Samba CE NAS** as a rolling Suricata PCAP packet dump vault (Tier 2) and a cold VM disaster recovery backup repository (Tier 3), completely independent of the hot TrueNAS block array (Tier 1).
- **Physical Cisco Backplane Integration:** Implement an enterprise hardware core/access topology using a physical **Cisco Catalyst 2960-CX** Layer 2 access switch and a **3560-CX** Layer 3 core routing switch. Aggregate multi-host links using **LACP EtherChannel bundling** and enforce strict **802.1Q VLAN trunk isolation** across the desktop workspace.
- **Out-of-Band Hardware Traffic Mirroring:** Configure a hardware **SPAN port** on the physical Cisco switches to mirror raw network traffic fragments straight into the out-of-band Raspberry Pi 5's capture interface (`eth0`), eliminating local hypervisor packet collection overhead.
- **Hybrid Cloud Identity Synchronization:** Deploy the **Microsoft Entra Cloud Sync** agent on the on-premises Windows Server Domain Controller to establish encrypted Password Hash Synchronization (PHS) to an **Azure Active Directory cloud tenant**, enabling hybrid identity management and unlocking username-tracked User-ID logging profiles on edge firewalls.
- **Cloud-Native Threat Intelligence Node:** Spin up a public cloud **Linode VPS instance** running a standalone **T-Pot Honeypot Framework** (Cowrie, Dionaea, Honeytrap) to safely capture live global internet attacker telemetry. Route the external JSON alert streams down via TLS into the local Logstash pipeline to cross-correlate public Indicators of Compromise (IOCs) against home network edge logs.
- **Shuffle SOAR Orchestration:** Build automated incident response and containment playbooks inside the **Shuffle SOAR platform**, parsing high-severity Wazuh alerts to execute automated API calls back to network security gateways.
- **SANS SIFT Forensics Workspace:** Integrate a dedicated **SANS SIFT Workstation** environment into the post-incident pipeline, providing security analysts with specialized tools to mount raw disk images, extract system artifacts, and run timeline forensics for deep-dive threat investigations.
- **Dynamic Edge Security Overhauls:** Package **DuckDNS** and an Nginx SSL reverse proxy inside the Raspberry Pi 5 Docker environment to track dynamic public WAN IP changes and enforce fully trusted Let's Encrypt HTTPS/SSL certificates across all internal lab management web UIs via secure **Tailscale Split-DNS** tunnels.


---

By setting up this comprehensive enterprise-grade SOC environment, users gain hands-on experience in next-generation firewall management, cross-vendor IPsec VPN, SIEM operations, IDS/IPS deployment, endpoint detection and response, SOAR orchestration, honeypot intelligence gathering, SAN storage architecture, hybrid cloud identity synchronization, and professional DFIR workflows. The lab provides a safe, controlled environment to practice real-world security monitoring skills in a realistic multi-site enterprise context with hardware-accelerated switching and dedicated storage fabric.

---

Repository Maintained by Ahmed Ashraf | Last Updated: 22 May 2026
