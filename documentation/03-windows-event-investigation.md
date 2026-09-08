# 03 — Windows Event Investigation

## Objective

The third stage of the SOC home lab focused on investigating **Windows Security Event Logs** collected by the Wazuh Agent.

The goal was to understand normal Windows authentication, privilege, process, logoff, and service activity and establish a baseline for identifying suspicious behavior.

---

# Windows Events Investigated

The following Windows Security Event IDs were investigated:

| Event ID | Description | Investigation Focus |
|---|---|---|
| `4624` | Successful logon | Authentication activity |
| `4625` | Failed logon | Failed authentication |
| `4672` | Special privileges assigned to a new logon | Privileged activity |
| `4634` | Logoff | Session termination |
| `4688` | Process creation | Process execution |
| `7045` | Service installation | Service/persistence investigation |

---

# 1. Event ID 4624 — Successful Logon

Event ID `4624` represents a successful authentication to the Windows endpoint.

During investigation, the following fields were considered:

- Target account
- Logon type
- Authentication package
- Source information
- Associated process
- Timestamp

Successful authentication events were used as context when reviewing failed authentication activity.

---

# 2. Event ID 4625 — Failed Logon

Event ID `4625` represents a failed authentication attempt.

The investigation focused on:

- Target username
- Failure reason
- Logon type
- Source IP
- Authentication package
- Associated process
- Frequency of failed attempts

A controlled failed-login scenario was later used for detection engineering.

The observed alert involved:

```text
Event ID:       4625
Target Account: Madhusughand
Logon Type:     2 — Interactive
Source IP:      127.0.0.1
Authentication: Negotiate
Process:        svchost.exe
```

The source address `127.0.0.1` indicated that the observed authentication attempt originated locally from the Windows endpoint. It was therefore not treated as evidence of an external attacker by itself.

---

# 3. Event ID 4672 — Special Privileges Assigned

Event ID `4672` was investigated to identify privileged logon activity.

The event can provide useful context when determining whether an account received elevated Windows privileges during a session.

Investigation focused on:

- Account associated with the event
- Logon/session context
- Privileges assigned
- Relationship with authentication events

No unauthorized privilege escalation was identified during the lab investigation.

---

# 4. Event ID 4634 — Logoff

Event ID `4634` represents a logoff event.

It was reviewed as part of understanding the lifecycle of Windows authentication sessions.

The event can be correlated with authentication activity to establish:

```text
Logon
  ↓
User Session
  ↓
Logoff
```

This provides useful context when investigating suspicious authentication activity.

---

# 5. Event ID 4688 — Process Creation

Event ID `4688` represents process creation.

Several Windows process creation events were investigated, including:

- `lsass.exe`
- `services.exe`
- `winlogon.exe`
- `wininit.exe`
- `csrss.exe`
- `autochk.exe`
- Registry-related process activity

The investigation considered:

- Process name
- Executable path
- Parent process
- Process relationship
- Execution context

The investigated processes were consistent with normal Windows system activity.

No confirmed malicious process execution was identified.

---

# 6. Event ID 7045 — Service Installation

Event ID `7045` represents the installation of a Windows service.

Service installation events were reviewed because attackers can abuse services for persistence or privileged execution.

The following services were investigated:

| Service | Path / Image | Assessment |
|---|---|---|
| `KslD` | `system32\drivers\wd\KslD.sys` | Legitimate Microsoft Defender component |
| `Microsoft Defender Core Service` | `C:\ProgramData\Microsoft\Windows Defender\platform\4.18.26080.3-0\MpDefenderCoreService.exe` | Legitimate Microsoft Defender component |
| `Microsoft WdAiNisDrv Driver` | `system32\drivers\wd\WdAiNisDrv.sys` | Legitimate Microsoft Defender component |
| `Microsoft Update Health Service` | `C:\Program Files\Microsoft Update Health Tools\uhssvc.exe` | Legitimate Microsoft component |

The services were assessed using their names, executable paths, service configuration, and relationship to known Microsoft security/update components.

No malicious service installation was identified.

---

# Authentication Investigation

The authentication investigation compared successful and failed logon activity.

Important fields included:

```text
Event ID
Target Username
Logon Type
Source IP
Authentication Package
Timestamp
Associated Process
```

The failed authentication activity later used for detection engineering targeted:

```text
Account: Madhusughand
Event:   4625
Source:  127.0.0.1
Logon:   Type 2
```

The activity was deliberately generated in the lab for detection testing.

---

# Event Correlation

Individual Windows events were not treated as isolated indicators.

The investigation used event relationships such as:

```text
4625 — Failed Logon
        ↓
Repeated Authentication Failures
        ↓
Detection Rule
        ↓
Wazuh Alert
        ↓
SOC Investigation
```

Other events such as `4624`, `4672`, `4634`, `4688`, and `7045` provided additional endpoint context.

This approach helped distinguish suspicious authentication behavior from normal Windows activity.

---

# Investigation Findings

The Windows event investigation established the following:

- Windows authentication events were successfully collected by Wazuh.
- Failed authentication could be identified using Event ID `4625`.
- Successful authentication could be reviewed using Event ID `4624`.
- Privileged activity could be investigated using Event ID `4672`.
- Process creation could be investigated using Event ID `4688`.
- Service installation could be investigated using Event ID `7045`.
- Investigated system processes were consistent with legitimate Windows activity.
- Investigated service installations were associated with legitimate Microsoft components.
- No confirmed malicious process or persistence mechanism was identified.

---

# Result

The Windows Security Event investigation established a practical baseline for endpoint activity.

The collected telemetry provided the information required to move from basic event review into **detection engineering and alert correlation**.

The next stage used the observed `4625` failed-authentication events to create custom Wazuh detection rules.
