# Session 01 — Reconnaissance

**Date:** 10 May 2026
**Target:** Metasploitable 3 Ubuntu (172.16.20.106)
**MITRE Technique:** T1046 — Network Service Scanning

---

## Objective

Map the attack surface of Metasploitable 3 Ubuntu. Identify open ports, running services, and software versions before attempting any exploitation.

---

## What I Did

Ran a ping sweep across the lab subnet to confirm which hosts were live, then followed up with a full service and version scan against the target.

---

## Commands Used

```bash
# Ping sweep — identify live hosts, no port scan
sudo nmap -sn 172.16.20.0/24

# Full scan — version detection, default scripts, all ports
nmap -sV -sC -p- 172.16.20.106
```

---

## What I Found

**Ping sweep results:**
- 172.16.20.1 — pfSense gateway
- 172.16.20.106 — Metasploitable 3 Ubuntu (target confirmed live)

**Open ports on 172.16.20.106:**

| Port | Service | Version | Notes |
|------|---------|---------|-------|
| 21 | FTP | ProFTPD 1.3.5 | Known mod_copy exploit available |
| 22 | SSH | OpenSSH | Accessible |
| 80 | HTTP | Apache | Drupal, phpMyAdmin, payroll_app.php |
| 445 | SMB | Samba | Guest access enabled, signing disabled |
| 631 | IPP | CUPS | Print service |
| 3306 | MySQL | MySQL | Exposed with no firewall restriction |
| 3500 | HTTP | Ruby on Rails (2018) | End of life |
| 6697 | IRC | UnrealIRCd | Known backdoor exploit in Metasploit |
| 8080 | HTTP | Java Jetty | Running as root |

---

## Alerts Generated

**Wazuh:**
- sshd: insecure connection attempt (scan)
- ProFTPD: FTP session opened
- ProFTPD: Attempt to login using non-existent user
- Web server 400 error code
- Web server 501 error code
- PAM: Login session closed (multiple)
- Successful sudo to ROOT executed
- Wazuh agent disconnected
- Gap: No consolidated port scan alert fired — individual service alerts required manual correlation

**Splunk:**
- 237 events captured from META-LINUX
- NmapSSH1-Hostkey and NmapNSE_1.0 visible in SSH logs — nmap fingerprint identified
- Multiple SSH preauth connections from 172.16.15.102
- FTP: USER ftp — no such user found from 172.16.15.102
- Multiple rapid Samba sessions for user nobody
- Gap: Forwarder crashes when running alongside Wazuh — machine too old to run both simultaneously

**Microsoft Sentinel:** Not available on this VM.

**Lima Charlie:** Not checked this session — flagged as next session priority.

---

## What I Learned

- nmap signs its own name in logs — NmapNSE_1.0 is visible in raw SSH logs in Splunk
- Wazuh and Splunk show the same attack from different angles — Wazuh enriches with MITRE and compliance context, Splunk shows granular raw detail
- Multiple service alerts firing simultaneously from the same source IP is the port scan signature to look for
- A single log line contains the full story: who, what, where, when, and result
- Ubuntu 14.04 is too old to run Wazuh and Splunk agents simultaneously — resource constraint to work around
- Always start VMs and restart agents before beginning a session

---

## Detection Gaps

- No consolidated port scan alert in Wazuh — requires manual correlation across individual service alerts
- Splunk forwarder unstable on META-LINUX alongside Wazuh agent
- Lima Charlie not checked

---

## Next Session

Exploit UnrealIRCd backdoor via Metasploit — easiest first shell, expected to generate rich telemetry across Wazuh and Splunk.

---

## Skills Demonstrated

Alert triage, log analysis, multi-platform SIEM monitoring, MITRE ATT&CK mapping, network reconnaissance detection
