# Session 04 — Lima Charlie EDR Introduction

**Date:** 15 May 2026
**Target:** Metasploitable 3 Ubuntu (172.16.20.106)
**Goal:** Explore Lima Charlie EDR and compare endpoint visibility with Wazuh and Splunk

---

## Objective

Get hands-on with Lima Charlie for the first time. Understand what EDR telemetry looks like at the process level during an active nmap scan, and compare it against what Wazuh and Splunk capture.

---

## What I Did

- Confirmed all 4 sensors installed across the lab in Lima Charlie
- Explored the META-LINUX sensor interface
- Ran an nmap scan and watched the Live Feed in real time
- Exported and analysed Timeline data
- Attempted the Network and Processes tabs

---

## Lima Charlie Interface

| Tab | Purpose |
|-----|---------|
| Live Feed | Real-time stream of all endpoint events |
| Timeline | Historical searchable event log |
| Processes | Visual process tree — who spawned what |
| Network | All active network connections |
| Detections | Alerts from D&R rules |
| Console | Run commands remotely on the sensor |
| File System | Browse machine files remotely |
| Event Collection | Configure which events are collected |

---

## What Lima Charlie Detected During nmap Scan

### Live Feed (real time)
- NETWORK_CONNECTIONS — burst of simultaneous connections during scan
- NEW_PROCESS — services spawning to handle nmap probes
- TERMINATE_PROCESS — processes dying after probe was rejected

### Timeline (00:42–00:45 UTC)
- 135 simultaneous connections from 172.16.15.102 hitting ports 22, 80, 445, 631, 3306, 3500, 6697, 8080 — the complete nmap scan visible in a single snapshot
- SSH process tree: 7 sshd children spawned and killed within 10 seconds — classic scan signature
- sshd: [accepted] process at 00:43:01 — connection briefly accepted before being rejected

### Suspicious Processes Discovered (No Attack Required)

These were found during normal inspection — no exploitation needed:

| Process | User | Risk |
|---------|------|------|
| nodejs /opt/chatbot/papa_smurf/chat_client.js | root (UID 0) | Critical — if exploited, attacker gets immediate root |
| ruby2.3 bin/rails (port 3500) | chewbacca | Intentional Metasploitable account |
| java Jetty (port 8080) | root (UID 0) | Java 6 (EOL 2013) running as root |

---

## Key Concepts Learned

**USER_ID 0 = root**
Every Lima Charlie process shows USER_ID and USER_NAME. USER_ID 0 is always root on Linux. papa_smurf running as root means any successful exploit gives the attacker immediate full machine control.

**Process tree forensics**
Lima Charlie shows PARENT_PROCESS_ID for every event, allowing full reconstruction of who spawned what:
```
PID 1 (init) → PID 1201 (sshd -D) → PID 2011 (sshd -D -R)
```
The -R flag indicates the process was spawned to handle an incoming connection.

**EDR vs SIEM vs XDR — in practice**

| Tool | What it showed |
|------|---------------|
| Wazuh (XDR) | "SSH scan detected" — high-level alert |
| Splunk (SIEM) | Raw log with NmapSSH fingerprint visible |
| Lima Charlie (EDR) | 7 sshd processes spawned and killed in 10 seconds — process-level behaviour |

Each platform shows a different layer of the same event. All three together give the complete picture.

---

## Lima Charlie vs CrowdStrike

Lima Charlie is powerful but requires manual configuration — D&R rules must be written from scratch. CrowdStrike is more polished and enterprise-ready out of the box. Lima Charlie is more educational because you build the detection logic yourself rather than relying on vendor defaults.

---

## Limitations Encountered

- Ubuntu 14.04 too old for full Lima Charlie support
- Network tab not loading — NETSTAT_REP not delivered
- Processes tab only showing kernel processes
- Detections tab empty — no D&R rules configured yet
- Port scan does not trigger default Lima Charlie detections

---

## Detection Gaps

- No D&R rules written yet for Lima Charlie
- papa_smurf running as root — no alert fired
- Port scan generated no Lima Charlie detection
- Network tab unavailable on Ubuntu 14.04

---

## Next Session Options

1. Exploit UnrealIRCd — first Metasploit shell, watch Lima Charlie on a modern OS
2. Write first Lima Charlie D&R rule
3. Test Lima Charlie on Windows 10 or Ubuntu LTS where full support is available
4. Write D&R rule to detect privileged Node.js processes (papa_smurf running as root)

---

## Skills Demonstrated

EDR telemetry analysis, process tree forensics, endpoint visibility, multi-platform SIEM/EDR comparison, detection gap identification, privilege escalation risk assessment
