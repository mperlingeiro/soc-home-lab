# SOC Home Lab

Built from scratch on bare metal hardware. A full SOC environment for red team vs blue team exercises. I attack with Kali Linux and Metasploit, then detect, triage, and investigate those alerts across Splunk, Wazuh, LimaCharlie, and Microsoft Sentinel.

This demonstrates: network architecture, threat detection, alert triage, multi-platform SIEM operation, and incident investigation.

## Phase Status

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 1 | Infrastructure build — hardware, VLANs, firewall, VMs, detection stack | Complete |
| Phase 2 | Red team exercises — Metasploit attacks on Metasploitable 3 targets, full SIEM detection and alert investigation | In Progress |
| Phase 3 | AI-driven attack automation and SOC workflow enhancement | Planned |

## Session Journal

Real investigation write-ups from active lab sessions. Each one follows the same structure: objective, what I did, what I found, alerts generated, and what I learned.

| Session | Focus | MITRE Technique |
|---------|-------|----------------|
| [Session 01 — Reconnaissance](sessions/session-01-recon.md) | nmap scan, service enumeration, first alerts across Wazuh and Splunk | T1046 Network Service Scanning |
| [Session 02 — Detection Engineering](sessions/session-02-detection-engineering.md) | Apache log collection fix, Linux permissions, first custom Wazuh rule written and firing | T1046 Network Service Scanning |
| [Session 03 — Splunk Investigation](sessions/session-03-splunk-investigation.md) | Full SOC investigation workflow from scratch, sourcetype fix, incident finding written | T1046 Network Service Scanning |
| [Session 04 — Lima Charlie EDR](sessions/session-04-lima-charlie.md) | EDR telemetry, process tree forensics, platform comparison across Wazuh, Splunk, Lima Charlie | T1046 Network Service Scanning |

## Custom Detection Rules

| Rule | Description | File |
|------|-------------|------|
| Wazuh Rule 100002 | Port scan detection — Level 12 alert on 8+ web probes from same IP within 60 seconds | [custom-wazuh-rule-100002.xml](rules/custom-wazuh-rule-100002.xml) |

## Physical Hardware

| Device | Specs | Role |
|--------|-------|------|
| GMKtec M6 Ultra | Ryzen 7 7640HS, 16GB DDR5, 512GB SSD, Dual 2.5GbE NIC | Proxmox host — pfSense VM + Mint jumpbox |
| Beelink SER5 Max | Ryzen 7 6800U, 32GB LPDDR5, 1TB NVMe PCIe 4.0 | Proxmox host — all lab VMs |
| TP-Link TL-SG108E | Managed Switch | VLAN trunk between both hosts |

Internet path: Aussie Broadband → NetComm modem → Orbi Router → Orbi Satellite → GMKtec WAN

## Network Architecture

```
Internet
    │
    ▼
Orbi Satellite (WAN)
    │
    ▼
┌─────────────────────────────┐
│  GMKtec (Proxmox)           │
│  ┌─────────────────────┐    │
│  │ pfSense VM          │    │
│  │  WAN: 192.168.1.20  │    │
│  │  LAN: 172.16.10.1   │    │
│  │  OPT1: 172.16.15.1  │    │
│  │  OPT2: 172.16.20.1  │    │
│  └─────────────────────┘    │
│  ┌─────────────────────┐    │
│  │ Mint Jumpbox        │    │
│  │  172.16.10.111      │    │
│  └─────────────────────┘    │
└─────────────────────────────┘
    │
    ▼
TP-Link Managed Switch (VLAN Trunk)
    │
    ▼
┌─────────────────────────────────────────────────┐
│  Beelink (Proxmox)                              │
│                                                 │
│  LAN (172.16.10.x)                              │
│  ├── Splunk Server        172.16.10.13          │
│  └── Wazuh Server         172.16.10.150         │
│                                                 │
│  VLAN 10 — Red Team (172.16.15.x)               │
│  └── Kali Linux           172.16.15.102         │
│                                                 │
│  VLAN 20 — Blue Team (172.16.20.x)              │
│  ├── Ubuntu LTS           172.16.20.101         │
│  ├── Windows 10           172.16.20.105         │
│  ├── Metasploitable3 Ubuntu  172.16.20.106      │
│  └── Metasploitable3 Win2008 172.16.20.107      │
└─────────────────────────────────────────────────┘
```

