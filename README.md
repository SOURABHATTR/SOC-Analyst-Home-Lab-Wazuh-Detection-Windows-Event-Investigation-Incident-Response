# SOC Analyst Home Lab — Wazuh Detection & Incident Response

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Windows](https://img.shields.io/badge/Endpoint-Windows%2010-blue)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-T1110-red)

## Overview

This project documents a hands-on **Security Operations Center (SOC) analyst home lab** built around **Wazuh SIEM** and a monitored **Windows 10 endpoint**.

The lab demonstrates a practical endpoint-monitoring and detection workflow:

```text
Wazuh Deployment
      ↓
Windows Endpoint Onboarding
      ↓
Windows Security Log Collection
      ↓
Event Investigation
      ↓
Process / Service Investigation
      ↓
Detection Engineering
      ↓
Alert Correlation
      ↓
IOC Analysis
      ↓
Incident Response
      ↓
SOC Reporting
```

The project focuses on practical SOC activities including endpoint telemetry collection, Windows event analysis, custom detection rules, alert correlation, investigation, MITRE ATT&CK mapping, and incident response.

---

# Objectives

- Deploy and configure Wazuh in a virtual lab
- Onboard a Windows 10 endpoint using the Wazuh Agent
- Collect Windows Security Event Logs
- Investigate Windows authentication activity
- Investigate process and service activity
- Analyze network connections associated with endpoint processes
- Create custom Wazuh detection rules
- Correlate repeated authentication failures
- Investigate a Level 10 Wazuh alert
- Perform IOC analysis
- Map suspicious behavior to MITRE ATT&CK
- Produce a professional SOC incident report

---

# Lab Architecture

```text
┌─────────────────────────┐
│       Windows 10        │
│                         │
│      Wazuh Agent        │
│      Security Logs      │
│      Process Events     │
└────────────┬────────────┘
             │
             │ Telemetry
             ▼
┌─────────────────────────┐
│      Wazuh Server       │
│                         │
│     Wazuh Manager       │
│     Wazuh Indexer       │
│     Wazuh Dashboard     │
└─────────────────────────┘
```

## Main Components

| Component | Purpose |
|---|---|
| Wazuh 4.14.7 | SIEM, endpoint monitoring and detection |
| Windows 10 | Monitored endpoint and Windows event investigation |
| Wazuh Agent | Collects Windows telemetry |
| VMware | Virtualized lab environment |
| Windows Event Logs | Authentication, process and security investigation |
| MITRE ATT&CK | Adversary behavior mapping |

---

# Lab Implementation

## 1. Wazuh Deployment

A **Wazuh 4.14.7 virtual appliance** was deployed in VMware.

The Wazuh environment provided:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Alert generation and event analysis

The Wazuh Dashboard was used as the primary SOC investigation interface.

---

## 2. Windows Endpoint Onboarding

A Windows 10 VM was connected to the isolated lab network and onboarded to Wazuh using the Wazuh Agent.

The endpoint was registered with Wazuh and configured to communicate with the Wazuh Manager.

The Windows agent was configured to collect the **Security Event Channel** for centralized investigation.

### Lab IPs

| System | IP Address |
|---|---|
| Wazuh Server | `192.168.244.128` |
| Windows 10 Endpoint | `192.168.244.130` |

---

# Windows Security Event Investigation

The following Windows events were investigated:

| Event ID | Description |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4672 | Special privileges assigned to a new logon |
| 4634 | Logoff |
| 4688 | Process creation |
| 7045 | Service installation |

## Investigation Areas

- Failed authentication
- Successful authentication
- Administrator activity
- Logon type
- Authentication package
- Process creation
- Windows services
- Network connections
- Event correlation

---

# Process & Endpoint Investigation

Endpoint activity was investigated using Windows process information and Wazuh telemetry.

Areas investigated included:

- Process IDs
- Parent-child process relationships
- `svchost.exe`
- Windows services
- Network connections
- Service installation events
- Process creation events

The investigated Windows processes and services were consistent with legitimate Windows activity.

No confirmed malicious process or persistence mechanism was identified.

---

# Detection Engineering

A controlled failed-authentication scenario was used to build a multi-stage Wazuh detection.

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

Detects failed authentication against the monitored administrator account:

```xml
<rule id="100101" level="8">
  <if_sid>60122</if_sid>
  <field name="win.eventdata.targetUserName">^Madhusughand$</field>
  <description>Custom Detection: Failed login attempt against administrator account</description>
  <group>windows,authentication_failed,custom_detection,</group>
</rule>
```

## Custom Rule 100102

Correlates repeated failed authentication events:

```xml
<rule id="100102" level="10" frequency="3" timeframe="120">
  <if_matched_sid>100101</if_matched_sid>
  <description>Custom Detection: Possible brute-force attack - 3 failed logins within 2 minutes</description>
  <group>windows,authentication_failed,bruteforce,custom_detection,</group>
</rule>
```

## Detection Result

The correlation rule successfully generated a **Level 10 alert** after detecting three failed authentication attempts within two minutes.

The correlation logic was validated through controlled testing in the isolated lab environment.

---

# Incident Investigation

## Potential Brute Force / Suspicious Authentication Activity

The Wazuh Level 10 alert was investigated as a complete SOC incident.

### Alert Details

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

---

# Investigation Findings

The investigation examined:

- Windows authentication events
- Failed and successful logons
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

---

# IOC Analysis

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

**IOC conclusion:** No confirmed malicious IOC was identified during the investigation.

---

# MITRE ATT&CK Mapping

## T1110 — Brute Force

The repeated failed authentication behavior was mapped to:

**MITRE ATT&CK T1110 — Brute Force**

This mapping represents the observed authentication behavior. It does not by itself prove that an attacker successfully compromised the endpoint.

---

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

---

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

> **Important:** The authentication failures were deliberately generated during controlled lab testing. The alert demonstrates that the detection logic worked; it should not be presented as evidence of a real-world attack against the endpoint.

---

# Incident Response

## Containment

For a real production incident, recommended actions would include:

- Monitor the targeted administrator account
- Review subsequent `4625`, `4624`, and `4672` events
- Apply account lockout or temporary restriction if attempts continue
- Identify and restrict the actual source if a malicious source is discovered
- Escalate if successful authentication follows repeated failures

Because this was a controlled lab investigation with no confirmed compromise, no destructive containment action was required.

## Eradication

No eradication action was required because no malicious process, persistence mechanism, or compromised account was identified.

## Recovery

- Continue endpoint monitoring
- Monitor authentication activity
- Maintain Wazuh detection rules
- Review future authentication events
- Escalate recurring suspicious activity

---

# Final SOC Verdict

> **True Detection — Potential Brute Force / Suspicious Authentication Activity, No Confirmed Compromise**

Wazuh successfully detected repeated failed authentication attempts against an administrator account.

The detection chain from Windows Event ID 4625 through Wazuh Rules **60122, 100101, and 100102** functioned as intended.

The associated authentication, process, and endpoint activity was investigated. No evidence of successful unauthorized access, malware execution, persistence, privilege escalation, or data compromise was identified.

The activity was generated as part of controlled security testing in the laboratory environment.

---

# Skills Demonstrated

- SOC alert triage
- SIEM monitoring
- Wazuh
- Windows Event Log analysis
- Endpoint monitoring
- Authentication investigation
- Process investigation
- Service investigation
- Event correlation
- Detection engineering
- Custom Wazuh rules
- Brute-force detection
- IOC analysis
- MITRE ATT&CK
- Incident response
- Incident documentation
- Security reporting

---

# Tools & Technologies

- Wazuh 4.14.7
- Windows 10
- VMware
- Windows Event Viewer
- PowerShell
- MITRE ATT&CK

---

# Project Outcome

This lab provided hands-on experience with a practical SOC workflow:

```text
Deploy
  ↓
Onboard Endpoint
  ↓
Collect Telemetry
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

The project demonstrates practical experience beyond simply configuring a SIEM, including endpoint onboarding, Windows telemetry analysis, detection engineering, alert investigation, endpoint/process analysis, incident classification, and professional security reporting.

---

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
│   ├── 01-wazuh-setup.md
│   ├── 02-windows-agent-setup.md
│   ├── 03-windows-event-investigation.md
│   ├── 04-process-investigation.md
│   ├── 05-wazuh-detection-engineering.md
│   └── 06-incident-response.md
│
├── detection-rules/
│   └── local_rules.xml
│
├── evidence/
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

---

# Disclaimer

This project was performed in an isolated home-lab environment for educational and defensive security training purposes.

All security testing and simulated incidents were performed against systems controlled by the lab environment.
