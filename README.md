# SOC Home Lab — Detection & Response Simulation

![Cybersecurity](https://img.shields.io/badge/Main_Domain-Cybersecurity-red?style=for-the-badge&logo=checkpointsoftware&logoColor=white)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue?style=for-the-badge&logo=wazuh&logoColor=white)
![Splunk](https://img.shields.io/badge/Analytics-Splunk-black?style=for-the-badge&logo=splunk&logoColor=f7a100)
![pfSense](https://img.shields.io/badge/Firewall-pfSense-orange?style=for-the-badge&logo=pfsense&logoColor=white)
![License-MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

> **Project Purpose:** This lab demonstrates the design and deployment of a multi-layered SOC environment. I successfully engineered a segmented network using pfSense to isolate attacker and victim zones, configured Wazuh for host-level EDR, and integrated Splunk as a centralized SIEM to correlate network IDS alerts with endpoint telemetry.

---

## Table of Contents
- [Lab Overview](#lab-overview)
- [Network Architecture](#network-architecture)
- [Deployment Summary](#-deployment-summary)
- [Tools & Technologies](#tools--technologies)
- [Phase 1  Virtual Network Architecture & Segmentation](#phase-1--virtual-network-architecture--segmentation)
- [Phase 2  pfSense Firewall Installation & Interface Configuration](#phase-2--pfsense-firewall-installation--interface-configuration)
- [Phase 3  pfSense Post-Installation & Network Services](#phase-3--pfsense-post-installation--network-services)
- [Phase 4  SIEM & EDR Deployment](#phase-4--siem--edr-deployment)
- [Phase 5  Attack Simulation & SIEM Detection Validation](#phase-5--attack-simulation--siem-detection-validation)
- [Key Findings](#key-findings)
- [Skills Demonstrated](#skills-demonstrated)

---

## Lab Overview
| Property | Detail |
|---|---|
| **Hypervisor** | VMware Workstation |
| **Firewall / Router** | pfSense CE 2.8.1-RELEASE |
| **SIEM** | Wazuh XDR + Splunk Enterprise |
| **IDS** | Suricata (via pfSense) |
| **Attack Platform** | Kali Linux (Metasploit / Nmap) |
| **Target Endpoints** | Windows Server 2019, Windows 10, Metasploitable 2 |

---

## Network Architecture
The environment is built on four isolated Host-only VMnets to replicate a real-world enterprise network where attacker traffic is separated from management and production assets.

```mermaid
graph TD
    subgraph External_Access [WAN / Internet]
        Internet((Internet))
    end

    subgraph Firewall [pfSense Firewall]
        PF[pfSense Virtual Appliance]
    end

    subgraph Attacker_Zone [VMnet2 - Attacker Subnet]
        Kali[Kali Linux - 10.0.1.10]
    end

    subgraph Monitoring_Zone [VMnet3 - Management & SIEM]
        SIEM[Ubuntu: Wazuh + Splunk - 10.0.2.10]
    end

    subgraph Target_Zone [VMnet4 & VMnet5 - Targets]
        AD[Win Server 2019 - 10.0.3.2]
        Vuln[Metasploitable 2 - 10.0.4.2]
    end

    Internet --> PF
    PF --> Kali
    PF --> SIEM
    PF --> AD
    PF --> Vuln

Deployment Summary
Network Engineering: Provisioned four isolated VMware Host-only VMnets (VMnet2–VMnet5) to host Attacker, Monitoring, AD, and Vulnerable zones.

Gateway & Security: Deployed pfSense CE 2.8.1-RELEASE as the core router, assigning physical adapters em0–em4 to manage cross-subnet traffic and IDS monitoring.

Detection Stack: Installed Wazuh Manager and Splunk Enterprise on an Ubuntu 22.04 LTS host (10.0.2.10).

Agent Enrollment: Deployed Wazuh agents to Windows Server 2019, Windows 10, and Kali Linux to capture 360-degree telemetry.
---------------------------------------------------------------------------------------------------------------------------
Tools & Technologies
VMware Workstation: Virtualization of all lab components.

pfSense: Centralized firewall, DHCP server, and IDS platform.

Suricata: Network Intrusion Detection (IDS) engine using ET SCAN signatures.

Wazuh XDR: Host-based EDR for log collection and integrity monitoring.

Splunk Enterprise: Centralized SIEM for log analysis and correlation.

Kali Linux: Offensive testing platform.
-------------------------------------------------------------------------------------------------------------------------
Phase 1  Virtual Network Architecture & Segmentation
Assigned four dedicated Host-only adapters (VMnet2 through VMnet5).

Established a consistent /24 subnet mask across all segments.

Configured the pfSense firewall as the gateway for each segment.

Phase 2 — pfSense Firewall Installation & Interface Configuration
Configured the WAN (DHCP) on em0.

Mapped internal interfaces:

em1 -> LAN (10.0.1.1)

em2 -> OPT1 (10.0.2.1)

em3 -> OPT2 (10.0.3.1)

em4 -> OPT3 (10.0.4.1)

Phase 3 — pfSense Post-Installation & Network Services
Configured DHCP pools for each subnet.

Established firewall rules to allow management traffic from the Monitoring subnet while restricting the Attacker subnet.

Phase 4 — SIEM & EDR Deployment
Deployed Ubuntu 22.04 and executed the Wazuh-Splunk integration.

Installed Wazuh agents on all endpoints and confirmed log ingestion in the Splunk dashboard.

## Phase 5 — Attack Simulation & SIEM Detection Validation

This phase validates the effectiveness of the defensive stack by executing common attack vectors and verifying the telemetry captured by the SIEM.

### 🛡️ Detection & Response Matrix
| Attack Phase | Tool Used | Technique | Detection Source | SIEM Evidence / Alert Triggered |
| :--- | :--- | :--- | :--- | :--- |
| **Reconnaissance** | Nmap | Network Scanning | **Suricata** | `ET SCAN Possible Nmap User-Agent Observed` |
| **Enumeration** | Nmap | OS Fingerprinting | **Suricata** | `SMB Malformed Request Dialects` |
| **Credential Access** | Metasploit | SMB Brute Force | **Wazuh (EDR)** | `Rule 60106`: Windows Logon Failure (Multiple) |
| **Lateral Movement** | Metasploit | SMB Login Success | **Wazuh (EDR)** | `Rule 60107`: Windows Logon Success |
| **Exfiltration/Command** | Meterpreter | Reverse TCP Shell | **pfSense Logs** | Unusual NTLM Session Setup / Outbound Traffic |

---

### 1. Network Intrusion Detection (Suricata)
The IDS captured multiple "Possible Nmap" alerts from the Kali attacker (`10.0.1.10`) targeting the Windows Server (`10.0.3.2`). 

**Visual Proof:**
![Suricata Alerts](screenshots/suricata-alerts.png) 
*(Note: Upload your Suricata alert screenshot to the screenshots folder and name it 'suricata-alerts.png')*

### 2. Brute-Force Execution (Metasploit)
I utilized the Metasploit Framework's auxiliary/scanner/smb/smb_login` module to simulate a high-speed password-guessing attack against the Windows Server's SMB service.
- **Wordlist Used:** `fasttrack.txt`
- **Target:** Administrator Account
- **Result:** Hundreds of `FAILED LOGIN` attempts captured before a lockout or successful guess.

### 3. Centralized SIEM Analysis (Splunk)
The "Single Pane of Glass" in Splunk unified the network and host-based alerts into a single timeline.
- **Host Context:** Wazuh alerts were forwarded into Splunk, identifying specific Windows Event IDs.
- **Unified Visibility:** By filtering for `host=10.0.2.10`, the dashboard displayed host-level security telemetry alongside firewall logs.

2. Brute-Force Attack Execution (Metasploit)
Utilized Metasploit smb_login module to target the Windows Server (10.0.3.2).

Simulated 100+ failed login attempts before a successful credential guess.

Key Findings
360-Degree Visibility: Proved that capturing network-level (Suricata) and host-level (Wazuh) data provides a richer security picture than either tool alone.

Network Isolation: Verified that pfSense successfully contained attack traffic within the intended subnets.

Skills Demonstrated
Virtual network design and subnet segmentation.

Firewall deployment and rule-based access control.

SIEM/EDR integration and log aggregation.

Offensive security simulation and threat detection validation.
