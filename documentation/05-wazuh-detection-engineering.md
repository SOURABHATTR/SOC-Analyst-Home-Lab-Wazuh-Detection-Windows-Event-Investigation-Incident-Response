# 05 — Wazuh Detection Engineering

## Objective

The fifth stage of the SOC home lab focused on building and validating **custom Wazuh detection rules** for repeated failed authentication activity.

The objective was to move from basic Windows event collection to a detection workflow capable of identifying a specific authentication pattern and generating a higher-severity correlated alert.

---

# Detection Scenario

A controlled failed-authentication scenario was generated against the monitored Windows endpoint.

The Windows Security Event:

```text
Event ID: 4625
Description: Failed logon
```

was used as the starting point for the detection chain.

The detection logic was designed in two stages:

```text
Windows Event 4625
        ↓
Wazuh Rule 60122
        ↓
Custom Rule 100101
        ↓
Target Administrator Account
        ↓
Repeated Failed Authentication
        ↓
Custom Rule 100102
        ↓
Level 10 Alert
```

---

# 1. Base Wazuh Detection

Windows Event ID `4625` was already being detected by Wazuh using rule:

```text
60122
```

The event represented a failed Windows authentication attempt.

The existing Wazuh rule provided the base event needed for custom detection.

---

# 2. Custom Rule 100101

The first custom rule was created to identify failed authentication attempts against the monitored administrator account.

```xml
<rule id="100101" level="8">
  <if_sid>60122</if_sid>
  <field name="win.eventdata.targetUserName">^Madhusughand$</field>
  <description>Custom Detection: Failed login attempt against administrator account</description>
  <group>windows,authentication_failed,custom_detection,</group>
</rule>
```

## Rule Logic

The rule:

1. Depends on Wazuh rule `60122`.
2. Examines the Windows target username.
3. Matches the specific account:
   ```text
   Madhusughand
   ```
4. Raises the alert level to `8`.
5. Places the event into custom authentication-related groups.

The rule therefore narrowed the generic failed-login detection into a more specific condition.

---

# 3. Custom Rule 100102

The second custom rule was created to correlate repeated detections from rule `100101`.

```xml
<rule id="100102" level="10" frequency="3" timeframe="120">
  <if_matched_sid>100101</if_matched_sid>
  <description>Custom Detection: Possible brute-force attack - 3 failed logins within 2 minutes</description>
  <group>windows,authentication_failed,bruteforce,custom_detection,</group>
</rule>
```

## Rule Logic

The rule required:

```text
3 matches
within
120 seconds
```

of the previous custom detection rule:

```text
100101
```

When the condition was satisfied, Wazuh generated a:

```text
Level 10 Alert
```

---

# Correlation Logic

The important part of the detection was the relationship between the two custom rules.

```text
4625 Failed Logon
       ↓
Rule 60122
       ↓
Rule 100101
       ↓
Administrator Account Targeted
       ↓
3 Matching Events
       ↓
120 Second Window
       ↓
Rule 100102
       ↓
Level 10 Alert
```

The correlation rule was intentionally built on `100101` rather than directly correlating from the original `60122` rule.

This ensured that the repeated events first satisfied the specific administrator-account condition before being counted toward the correlation threshold.

---

# Detection Testing

The detection was tested using controlled failed authentication attempts in the isolated lab.

The test generated:

```text
Attempt 1 → Rule 100101
Attempt 2 → Rule 100101
Attempt 3 → Rule 100101
                 ↓
          Rule 100102
                 ↓
           Level 10 Alert
```

The correlation successfully generated a Level 10 alert after three failed authentication attempts occurred within two minutes.

---

# Alert Result

The resulting alert contained:

| Field | Value |
|---|---|
| Wazuh Rule | `100102` |
| Alert Level | `10` |
| Event ID | `4625` |
| Target Account | `Madhusughand` |
| Frequency | 3 attempts |
| Timeframe | 120 seconds |
| Source IP | `127.0.0.1` |
| Logon Type | `2 — Interactive` |
| Authentication | `Negotiate` |
| Process | `svchost.exe` |

The alert demonstrated that the custom detection and correlation logic functioned as intended.

---

# Detection Engineering Considerations

The detection was designed to demonstrate several SOC detection concepts:

### Specificity

Rule `100101` narrowed generic failed authentication events to a specific monitored administrator account.

### Correlation

Rule `100102` correlated multiple matching authentication events within a defined time window.

### Severity Escalation

The detection progressed from:

```text
Rule 60122 → Base Detection
Rule 100101 → Level 8
Rule 100102 → Level 10
```

### Context

The alert was not treated as automatic proof of compromise. The resulting event required investigation of:

- Source IP
- Target account
- Logon type
- Process
- Authentication package
- Related authentication events
- Endpoint activity

---

# Validation

The custom detection was successfully validated in the lab.

The final detection chain was:

```text
Windows Security Event 4625
        ↓
Wazuh Rule 60122
        ↓
Custom Rule 100101
        ↓
3 Repeated Matches
        ↓
Custom Rule 100102
        ↓
Level 10 Alert
```

This confirmed that the custom rules were being evaluated correctly and that the correlation threshold worked as designed.

---

# SOC Interpretation

The detection represents **potential brute-force or suspicious authentication behavior**, not confirmed compromise.

The failed authentication attempts were deliberately generated during controlled lab testing.

The observed source IP was:

```text
127.0.0.1
```

which is the Windows localhost address. It was therefore not treated as evidence of an external malicious source.

No successful unauthorized login, malicious process, persistence mechanism, or data compromise was identified during the associated investigation.

---

# MITRE ATT&CK Mapping

The observed repeated authentication failure behavior was mapped to:

**T1110 — Brute Force**

The mapping describes the observed authentication behavior and does not establish that an attacker successfully compromised the endpoint.

---

# Result

The detection engineering stage successfully demonstrated:

- Use of an existing Wazuh rule as a detection base
- Creation of custom Wazuh rules
- Field-based event matching
- Account-specific detection
- Frequency-based correlation
- Time-window correlation
- Severity escalation
- Controlled detection validation
- SOC-oriented alert interpretation

The resulting Level 10 alert was then used as the basis for the following stage: **Incident Response**.
