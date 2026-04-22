# SOC Home Lab — Detection & Response Simulation

![Cybersecurity](https://img.shields.io/badge/Main_Domain-Cybersecurity-red?style=for-the-badge)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue?style=for-the-badge)
![Splunk](https://img.shields.io/badge/Analytics-Splunk-black?style=for-the-badge)
![pfSense](https://img.shields.io/badge/Firewall-pfSense-orange?style=for-the-badge)

> **Project Purpose:** This lab demonstrates the design and deployment of a multi-layered SOC environment. I successfully engineered a segmented network using pfSense to isolate attacker and victim zones, configured Wazuh for host-level EDR, and integrated Splunk as a centralized SIEM to correlate network IDS alerts with endpoint telemetry.

---

## Table of Contents
- [Lab Overview](#lab-overview)
- [Network Architecture](#network-architecture)
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
| **Attack Platform** | Kali GNU/Linux 2025.4 |
| **Target** | Metasploitable 2 |
| **Domain Controller** | Windows Server 2019 |
| **Workstation** | Windows 10 Enterprise |
| **Monitoring Host** | Ubuntu 22.04 LTS (Wazuh Manager + Splunk) |

---

## Network Architecture

The lab is divided into four isolated VMware Host-only subnets, all routed through a pfSense firewall. This design ensures offensive activity is safely contained and cannot leak into the host network, while still allowing realistic east-west traffic between attacker and target zones.

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
