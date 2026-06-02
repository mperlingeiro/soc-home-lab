# Session 02 — Detection Engineering & Log Tuning

**Date:** 10–12 May 2026
**Target:** Metasploitable 3 Ubuntu (172.16.20.106)
**MITRE Technique:** T1046 — Network Service Scanning

---

## Objective

Improve Wazuh detection quality by fixing log collection gaps and write the first custom detection rule to catch port scan behaviour.

---

## What I Did

### Wazuh Discover Setup
- Configured Explore → Discover with useful columns: agent.name, rule.description, rule.level, rule.mitre.technique, rule.mitre.tactic, rule.groups
- Filtered to META-LINUX using DQL query
- Saved view as "META-LINUX - All Events" for reuse across sessions

### Log Collection Investigation
- Discovered Wazuh only collects logs explicitly configured in agent.conf — unlike Splunk which collects everything by default
- Found Apache logs were not readable by the Wazuh agent due to Linux file permissions (rw-r----- root adm)

### Permission Fix

```bash
# Identify what user Wazuh runs as
ps aux | grep wazuh

# Add wazuh user to adm group to grant log access
sudo usermod -a -G adm wazuh

# Verify the fix
sudo -u wazuh cat /var/log/apache2/error.log
```

### Log Collection Expanded

Added to agent.conf:
- /var/log/apache2/access.log (apache format)
- /var/log/apache2/error.log (apache format)
- /var/log/samba/log.smbd (syslog format)

Result: Wazuh now captures full HTTP request detail including data.srcip, data.url, and data.protocol.

### Custom Detection Rule Written

Written in Server Management → Rules → local_rules.xml:

```xml
<rule id="100002" level="12" frequency="8" timeframe="60">
  <if_matched_sid>31101</if_matched_sid>
  <same_source_ip />
  <description>Port scan detected - multiple web probes from same IP</description>
  <group>recon,port_scan,pci_dss_11.4,</group>
</rule>
```

**Logic:** If rule 31101 (web server 400 error) fires 8 or more times within 60 seconds from the same source IP, escalate to a Level 12 alert and tag as port scan behaviour.

---

## What I Found

**Before log collection fix:**
- Web server alerts showed generic descriptions only
- No attacker IP visible in Wazuh
- No URL detail or HTTP method

**After log collection fix:**
- data.srcip: 172.16.15.102 (Kali) visible in alerts
- data.url: /evox/about, /HNAP1, /drupal/ etc. visible
- data.protocol: GET, POST, PROPFIND visible
- Full nmap fingerprint visible in full_log: "Mozilla/5.0 (compatible; Nmap Scripting Engine)"

---

## Alerts Generated

**Wazuh:**
- Web server 400 error code (rule 31101) — Level 5 — fired multiple times with full Apache detail
- Web server 501 error code — Level 4
- sshd: insecure connection attempt (scan) — Level 6
- ProFTPD: FTP session opened — Level 3
- **NEW: Port scan detected — multiple web probes from same IP (rule 100002) — Level 12 — first custom rule firing**

**Splunk / Sentinel / Lima Charlie:** Not used this session.

---

## What I Learned

### Linux file permissions
- rw-r----- means owner and group can read, others cannot
- Must verify file permissions before assuming logs are being collected
- Use `ps aux | grep [service]` to identify what user a service runs as

### Wazuh vs Splunk log collection
- Splunk collects everything indiscriminately
- Wazuh only collects explicitly configured log files
- Wazuh enriches logs with MITRE tags and compliance context automatically
- Splunk shows raw detail — Wazuh shows structured context

### Wazuh XDR vs SIEM
- Wazuh is primarily XDR (endpoint focused) that also performs SIEM functions
- Enterprise equivalent of Wazuh: CrowdStrike Falcon
- Enterprise equivalent of Splunk: Splunk Enterprise or Microsoft Sentinel

### Detection rule debugging
- if_matched_sid requires the parent rule to actually be firing before the child rule can trigger
- same_field syntax is incorrect in Wazuh 4.x — correct syntax is `<same_source_ip />`
- Always verify the parent rule is firing before debugging the child rule

---

## Detection Gaps Addressed

- Fixed: nmap scan now generates a Level 12 alert
- Fixed: Apache logs now collected with full request detail
- Pending: Samba logs empty — needs tuning
- Pending: MySQL logs empty — low priority
- Pending: Splunk forwarder stability on META-LINUX
- Pending: Sentinel and Lima Charlie not checked

---

## Next Session Options

1. Exploit UnrealIRCd backdoor on port 6697 — first Metasploit shell, rich telemetry expected
2. Check Sentinel and Lima Charlie
3. Fix Splunk forwarder on META-LINUX
4. Write brute force detection rule for SSH

---

## Skills Demonstrated

Detection engineering, custom rule writing, Linux permissions, log collection configuration, Wazuh tuning, MITRE ATT&CK mapping
