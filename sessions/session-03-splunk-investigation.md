# Session 03 — Splunk Investigation Workflow

**Date:** 14 May 2026
**Target:** Metasploitable 3 Ubuntu (172.16.20.106)
**Goal:** Configure Splunk properly and practice a full SOC analyst investigation workflow from scratch

---

## Objective

Fix Splunk sourcetype configuration so logs are properly parsed, then conduct a structured investigation of suspicious activity using only Splunk — simulating a real SOC analyst workflow.

---

## What I Did

### Splunk Sourcetype Fix

Discovered all logs were ingesting as generic syslog — no source IP fields were being parsed, which made the data useless for investigation.

Edited inputs.conf to assign correct sourcetypes:

| Log File | Sourcetype Assigned |
|----------|-------------------|
| Apache access log | access_combined |
| Apache error log | apache_error |
| auth.log | linux_secure |
| syslog + kern.log | syslog |

Also discovered a hostname inconsistency across the environment:
- metasploitable3-ubuntu (inputs.conf)
- 172.16.20.106 (old hostname)
- ubuntu (syslog hostname)

All three hostnames must be included in search queries to get complete data.

---

## Investigation Workflow

Learned and applied the correct SOC analyst investigation sequence:

1. **WHAT happened** — `stats count by sourcetype`
2. **WHEN did it happen** — `timechart span=1d/1h`
3. **WHO did it** — `stats count by clientip`
4. **WHAT exactly** — `stats count by uri, status`

---

## Full Investigation Walkthrough

### Step 1 — 7-day trend (big picture)

| Date | Activity |
|------|----------|
| May 10 | First activity — linux_secure + syslog only |
| May 12 | Massive spike — 11,527 web requests |
| May 14 | 1,449 web requests + 846 syslog |

Finding: Clear anomaly on May 12. Normal baseline is near zero.

### Step 2 — Which services were hit (May 14)

| Sourcetype | Count | Meaning |
|------------|-------|---------|
| access_combined | 1,449 | Apache — highest volume |
| syslog | 846 | FTP and system events |
| pfsense_logs | 79 | Firewall — bonus discovery |
| linux_secure | 39 | SSH and auth |

### Step 3 — Suspicious IP identified

172.16.15.102 generating multiple HTTP status codes: 200, 404, 405, 501

Multiple status codes from a single IP = automated scanner behaviour. Normal users only get 200.

### Step 4 — URLs requested by suspicious IP

| URL | Meaning |
|-----|---------|
| /nmaplowercheck[random] | nmap fingerprint — confirms automated scanner |
| /.git/HEAD | Source code exposure probe |
| /HNAP1, /evox/about | Device fingerprinting |
| /drupal/ | 200 OK — application mapped |
| /phpmyadmin/ | 200 OK — application mapped |
| /chat/ | 200 OK — application mapped |

Finding: Attacker successfully mapped all web applications running on the target.

---

## Key Splunk Queries

**Big picture — what sourcetypes are present:**
```
index=* (host="metasploitable3-ubuntu" OR host="172.16.20.106" OR host="ubuntu") earliest=-24h 
| stats count by sourcetype
```

**Daily trend — spot anomalies:**
```
index=* (host="metasploitable3-ubuntu" OR host="172.16.20.106" OR host="ubuntu") earliest=-7d 
| timechart span=1d count by sourcetype
```

**Hourly breakdown — narrow the time window:**
```
index=* (host="metasploitable3-ubuntu" OR host="172.16.20.106" OR host="ubuntu") earliest=-24h 
| timechart span=1h count by sourcetype
```

**Which services were hit:**
```
index=* (host="metasploitable3-ubuntu" OR host="172.16.20.106" OR host="ubuntu") earliest=-24h 
| stats count by sourcetype, source
```

**What did a suspicious IP request:**
```
index=* (host="metasploitable3-ubuntu" OR host="172.16.20.106" OR host="ubuntu") 
sourcetype=access_combined clientip="172.16.15.102" earliest=-24h 
| stats count by uri, status
```

**Full IP search across all sourcetypes:**
```
index=* (host="metasploitable3-ubuntu" OR host="172.16.20.106" OR host="ubuntu") 
"172.16.15.102" earliest=-24h 
| stats count by sourcetype
```

---

## Incident Finding

```
SUSPICIOUS ACTIVITY DETECTED

Source IP:  172.16.15.102
Time:       05:00 window, 14 May 2026
Activity:   Automated web scanner

Evidence:
- /nmaplowercheck — nmap fingerprint confirmed
- /.git/HEAD — source code exposure probe
- /HNAP1, /evox/about — device fingerprinting
- Successfully mapped: /chat, /drupal, /phpmyadmin (all returned 200 OK)

Assessment:  Automated reconnaissance performed against web services
Risk:        High — attacker has mapped all web applications on the target
```

---

## What I Learned

**Sourcetypes are critical** — without correct sourcetypes Splunk cannot parse fields. Generic syslog means no clientip, no url, no structured data to search.

**Never start with raw logs** — use stats and timechart to summarise first. Raw logs come at the end to confirm findings, not to start them.

**Baseline before anomaly** — a count of 1,449 events means nothing in isolation. Compared to zero on surrounding days, it is a clear anomaly.

**Wazuh and Splunk work together** — Wazuh fires the Level 12 alert and tells you where to look. Splunk does the detective work and tells you exactly what happened.

**Multiple status codes = scanner** — normal users get 200. Getting 200, 404, 405, and 501 from the same IP means an automated tool.

**nmap leaves fingerprints everywhere:**
- /nmaplowercheck in Apache logs
- NmapNSE_1.0 in SSH logs
- NmapSSH1-Hostkey in auth logs

---

## Detection Gaps

- syslog hostname inconsistency not yet resolved
- pfsense_logs discovered but not investigated
- Lima Charlie not checked this session

---

## Next Session Options

1. Exploit UnrealIRCd — first Metasploit shell
2. Complete syslog investigation in Splunk
3. Investigate pfsense_logs — firewall perspective
4. Check Lima Charlie sensors

---

## Skills Demonstrated

Splunk investigation workflow, log parsing and sourcetype configuration, anomaly detection, incident documentation, threat hunting, multi-source correlation
