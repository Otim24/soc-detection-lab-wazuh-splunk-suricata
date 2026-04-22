# SOC Home Lab — Detection & Response Simulation

![Cybersecurity](https://img.shields.io/badge/Main_Domain-Cybersecurity-red?style=for-the-badge)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue?style=for-the-badge)
![Splunk](https://img.shields.io/badge/Analytics-Splunk-black?style=for-the-badge)
![pfSense](https://img.shields.io/badge/Firewall-pfSense-orange?style=for-the-badge)

> **Project Purpose:** This lab demonstrates the design and deployment of a multi‑layered SOC environment. I successfully engineered a segmented network using pfSense to isolate attacker and victim zones, configured Wazuh for host‑level EDR, and integrated Splunk as a centralized SIEM to correlate network IDS alerts with endpoint telemetry.

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

The lab is divided into four isolated VMware Host‑only subnets, all routed through a pfSense firewall. This design ensures offensive activity is safely contained and cannot leak into the host network, while still allowing realistic east‑west traffic between attacker and target zones.



### Subnet Map

| VMware Switch | CIDR | pfSense Interface | Purpose | Key Hosts |
|---|---|---|---|---|
| VMnet2 | 10.0.1.0/24 | LAN (em1) | Attacker Subnet | Kali Linux — 10.0.1.10 |
| VMnet3 | 10.0.2.0/24 | OPT1 — MONITORINGMACHINE (em2) | Monitoring Subnet | Ubuntu 22.04 LTS — 10.0.2.10 |
| VMnet4 | 10.0.3.0/24 | OPT2 — ACTIVEDIRECTORY (em3) | AD Subnet | Windows Server 2019 — 10.0.3.2, Windows 10 — 10.0.3.101 |
| VMnet5 | 10.0.4.0/24 | OPT3 — VULNERABLE (em4) | Vulnerable Subnet | Metasploitable 2 |

---

## Tools & Technologies

| Tool | Version | Role |
|---|---|---|
| **pfSense CE** | 2.8.1-RELEASE | Central firewall, routing, DHCP, Suricata host |
| **Wazuh** | Latest | XDR/SIEM — agent‑based endpoint detection & response |
| **Splunk Enterprise** | Latest | Log aggregation, syslog ingestion, unified alerting |
| **Suricata** | Via pfSense | Network‑based IDS, ET SCAN signature ruleset |
| **Metasploit Framework** | Latest | SMB brute‑force + exploitation simulation |
| **Nmap** | Latest | Network reconnaissance and port scanning |
| **Active Directory DS** | Windows Server 2019 | Domain environment for enterprise simulation |
| **Kali GNU/Linux** | 2025.4 | Attacker platform |
| **Metasploitable 2** | — | Intentionally vulnerable Linux target |
| **Ubuntu** | 22.04 LTS | Monitoring host — Wazuh Manager + Splunk indexer |

---

## Phase 1 — Virtual Network Architecture & Segmentation

**Objective:** Design and implement a layered network architecture using isolated virtual switches to replicate an enterprise‑grade environment.

VMware Host‑only networking was used to create four private, internal subnets. This ensures all offensive techniques are safely contained within the lab and cannot leak into the host network.

Each subnet was assigned a dedicated Host‑only adapter (VMnet2–VMnet5) to create isolated communication paths between functional roles. A consistent /24 subnet mask was applied across all segments, supporting up to 254 hosts per subnet while enforcing strict boundaries between attacker and target zones.

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![Network Adaptors](screenshots/01-network-setup/network-adaptors-assigned.png) | VMware Host‑only virtual network adaptors assigned (VMnet2–VMnet5) |
| ![Subnet Config](screenshots/01-network-setup/vmware-subnet-config.png) | VMware subnet configuration |

---

## Phase 2 — pfSense Firewall Installation & Interface Configuration

**Objective:** Deploy and configure a central firewall as the primary router and security enforcement point.

pfSense CE 2.8.1-RELEASE was installed on a dedicated VM using the ZFS file system with a GPT partition scheme for data integrity. The da0 virtual disk was used for the system OS.

### Interface Assignment

| Interface | Physical | IP Assignment | Role |
|---|---|---|---|
| WAN | em0 | 192.168.202.138/24 | External — connects lab to internet via host |
| LAN | em1 | 10.0.1.1/24 | Attacker gateway — default gateway for Kali subnet |
| OPT1 (MONITORINGMACHINE) | em2 | 10.0.2.1/24 | Monitoring gateway — Wazuh/Splunk subnet |
| OPT2 (ACTIVEDIRECTORY) | em3 | 10.0.3.1/24 | AD gateway — Windows Server/client subnet |
| OPT3 (VULNERABLE) | em4 | 10.0.4.1/24 | Vulnerable machine gateway |

The pfSense web configurator is accessible via `https://10.0.1.1/` from any machine on the LAN subnet.

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![pfSense Menu](screenshots/01-network-setup/pfsense-menu.png) | pfSense console menu post‑boot |
| ![pfSense Interfaces](screenshots/01-network-setup/pfsense-interfaces-selected.png) | Interface assignment — em0 through em4 |
| ![pfSense GUI](screenshots/01-network-setup/pfsense-gui.png) | pfSense web configurator |
| ![pfSense Dashboard](screenshots/01-network-setup/pfsense-dashboard.png) | Dashboard — all five interfaces active |
| ![Interface IP 1](screenshots/01-network-setup/interface-ip-1.png) | Interface static IP configuration |
| ![Interface IP 2](screenshots/01-network-setup/interface-ip-2.png) | Interface IP configuration (continued) |
| ![Interface IP 3](screenshots/01-network-setup/interface-ip-3.png) | Interface IP configuration (continued) |

---

## Phase 3 — pfSense Post-Installation & Network Services

**Objective:** Configure logical interfaces, DHCP services, and firewall rules to facilitate automated IP management and baseline connectivity.

### DHCP Configuration

| Interface | Subnet | DHCP Pool |
|---|---|---|
| MONITORINGMACHINE | 10.0.2.0/24 | 10.0.2.10 – 10.0.2.50 |
| ACTIVEDIRECTORY | 10.0.3.0/24 | 10.0.3.10 – 10.0.3.50 |
| VULNERABLE | 10.0.4.0/24 | 10.0.4.10 – 10.0.4.50 |

### Baseline Firewall Policy

A default‑allow policy was initially applied to LAN and MONITORINGMACHINE interfaces to verify connectivity before hardening:

- **Anti‑Lockout Rule** — preserves web GUI access at `10.0.1.1` from the LAN at all times
- **Subnet Communication Rules** — IPv4 "Any" rules allow the Monitoring machine and Kali to reach target subnets for scanning and monitoring activity

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![DHCP - AD](screenshots/02-active-directory/dhcp-server-for-active-directory.png) | DHCP scope — Active Directory subnet |
| ![DHCP - Monitor](screenshots/02-active-directory/dhcp-server-for-monitoring-machine.png) | DHCP scope — Monitoring subnet |
| ![DHCP - Vulnerable](screenshots/02-active-directory/dhcp-server-for-vulnerable-machine.png) | DHCP scope — Vulnerable subnet |
| ![Rules - LAN](screenshots/01-network-setup/rules-for-lan.png) | Firewall rules — LAN (Attacker) subnet |
| ![Rules - AD](screenshots/01-network-setup/rules-for-active-directory.png) | Firewall rules — Active Directory subnet |
| ![Rules - Monitor](screenshots/01-network-setup/rules-for-monitoring-machine.png) | Firewall rules — Monitoring subnet |
| ![Rules - Vulnerable](screenshots/01-network-setup/rules-for-vulnerable-subnet.png) | Firewall rules — Vulnerable subnet |
| ![Ping Firewall](screenshots/01-network-setup/ping-firewall.png) | Connectivity test — pinging pfSense gateway |
| ![Connectivity Check](screenshots/01-network-setup/connectivity-check.png) | Inter‑subnet connectivity verified |

### Active Directory & Windows Endpoints

Windows Server 2019 was configured as the Domain Controller with AD DS and DHCP roles. A Windows 10 Enterprise workstation was joined to the domain to simulate a realistic enterprise endpoint.

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![Windows Server 2019](screenshots/02-active-directory/windows-server-2019.png) | Windows Server 2019 VM |
| ![Computer Name Change](screenshots/02-active-directory/windows-server-computer-name-change.png) | Server rename prior to domain promotion |
| ![Static IP Setup](screenshots/02-active-directory/static-ip-and-dns-setup.png) | Static IP and DNS configuration |
| ![CMD ipconfig](screenshots/02-active-directory/cmd-windows-server.png) | IP verification via ipconfig |
| ![Windows 10 Client](screenshots/02-active-directory/windows-10-client.png) | Windows 10 Enterprise — domain‑joined workstation |

---

## Phase 4 — SIEM & EDR Deployment

**Objective:** Establish a centralised command centre for security telemetry and endpoint visibility across the entire lab environment.

### Ubuntu Monitoring Host

Ubuntu 22.04 LTS was assigned static IP `10.0.2.10/24` as a fixed destination for all incoming syslog and agent data. Connectivity to the pfSense gateway (`10.0.2.1`) was confirmed via ping before tool deployment.

### Wazuh XDR — Agent Registration

Agents were deployed across three distinct operating systems, providing visibility across attacker, domain controller, and workstation roles simultaneously:

| Agent ID | Hostname | IP Address | OS | Role |
|---|---|---|---|---|
| 002 | WinServer | 10.0.3.2 | Windows Server 2019 | Target / Domain Controller |
| 004 | Client01 | 10.0.3.101 | Windows 10 Enterprise | Target / Workstation |
| 006 | KALIATTACKER | 10.0.1.10 | Kali GNU/Linux 2025.4 | Attacker Machine |

### Splunk — Log Aggregation

Splunk Enterprise was installed on the same Ubuntu host and configured to ingest pfSense syslog data via UDP. This unified network‑level firewall logs with host‑level Wazuh telemetry into a single searchable index.

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![Ubuntu Connectivity](screenshots/02-active-directory/ubuntu-communication.png) | Ubuntu host — static IP and gateway connectivity confirmed |
| ![Wazuh Install](screenshots/03-wazuh/wazuh-download-and-installation.png) | Wazuh Manager installation on Ubuntu 22.04 |
| ![Wazuh Agent Windows](screenshots/03-wazuh/wazuh-install-on-windows-server.png) | Wazuh agent deployed to Windows Server 2019 |
| ![Wazuh Dashboard](screenshots/03-wazuh/wazuh-dashboard.png) | Wazuh XDR main dashboard |
| ![Active Agents](screenshots/03-wazuh/active-wazuh-devices.png) | All three agents active and reporting |
| ![Windows Agent](screenshots/03-wazuh/windows-server-agent-on-wazuh.png) | WinServer registered — Agent ID 002 |
| ![Splunk Install](screenshots/04-splunk/installing-splunk.png) | Splunk Enterprise installation |
| ![Splunk Input](screenshots/04-splunk/splunk-input-settings.png) | UDP syslog data input configured |
| ![Splunk Review](screenshots/04-splunk/splunk-review.png) | Input review before saving |
| ![Splunk Dashboard](screenshots/04-splunk/splunk-dashboard.png) | Splunk main dashboard |
| ![Splunk Syslogs](screenshots/04-splunk/splunk-syslogs.png) | pfSense syslog events ingested |

---

## Phase 5 — Attack Simulation & SIEM Detection Validation

**Objective:** Execute a simulated attack chain and verify that the integrated security stack — Suricata, Wazuh, and Splunk — effectively detects and logs every stage of malicious activity.

### Attack Chain Summary



---

### 1. Network Intrusion Detection — Suricata

Suricata was configured on pfSense monitoring the `ACTIVEDIRECTORY` interface. Two distinct signatures fired during the simulated attack:

- **`ET SCAN Possible Nmap User-Agent Observed`** — triggered by Nmap scan traffic from Kali (10.0.1.10) targeting Windows Server (10.0.3.2)
- **SMB Malformed Request Dialects** — flagged during OS fingerprinting; a common indicator of an attacker probing the specific Windows version on a target

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![Suricata On](screenshots/05-suricata/suricata-is-on.png) | Suricata IDS enabled and running on pfSense |
| ![Suricata Alert](screenshots/05-suricata/suricata-final-nmap-detection.png) | IDS alert — ET SCAN Nmap signature triggered |

---

### 2. SMB Brute‑Force — Metasploit

| Detail | Value |
|---|---|
| **Module** | `auxiliary/scanner/smb/smb_login` |
| **Target** | 10.0.3.2 (Windows Server 2019) |
| **Wordlist** | `fasttrack.txt` |
| **Account targeted** | Administrator |
| **Result** | Hundreds of failed login attempts — passwords cycled include `sql2008`, `monkey`, `starwars` |

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![msfconsole](screenshots/06-attack-simulation/msfconsole.png) | Metasploit Framework console |
| ![MSF Module](screenshots/06-attack-simulation/msf-module-config.png) | smb_login module configured |
| ![MSF Exploit](screenshots/06-attack-simulation/msf-exploit-execution.png) | Brute‑force in progress — failed attempts visible |

---

### 3. Exploitation — Metasploitable 2

A Metasploit exploit was executed against Metasploitable 2, achieving a Meterpreter session and confirming full remote access.

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![Ping Target](screenshots/06-attack-simulation/pinging-vulnerable-machine.png) | Connectivity confirmed to Metasploitable 2 |
| ![Nmap Scan](screenshots/06-attack-simulation/nmap-scan-1.png) | Initial Nmap recon |
| ![Nmap Full](screenshots/06-attack-simulation/nmap-scan-1.1.png) | Full port enumeration output |
| ![Nmap Target](screenshots/06-attack-simulation/nmap-scan-on-vulnerable-machine.png) | Open ports and services on Metasploitable 2 |
| ![First Exploit](screenshots/06-attack-simulation/first-exploit-success.png) | Meterpreter session opened — exploitation confirmed |
| ![Exploit 1](screenshots/06-attack-simulation/exploit-1.png) | Exploitation in progress |
| ![Listener](screenshots/06-attack-simulation/creating-a-network-listener.png) | Netcat reverse listener established |
| ![Scans](screenshots/06-attack-simulation/scans.png) | Scan results summary |

---

### 4. Centralized SIEM Analysis — Splunk

Splunk served as the unified view across all detection sources:

- **Network layer:** NTLM Session Setup requests captured from the SMB brute‑force — proving visibility into the authentication handshake itself, not just the outcome
- **Host layer:** Wazuh forwarded Windows authentication events into Splunk — **Rule 60106** (Windows Logon Success) and **Rule 60137** (User Logoff) directly tied to the brute‑force activity
- **Unified correlation:** Filtering by `host=10.0.2.10` confirmed host‑level telemetry from Windows Server sitting alongside pfSense firewall logs in the same index

**Screenshots:**

| Screenshot | Description |
|---|---|
| ![Attack Events](screenshots/03-wazuh/attack-events.png) | Wazuh alerts — attack events detected |
| ![Nmap Logs Splunk](screenshots/04-splunk/nmap-log-splunk.png) | Nmap scan traffic visible in Splunk |
| ![Splunk Syslogs](screenshots/04-splunk/splunk-syslogs.png) | Unified syslog view — pfSense + Wazuh data correlated |

---

## Key Findings

Detection coverage across the full attack chain was confirmed end‑to‑end:

- **Suricata** fired before a single exploit was launched — flagging both the Nmap user‑agent and the SMB OS fingerprinting as two separate signatures, demonstrating layered network‑level detection
- **Wazuh** captured Windows authentication events (rules 60106 and 60137) generated by the SMB brute‑force, providing host‑level corroboration of the network‑level IDS alerts
- **Splunk** successfully correlated NTLM Session Setup requests from the network layer with Wazuh host telemetry — the two data sources together produce a richer picture than either does independently
- **pfSense firewall rules** enforced subnet isolation throughout — attacker traffic reached targets only as explicitly permitted by policy, and all cross‑subnet traffic remained auditable
- **Metasploitable 2 exploitation** succeeded, confirming the attack path is realistic and that detection tools had genuine live malicious traffic to respond to

---

## Skills Demonstrated

- Virtual network design and subnet segmentation using VMware Host‑only adapters
- Firewall deployment, physical interface assignment, and rule‑based access control (pfSense CE 2.8.1)
- DHCP server configuration across multiple isolated subnets
- SIEM deployment and multi‑OS agent registration — Windows Server 2019, Windows 10, Kali Linux (Wazuh XDR)
- Log aggregation, syslog ingestion, and cross‑source correlation (Splunk Enterprise)
- Network intrusion detection and signature‑based alerting (Suricata — ET SCAN ruleset)
- Active Directory administration and domain environment configuration (Windows Server 2019)
- Offensive security simulation — reconnaissance, scanning, SMB credential attack, exploitation (Nmap, Metasploit `auxiliary/scanner/smb/smb_login`, Meterpreter)
- Full attack lifecycle simulation with simultaneous detection coverage across IDS, SIEM, and EDR layers
