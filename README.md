# SOC Home Lab — Detection & Response Simulation

![Cybersecurity](https://img.shields.io/badge/Main_Domain-Cybersecurity-red?style=for-the-badge)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue?style=for-the-badge)
![Splunk](https://img.shields.io/badge/Analytics-Splunk-black?style=for-the-badge)
![pfSense](https://img.shields.io/badge/Firewall-pfSense-orange?style=for-the-badge)

> **Project Purpose:** This lab demonstrates the design and deployment of a multi-layered SOC environment. I successfully engineered a segmented network using pfSense to isolate attacker and victim zones, configured Wazuh for host-level EDR, and integrated Splunk as a centralized SIEM to correlate network IDS alerts with endpoint telemetry.

---

## 🛠️ Deployment Summary
1. **Network Engineering:** Provisioned four isolated VMware Host-only VMnets (VMnet2–VMnet5) to host Attacker, Monitoring, AD, and Vulnerable zones.
2. **Gateway & Security:** Deployed pfSense CE 2.8.1-RELEASE as the core router, assigning physical adapters em0–em4 to manage cross-subnet traffic and IDS monitoring.
3. **Detection Stack:** Installed Wazuh Manager and Splunk Enterprise on an Ubuntu 22.04 LTS host (10.0.2.10).
4. **Agent Enrollment:** Deployed Wazuh agents to Windows Server 2019, Windows 10, and Kali Linux to capture 360-degree telemetry.

---

## 📊 Phase 5 — Attack Simulation & SIEM Detection Validation

| Attack Phase | Tool Used | Detection Source | SIEM Evidence / Alert |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | Nmap | **Suricata** | `Possible Nmap User-Agent` |
| **Fingerprinting** | Nmap | **Suricata** | `SMB Malformed Request` |
| **Brute Force** | Metasploit | **Wazuh (EDR)** | `Rule 60106`: Logon Failure |
| **Lateral Movement** | Metasploit | **Wazuh (EDR)** | `Rule 60107`: Logon Success |
| **Exfiltration** | Meterpreter | **pfSense Logs** | Unusual Outbound Traffic |

---

## 🏗️ Network Architecture
The environment is built on four isolated Host-only VMnets to replicate a real-world enterprise network.

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
    Internet --> PF
    PF --> Kali
    PF --> SIEM