## Virtual Machines

### LAN — Management Segment (172.16.10.x)

| VM | OS | IP | Role |
|----|----|----|------|
| Mint Jumpbox | Linux Mint | 172.16.10.111 | SSH pivot / management access |
| Wazuh Server | Ubuntu | 172.16.10.150 | SIEM / EDR |
| Splunk Server | Ubuntu | 172.16.10.13 | SIEM |

### VLAN 10 — Red Team (172.16.15.x)

| VM | OS | IP | Role |
|----|----|----|------|
| Kali | Kali Linux | 172.16.15.102 | Attacker node |

### VLAN 20 — Blue Team (172.16.20.x)

| VM | OS | IP | Role |
|----|----|----|------|
| Ubuntu LTS | Ubuntu | 172.16.20.101 | Target — Ubuntu server |
| Windows 10 | Windows 10 | 172.16.20.105 | Target — Windows workstation |
| Metasploitable3 Ubuntu | Ubuntu | 172.16.20.106 | Vulnerable Linux target |
| Metasploitable3 Win2008 | Windows Server 2008 R2 | 172.16.20.107 | Vulnerable Windows target |

## Detection Stack

### Tools

| Tool | Purpose |
|------|---------|
| Splunk | SIEM — log aggregation, search, dashboards |
| Wazuh | SIEM + EDR — agent-based threat detection |
| LimaCharlie | EDR — endpoint telemetry and response |
| Microsoft Sentinel | Cloud SIEM — Azure-native detection and analytics |

### Agent Coverage

| Target VM | Splunk Forwarder | Wazuh Agent | LimaCharlie | MS Sentinel |
|-----------|-----------------|-------------|-------------|-------------|
| Metasploitable3 Ubuntu | Yes | Yes | Yes | Yes |
| Metasploitable3 Win2008 | Yes | Yes | Yes | No |
| Windows 10 | Yes | Yes | Yes | Yes |
| Ubuntu LTS | Yes | Yes | Yes | Yes |

## Firewall — pfSense

**Interfaces**

| Interface | IP | Purpose |
|-----------|----|---------|
| WAN | 192.168.1.20 | Uplink to home network |
| LAN | 172.16.10.1 | Management segment |
| OPT1 | 172.16.15.1 | VLAN 10 — Red Team |
| OPT2 | 172.16.20.1 | VLAN 20 — Blue Team |

**NAT Port Forwards**

| Port | Destination | Description |
|------|-------------|-------------|
| 8000 (TCP) | 172.16.10.13:8000 | Splunk Web GUI |
| 2222 (TCP) | 172.16.10.13:22 | SSH to Splunk |
| 443 (TCP) | 172.16.10.150:443 | Wazuh Web UI |
| 5514 (UDP) | 172.16.10.150:514 | LimaCharlie to Wazuh syslog |
| 22 (TCP) | 172.16.10.111:22 | Mint jumpbox SSH tunnel |

## Tools & Technologies

- Hypervisor: Proxmox VE
- Firewall: pfSense
- Attacker: Kali Linux, Metasploit Framework
- Targets: Metasploitable 3, Windows 10
- SIEM: Splunk, Wazuh, Microsoft Sentinel
- EDR: LimaCharlie, Wazuh
- Switching: TP-Link TL-SG108E (VLAN-aware)

## What This Demonstrates

- Network architecture and segmentation design
- Threat detection across multiple SIEM and EDR platforms
- Alert triage and incident investigation
- Detection engineering and custom rule writing
- Red team vs blue team exercise documentation
- Incident finding and recommendation writing

## Author

Built and maintained by Marcelo Perlingeiro.

- LinkedIn: [linkedin.com/in/marceloperlingeiro](https://linkedin.com/in/mperlingeiro)
