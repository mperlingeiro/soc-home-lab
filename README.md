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
