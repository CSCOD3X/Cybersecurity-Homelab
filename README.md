# Cybersecurity-Homelab
![Status](https://img.shields.io/badge/Status-Under%20Development-orange)
![Last Updated](https://img.shields.io/badge/Updated-May%202026-blue)

> This is a continuous learning project

## Overview

The Homelab project implements a complete Security Operations Center platform for security monitoring, threat detection. The lab simulates a multi-site enterprise environment with network segmentation, centralized logging, intrusion detection, and endpoint security, providing hands-on experience with professional security tools and workflows.

---

<img width="7936" height="9374" alt="Cybersecurity Homelab" src="https://github.com/user-attachments/assets/bbb4d389-26a6-4be3-805c-68b117e2a366" />

---

## Components and Their Purposes

### pfSense Firewalls (Virtual Machines - Site A & Site B)

Serve as enterprise-grade network security appliances providing advanced firewalling, routing, and site-to-site VPN capabilities. Allow creation of network segments, firewall rules with default deny policies, and management access restrictions. Facilitate practice of network security, traffic monitoring, and firewall logging to SIEM. Also added Snort IDS/IPS for internal network traffic detection and blocking suspicious traffic from the attacker.

### Physical Cisco Catalyst Switch Stack (Cisco Catalyst 2960CX & Cisco Catalyst 3560CX)

Acts as the high-speed physical network backplane across the desk. The Catalyst 2960CX Layer 2 switch aggregates physical PC hosts and trunks VLAN tags over a 2-cable LACP EtherChannel bundle to the Catalyst 3560CX Layer 3 Core switch, which processes hardware-level inter-VLAN routing and mirrors packet fragments via a SPAN port.

### TerraMaster F4-425 NAS — iSCSI IP SAN

Serves as the dedicated enterprise-grade network storage appliance providing centralized block storage and file services across the lab. The TerraMaster F4-425 is a 4-bay NAS with a quad-core Intel processor and 10GbE capability, running TOS (TerraMaster Operating System) to manage storage pools, RAID arrays, and iSCSI targets. Configured with an isolated Storage VLAN (VLAN 88, Jumbo MTU 9000) to deliver low-latency iSCSI LUN block storage directly to Elasticsearch data directories for optimized disk I/O performance.

### Suricata IDS/IPS (Docker Container on Raspberry Pi 5)

Runs as a dedicated network security sensor on Raspberry Pi 5 hardware. Intercepts raw, uncompressed network traffic directly from the physical Cisco switch's hardware SPAN port. Loads **65,796 detection rules** including malware C2, port scanning, exploit detection, and web attack signatures.

### Wazuh XDR (Docker Container on Ubuntu VM - SOC Controller)

Acts as a centralized Extended Detection and Response platform for endpoint security. Collects telemetry from Windows 10, Windows Domain Controller, Red Hat Linux, and K8s Node agents. Provides file integrity monitoring, malware detection, rootkit detection, and **MITRE ATT&CK mapping** for security events.

### ELK Stack (Docker Container on Ubuntu VM - SOC Controller)

Elasticsearch, Logstash, and Kibana work together as a complete SIEM solution. Logstash centralizes log intake, receiving syslog from pfSense firewalls on port 5140 and processing Suricata and Filebeat fields. Elasticsearch indexes **300,000+ Suricata events** and **140,000+ firewall logs**, utilizing the TerraMaster iSCSI SAN LUN over VLAN 88 to optimize disk I/O performance. Kibana provides real-time security dashboards for threat visualization and incident investigation.

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

### SANS SIFT Workstation (DFIR VM — Proxmox on Dell Optiplex)

Provides an isolated, professional Digital Forensics and Incident Response (DFIR) workspace dedicated to post-incident analysis. Runs as a standalone VM on the Proxmox hypervisor hosted on the Dell Optiplex, keeping forensic workloads completely separate from the live SOC environment. Used to mount raw disk images, extract system artifacts, analyze memory dumps, and perform timeline forensics for advanced threat investigations. Forensic evidence stored on TerraMaster NAS for integrity and long-term retention.

### REMnux Workstation (Malware Analysis VM — Proxmox on Dell Optiplex)

Provides a dedicated malware analysis and reverse engineering environment running as a standalone VM on the Proxmox hypervisor. REMnux is a Linux distribution purpose-built for analyzing malware samples, examining suspicious files, and reverse engineering binaries. Used alongside SANS SIFT to provide a complete post-incident analysis pipeline — SIFT handles digital forensics and incident reconstruction while REMnux handles malware sample dissection and behavioral analysis of artifacts recovered from compromised endpoints.

### Analysis and CTI VM (Proxmox on Dell Optiplex):

A dedicated virtual machine running on the Proxmox hypervisor hosting two enterprise-grade security platforms via Docker:

**Splunk** — Industry-leading SIEM platform deployed alongside ELK Stack to provide comparative SIEM experience and practice with Splunk Processing Language (SPL). Ingests forwarded logs from Logstash for cross-platform security analysis and dashboard building, reflecting the reality that Splunk dominates enterprise SOC environments.

**OpenCTI** — Enterprise-grade Cyber Threat Intelligence platform used by government agencies, MSSPs, and large enterprise SOC teams. Manages structured threat intelligence including IOCs, TTPs, threat actor profiles, and campaign tracking. Integrates with T-Pot honeypot telemetry, VirusTotal enrichment, and the MITRE ATT&CK framework to provide actionable intelligence that feeds back into detection rules and Wazuh alerts.

### Internal Endpoints

**Windows 10 VM:** Provides Windows-based endpoint for monitoring testing and client-side attack simulation.

**Windows Domain Controller:** Acts as central authority for authentication and Active Directory services. Runs the **Microsoft Entra Connect Sync agent** to establish outbound Password Hash Synchronization (PHS) to a cloud tenant, enabling hybrid identity tracking and User-ID firewall integration.

**Red Hat Linux:** Serves as Linux production server for SSH brute force testing and Linux security monitoring.

**K8s Node:** Provides container host environment for Kubernetes security monitoring practice.

---

## Architecture Overview

The lab splits computing workloads across a multi-host distributed topology to prevent performance bottlenecks. PC 1 (ThinkPad Laptop) runs VirtualBox to host the pfSense-1 VM, Kali Linux, and the central ELK/Wazuh SIEM stack. PC 2 (Desktop Tower) runs VirtualBox on the base OS to host the pfSense-2 VM, Windows DC, and client endpoints. A dedicated **TerraMaster F4-425 NAS** provides centralized iSCSI block storage and file services independent of any hypervisor. The **Dell Optiplex** runs a bare-metal **Proxmox hypervisor** hosting three specialized VMs — SANS SIFT for digital forensics, REMnux for malware analysis, and a Monitoring and CTI VM running Splunk and OpenCTI via Docker. All physical nodes connect via Cat6 to a Cisco Catalyst 2960CX Layer 2 switch, bundling traffic over an LACP EtherChannel trunk to a Cisco Catalyst 3560CX Layer 3 switch. Networks are dual-homed across the Cisco backplane: management and Entra ID cloud identity traffic routes over **VLAN 99 (MTU 1500)**, while the TerraMaster NAS delivers raw iSCSI block storage out-of-band over **VLAN 88 (Jumbo MTU 9000)** directly to Elasticsearch data directories. Logstash parses incoming local firewall logs, public cloud Linode T-Pot honeypot events, and Raspberry Pi 5 Suricata sensor traffic, ascending data vertically up a strict security operations hierarchy.

---

## Detection Capabilities

**Network Layer:** Port scanning detection via threshold monitoring, malware C2 via signature matching, suspicious protocols via protocol anomaly detection, DDoS patterns via rate-based monitoring, known exploits via vulnerability signatures.

**Endpoint Layer:** Malware execution via file integrity monitoring, privilege escalation via auditd, persistence via registry monitoring, lateral movement via authentication failure correlation.

**Firewall Layer:** Unauthorized access via blocked connection logs, reconnaissance via multiple blocked ports from same source, policy violations via rule match logging, VPN issues via authentication failure logs.

**Cloud Identity & Ingestion Layer:** Account compromise via anomalous Microsoft Entra ID sign-in indicators, automated brute-force containment via Shuffle SOAR orchestration, full User-ID mapping to track malicious traffic by on-premises and cloud user profiles rather than static IPs.

---

## Attack Simulations Validated

**Port Scan:** Nmap scan against pfSense WAN interface triggered Suricata "ET SCAN Nmap" alert with **4,180+ hits** in Kibana.

**SSH Brute Force:** Hydra and repeated failed SSH attempts triggered Wazuh multiple failed login alerts and pfSense NAT access logs.

**Ping Flood:** ICMP flood triggered pfSense DoS pattern detection logs.

**SMB Enumeration:** enum4linux scan against NAS storage detected and alerted in Kibana.

---

## Key Achievements

- Deployed dual pfSense firewalls with site-to-site IPsec VPN and network segmentation
- Implemented Suricata IDS/IPS on dedicated Raspberry Pi 5 with **65,796 detection rules**
- Deployed Snort IDS/IPS inline on both pfSense firewalls for internal east-west traffic inspection
- Deployed Wazuh XDR across Windows and Linux endpoints with FIM and malware detection
- Built complete ELK Stack processing **300,000+ Suricata events** and **140,000+ pfSense logs**
- Created professional Kibana dashboards for network and endpoint security visualization
- Validated **4 attack types** (port scan, SSH brute force, DoS, SMB enumeration) with full detection coverage

---

## Service Access

**Kibana Dashboard:** https://kibana.duckdns.org:5601

**Uptime Kuma:** https://status.duckdns.org:3001

**Shuffle SOAR:** https://soar.duckdns.org:3002

**pfSense-1 GUI:** https://192.168.1.40:8443

**pfSense-2 GUI:** https://192.168.1.41:8443

**Pi-hole GUI:** http://192.168.1.39:80

---

## Upcoming Enhancements

- **TerraMaster iSCSI Optimization:** Configure dedicated iSCSI LUN over isolated Storage VLAN 88 with Jumbo Frames (MTU 9000) to maximize Elasticsearch disk IOPS and reduce storage latency.
- **Three-Tier Storage Activation:** Fully activate all three storage tiers — TerraMaster iSCSI for hot Elasticsearch data, Raspberry Pi 5 Samba for warm PCAP and backup storage, and cold external archive for VM disaster recovery.
- **Physical Cisco Backplane Integration:** Implement enterprise hardware topology using Cisco Catalyst 2960CX and Cisco Catalyst 3560CX with LACP EtherChannel bundling and 802.1Q VLAN trunk isolation.
- **Out-of-Band Hardware Traffic Mirroring:** Configure hardware SPAN port on Cisco switches to mirror raw traffic directly into Raspberry Pi 5 eth0, eliminating hypervisor packet collection overhead.
- **Hybrid Cloud Identity Synchronization:** Deploy Microsoft Entra Cloud Sync agent on Windows Server Domain Controller to establish Password Hash Synchronization to Azure cloud tenant, enabling hybrid identity management.
- **Cloud-Native Threat Intelligence Node:** T-Pot Honeypot Framework on Linode VPS capturing live global attacker telemetry and shipping to local Logstash pipeline via TLS.
- **Shuffle SOAR Playbooks:** Automated incident response playbooks parsing high-severity Wazuh alerts and executing API calls for automated containment.
- **SANS SIFT Forensics Integration:** Full integration of SANS SIFT Workstation into post-incident pipeline with evidence storage on TerraMaster NAS for forensic integrity.
- **REMnux Malware Analysis Pipeline:** Integrate REMnux Workstation into the post-incident workflow — malware samples recovered by Wazuh FIM or T-Pot honeypot automatically staged for REMnux analysis, with findings documented and fed back into OpenCTI as new IOCs.
- **OpenCTI Threat Intelligence Platform:** Deploy OpenCTI via Docker on the Monitoring and CTI VM to provide structured threat intelligence management. Integrate with T-Pot honeypot telemetry, VirusTotal enrichment, and MITRE ATT&CK framework. Feed enriched IOCs back into Wazuh detection rules and Kibana dashboards for closed-loop threat intelligence.
- **Splunk SIEM Deployment:** Deploy Splunk alongside ELK Stack on the Monitoring and CTI VM for comparative enterprise SIEM experience. Forward logs from Logstash to Splunk for SPL query practice and cross-platform dashboard building reflecting real enterprise SOC tooling.
- **OpenCTI → ELK Integration:** Configure bidirectional enrichment between OpenCTI and Elasticsearch — Kibana alerts trigger OpenCTI IOC lookups, and OpenCTI threat feeds automatically create Wazuh detection rules.
- **n8n + Jira Ticketing:** Automated Jira ticket creation from Shuffle SOAR via n8n workflows for full SOC incident tracking.

---

By setting up this comprehensive enterprise-grade SOC environment, users gain hands-on experience in firewall management, IPsec VPN, SIEM operations, multi-layer IDS/IPS deployment, endpoint detection and response, SOAR orchestration, honeypot intelligence gathering, NAS storage architecture, hybrid cloud identity synchronization, malware analysis, cyber threat intelligence management, and professional DFIR workflows. The lab provides a safe, controlled environment to practice real-world security monitoring skills in a realistic multi-site enterprise context with hardware-accelerated switching and dedicated SAN storage fabric.

---

Repository Maintained by Ahmed Ashraf | Last Updated: May 2026
