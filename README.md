# SOC Analyst Home Lab — Wazuh Detection & Incident Response

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Windows](https://img.shields.io/badge/Endpoint-Windows%2010-blue)
![Linux](https://img.shields.io/badge/Linux-Kali%20Linux-black)
![Wireshark](https://img.shields.io/badge/Network-Wireshark-blue)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-T1110-red)

## Overview

This project documents a hands-on Security Operations Center (SOC) analyst home lab designed to practice the complete security monitoring, detection, investigation, and incident response workflow.

The lab covers:

- Network traffic analysis
- Linux log investigation
- Windows Event Log investigation
- Endpoint and process analysis
- Wazuh SIEM monitoring
- Custom detection engineering
- Alert correlation
- IOC analysis
- MITRE ATT&CK mapping
- Incident response
- Professional SOC reporting

The project follows a practical SOC workflow:

```text
Log Collection
      ↓
Detection
      ↓
Alert Triage
      ↓
Investigation
      ↓
Event Correlation
      ↓
IOC Analysis
      ↓
Incident Response
      ↓
Reporting
```

## Objectives

- Understand SOC monitoring and alert triage
- Analyze network traffic using Wireshark
- Investigate Linux authentication and system logs
- Investigate Windows Security Events
- Monitor Windows endpoints using Wazuh
- Create custom Wazuh detection rules
- Perform alert correlation
- Investigate suspicious authentication activity
- Analyze processes and network connections
- Map suspicious activity to MITRE ATT&CK
- Perform incident response and impact assessment
- Produce professional SOC incident documentation

## Lab Architecture

```text
                         ┌─────────────────────┐
                         │     Kali Linux      │
                         │ Network Analysis    │
                         │ Security Testing    │
                         └──────────┬──────────┘
                                    │
                             Isolated Lab
                                  Network
                                    │
                 ┌──────────────────┴──────────────────┐
                 │                                     │
        ┌────────▼─────────┐                  ┌────────▼─────────┐
        │    Windows 10    │                  │   Wazuh Server   │
        │                  │                  │                  │
        │  Wazuh Agent    │─────────────────▶│ Wazuh Manager    │
        │  Security Logs  │                  │ Wazuh Indexer    │
        │  Process Events │                  │ Wazuh Dashboard  │
        └──────────────────┘                  └──────────────────┘
```

### Main Components

| Component | Purpose |
|---|---|
| Wazuh | SIEM / endpoint monitoring and detection |
| Windows 10 | Endpoint monitoring and Windows Event analysis |
| Kali Linux | Security analysis and testing |
| Wireshark | Network packet analysis |
| Windows Event Logs | Authentication and process investigation |
| MITRE ATT&CK | Adversary behavior mapping |
| VMware | Virtualized lab environment |

# Day 1 — Wireshark Packet Analysis

## Focus

Network traffic analysis and packet-level investigation.

## Activities

- Inspected network packets
- Analyzed IP communication
- Examined TCP and UDP traffic
- Investigated DNS traffic
- Examined HTTP/HTTPS communication
- Identified network communication patterns

## Skills Demonstrated

- Packet analysis
- Network protocol understanding
- Traffic investigation
- Basic network threat analysis

# Day 2 — Linux Log Analysis

## Focus

Linux authentication and system log investigation.

## Activities

- Reviewed authentication events
- Investigated successful and failed authentication
- Analyzed timestamps and user activity
- Correlated related log events

## Skills Demonstrated

- Linux log analysis
- Authentication investigation
- Event correlation
- Timeline analysis

# Day 3 — Windows Event Investigation

## Focus

Windows Security Event analysis.

## Events Investigated

| Event ID | Description |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4672 | Special privileges assigned to a new logon |
| 4634 | Logoff |
| 4688 | Process creation |

## Investigation Areas

- Failed authentication
- Successful authentication
- Administrator activity
- Logon types
- Authentication packages
- Process creation
- Security event correlation

# Day 4 — Process Investigation & Alert Correlation

## Focus

Endpoint process and service investigation.

## Activities

- Investigated process IDs
- Examined parent-child process relationships
- Investigated `svchost.exe`
- Examined Windows services
- Investigated network connections
- Reviewed service installation events
- Investigated process creation events

## Investigation Result

The investigated Windows processes and services were consistent with legitimate Windows system activity.

No confirmed malicious process or persistence mechanism was identified.

# Day 5 — Wazuh Detection Engineering

## Focus

Custom detection and alert correlation.

A Windows failed authentication event was used to build a multi-stage Wazuh detection.

## Detection Chain

```text
Windows Event ID 4625
        ↓
Wazuh Rule 60122
        ↓
Custom Rule 100101
        ↓
Administrator Account Targeted
        ↓
Repeated Failed Authentication
        ↓
Custom Rule 100102
        ↓
3 Attempts Within 2 Minutes
        ↓
Level 10 Alert
```

## Custom Rule 100101

Detects failed authentication against the administrator account:

```xml
<rule id="100101" level="8">
  <if_sid>60122</if_sid>
  <field name="win.eventdata.targetUserName">^Madhusughand$</field>
  <description>Custom Detection: Failed login attempt against administrator account</description>
  <group>windows,authentication_failed,custom_detection,</group>
</rule>
```

## Custom Rule 100102

Correlates repeated failed authentication attempts:

```xml
<rule id="100102" level="10" frequency="3" timeframe="120">
  <if_matched_sid>100101</if_matched_sid>
  <description>Custom Detection: Possible brute-force attack - 3 failed logins within 2 minutes</description>
  <group>windows,authentication_failed,bruteforce,custom_detection,</group>
</rule>
```

## Detection Result

The correlation rule successfully generated a **Level 10** alert after detecting three failed authentication attempts within two minutes.

# Day 6 — Incident Response & SOC Reporting

## Incident

### Potential Brute Force / Suspicious Authentication Activity

The existing Wazuh Level 10 alert was investigated as a complete SOC incident.

## Alert Details

| Field | Value |
|---|---|
| Host | `DESKTOP-2LUO2PD` |
| Host IP | `192.168.244.130` |
| Target Account | `Madhusughand` |
| Event ID | `4625` |
| Wazuh Rule | `100102` |
| Alert Level | `10` |
| Frequency | 3 attempts |
| Timeframe | 120 seconds |
| Source IP | `127.0.0.1` |
| Logon Type | 2 — Interactive |
| Authentication | `Negotiate` |
| Process | `svchost.exe` |

## Incident Timeline

```text
Failed Authentication
        ↓
Windows Event ID 4625
        ↓
Wazuh Rule 60122
        ↓
Custom Rule 100101
        ↓
Administrator Account Targeted
        ↓
Repeated Failed Authentication
        ↓
3 Attempts Within 2 Minutes
        ↓
Custom Rule 100102
        ↓
Level 10 Alert
        ↓
SOC Investigation
        ↓
No Confirmed Compromise
```

## Investigation

The investigation examined:

- Windows authentication events
- Failed logons
- Successful logons
- Privileged logon activity
- Target account
- Logon type
- Authentication package
- Source information
- Associated process
- Windows services
- Network connections
- Endpoint activity

The associated `svchost.exe` process was investigated and found to be consistent with legitimate Windows system activity.

No evidence of malicious process execution, persistence, unauthorized privilege escalation, or data compromise was identified.

## IOC Analysis

| Indicator | Assessment |
|---|---|
| `127.0.0.1` | Localhost; not malicious by itself |
| `Madhusughand` | Administrator account targeted |
| Event `4625` | Failed authentication |
| Rule `100102` | Potential brute-force detection |
| `svchost.exe` | Legitimate Windows process |
| `C:\Windows\System32\svchost.exe` | Legitimate Windows path |
| Malicious external IP | Not identified |
| Malicious file hash | Not identified |
| Suspicious executable | Not identified |

### IOC Conclusion

No confirmed malicious IOC was identified during the investigation.

# MITRE ATT&CK Mapping

## T1110 — Brute Force

The repeated failed authentication pattern was mapped to:

**MITRE ATT&CK T1110 — Brute Force**

The mapping represents the detected authentication behavior. It does not by itself prove that an actual attacker successfully compromised the endpoint.

# Impact Assessment

## Authentication Impact

Three failed authentication attempts were detected against an administrator account within a two-minute window.

## System Impact

No evidence of:

- Unauthorized system access
- Malware execution
- Persistence
- Unauthorized privilege escalation
- Malicious service installation

was identified.

## Data Impact

No evidence of:

- Data access
- Data modification
- Data theft
- Data exfiltration

was identified.

## Overall Impact

**Low**

# Severity Assessment

| Factor | Assessment |
|---|---|
| Wazuh Alert Level | Level 10 — High |
| Suspicious Authentication | Yes |
| Successful Unauthorized Login | Not confirmed |
| Malware | Not detected |
| Persistence | Not detected |
| Data Compromise | Not detected |
| Overall Impact | Low |
| Final Classification | Potential Brute Force |
| Confirmed Compromise | No |

# Incident Response

## Containment

Recommended actions for a real production incident:

- Monitor the targeted administrator account
- Review subsequent `4625`, `4624`, and `4672` events
- Apply account lockout or temporary restriction if repeated attempts continue
- Identify and restrict the actual source if a malicious source is discovered
- Escalate if successful authentication follows repeated failures

Because this was a controlled lab investigation with no confirmed compromise, no destructive containment action was required.

## Eradication

No eradication action was required because no malicious process, persistence mechanism, or compromised account was identified.

## Recovery

Recommended actions:

- Continue endpoint monitoring
- Monitor authentication activity
- Maintain Wazuh detection rules
- Review future authentication events
- Escalate recurring suspicious activity

# Final SOC Verdict

> **True Detection — Potential Brute Force / Suspicious Authentication Activity, No Confirmed Compromise**

Wazuh successfully detected repeated failed authentication attempts against an administrator account.

The detection chain from Windows Event ID 4625 through Wazuh Rules 60122, 100101, and 100102 functioned as intended.

The associated authentication, process, and endpoint activity was investigated. No evidence of successful unauthorized access, malware execution, persistence, privilege escalation, or data compromise was identified.

The activity was generated as part of controlled security testing in the laboratory environment.

# Skills Demonstrated

- SOC alert triage
- SIEM monitoring
- Wazuh
- Windows Event Log analysis
- Linux log analysis
- Network traffic analysis
- Wireshark
- Authentication investigation
- Process investigation
- Event correlation
- Detection engineering
- Custom Wazuh rules
- Brute-force detection
- IOC analysis
- MITRE ATT&CK
- Incident response
- Incident documentation
- Security reporting

# Tools & Technologies

- Wazuh 4.14.7
- Windows 10
- Kali Linux
- Wireshark
- VMware
- Windows Event Viewer
- PowerShell
- Linux command line
- MITRE ATT&CK

# Project Outcome

This lab provided hands-on experience with the complete SOC workflow:

```text
Collect
  ↓
Detect
  ↓
Triage
  ↓
Investigate
  ↓
Correlate
  ↓
Analyze IOCs
  ↓
Respond
  ↓
Document
  ↓
Close
```

The project demonstrates practical experience beyond simply configuring a SIEM, including detection engineering, alert investigation, endpoint analysis, incident classification, and professional security reporting.

# Repository Structure

```text
soc-analyst-home-lab/
│
├── README.md
│
├── architecture/
│   └── soc-lab-architecture.png
│
├── documentation/
│   ├── day1-wireshark.md
│   ├── day2-linux-logs.md
│   ├── day3-windows-events.md
│   ├── day4-process-investigation.md
│   ├── day5-wazuh-detection.md
│   └── day6-incident-response.md
│
├── detection-rules/
│   └── local_rules.xml
│
├── evidence/
│   ├── wireshark/
│   ├── linux/
│   ├── windows/
│   └── wazuh/
│
└── screenshots/
    ├── wazuh-dashboard.png
    ├── failed-login-4625.png
    ├── rule-100101.png
    ├── rule-100102.png
    └── incident-alert.png
```

# Disclaimer

This project was performed in an isolated home-lab environment for educational and defensive security training purposes.

All security testing and simulated incidents were performed against systems controlled by the lab environment.
