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
- [Phase 1 — Virtual Network Architecture & Segmentation](#phase-1--virtual-network-architecture--segmentation)
- [Phase 2 — pfSense Firewall Installation & Interface Configuration](#phase-2--pfsense-firewall-installation--interface-configuration)
- [Phase 3 — pfSense Post-Installation & Network Services](#phase-3--pfsense-post-installation--network-services)
- [Phase 4 — SIEM & EDR Deployment](#phase-4--siem--edr-deployment)
- [Phase 5 — Attack Simulation & SIEM Detection Validation](#phase-5--attack-simulation--siem-detection-validation)
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
