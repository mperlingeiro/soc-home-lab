# SOC Home Lab

A personal cybersecurity home lab built to practice SOC analyst work — covering red team / blue team exercises, SIEM monitoring, EDR detection, and threat hunting. Built from scratch on bare metal with enterprise-grade tooling.

---

## Overview

This lab simulates a real SOC environment with a dedicated attacker subnet, a victim subnet, and a full detection stack. The goal is hands-on practice: attack with Kali, detect with Splunk, Wazuh, LimaCharlie, and Microsoft Sentinel, then investigate the alerts.

---

## Phase Status

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 1 | Infrastructure build — hardware, VLANs, firewall, VMs, detection stack | ✅ Complete |
| Phase 2 | Red team exercises — Metasploit attacks on Metasploitable 3 targets, full SIEM detection and alert investigation | 🔄 In Progress |
| Phase 3 | AI-driven attack automation and SOC workflow enhancement | 📋 Planned |

---

## Physical Hardware

| Device | Specs | Role |
|--------|-------|------|
| GMKtec M6 Ultra | Ryzen 7 7640HS, 16GB DDR5, 512GB SSD, Dual 2.5GbE NIC | Proxmox host — pfSense VM + Mint jumpbox |
| Beelink SER5 Max | Ryzen 7 6800U, 32GB LPDDR5, 1TB NVMe PCIe 4.0 | Proxmox host — all lab VMs |
| TP-Link TL-SG108E | Managed Switch | VLAN trunk between both hosts |

**Internet path:** Aussie Broadband → NetComm modem → Orbi Router → Orbi Satellite → GMKtec WAN

---

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

---

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

---

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
| Metasploitable3 Ubuntu | ✓ | ✓ | ✓ | ✓ |
| Metasploitable3 Win2008 | ✓ | ✓ | ✓ | — |
| Windows 10 | ✓ | ✓ | ✓ | ✓ |
| Ubuntu LTS | ✓ | ✓ | ✓ | ✓ |

---

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

---

## Attack Exercises (Phase 2)

> Documentation will be added as exercises are completed.

Each exercise will follow this format:

- **Objective** — what is being tested
- **Attack** — tools and commands used on Kali
- **Detection** — what each SIEM platform caught and how
- **Investigation** — alert triage walkthrough
- **Remediation** — how the vulnerability could be addressed

---

## Tools & Technologies

- **Hypervisor:** Proxmox VE
- **Firewall:** pfSense
- **Attacker:** Kali Linux, Metasploit Framework
- **Targets:** Metasploitable 3, Windows 10
- **SIEM:** Splunk, Wazuh, Microsoft Sentinel
- **EDR:** LimaCharlie, Wazuh
- **Switching:** TP-Link TL-SG108E (VLAN-aware)

---

## Author

Built and maintained by Marcelo Perlingeiro as a hands-on cybersecurity learning environment.

- LinkedIn: [linkedin.com/in/marceloperlingeiro](https://linkedin.com/in/marceloperlingeiro)
