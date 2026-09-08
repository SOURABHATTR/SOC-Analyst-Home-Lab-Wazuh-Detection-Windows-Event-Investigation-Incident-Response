# 04 — Process Investigation

## Objective

The fourth stage of the SOC home lab focused on investigating **Windows processes, parent-child relationships, services, and network connections** associated with endpoint activity.

The objective was to determine whether observed process activity was consistent with normal Windows behavior or indicated potentially malicious execution.

---

# Investigation Method

Process investigation focused on the relationship between:

```text
Process
  ↓
Parent Process
  ↓
Executable Path
  ↓
Associated Services
  ↓
Network Connections
  ↓
Security Events
```

The investigation used Windows process information together with Wazuh telemetry and Windows security events.

---

# 1. svchost.exe Investigation

One of the investigated processes was:

```text
Process: svchost.exe
PID:     348
Path:    C:\Windows\System32\svchost.exe
Parent:  services.exe
Parent PID: 636
```

The executable path was consistent with the legitimate Windows System32 location.

The parent-child relationship was also consistent with normal Windows service hosting:

```text
services.exe
     ↓
svchost.exe
```

---

# 2. Services Hosted by svchost.exe

The investigated `svchost.exe` instance was associated with multiple Windows services.

The observed services included:

- Appinfo
- gpsvc
- iphlpsvc
- LanmanServer
- lfsvc
- ProfSvc
- Schedule
- seclogon
- SENS
- ShellHWDetection
- Themes
- TokenBroker
- UserManager
- UsoSvc
- Winmgmt
- WpnService
- wuauserv

These services were assessed as normal Windows services.

The process was therefore not considered suspicious based on its name alone.

---

# 3. Process Path Validation

Executable paths were reviewed as part of the investigation.

For example:

```text
C:\Windows\System32\svchost.exe
```

is a standard Windows system location.

Path validation is important during SOC investigations because malware may attempt to imitate legitimate Windows process names while executing from unusual locations.

In this investigation, the observed `svchost.exe` path was consistent with legitimate Windows activity.

---

# 4. Network Connection Investigation

Network connections associated with endpoint processes were reviewed to provide additional context.

Observed connections included:

| Process | PID | Destination |
|---|---:|---|
| `msedge.exe` | 6656 | `150.171.28.11:443` |
| `MpDefenderCoreService` | 5116 | `20.42.73.31:443` |
| `svchost.exe` | 364 | `199.232.214.172:80` |
| `svchost.exe` | 364 | `4.213.25.240:443` |
| Wazuh Agent | 2264 | `192.168.244.128:1514` |

The connections were assessed in the context of their associated processes.

The Wazuh Agent connection to:

```text
192.168.244.128:1514
```

was expected because it represented communication between the Windows endpoint and the Wazuh Manager.

---

# 5. svchost.exe Network Activity

Another `svchost.exe` instance was investigated:

```text
PID: 364
Path: C:\Windows\System32\svchost.exe -k netsvcs -p
```

The associated network connections were:

```text
199.232.214.172:80
4.213.25.240:443
```

The process path and service-hosting configuration were consistent with normal Windows system activity.

No confirmed malicious process was identified from this investigation.

---

# 6. Service Installation Investigation

Windows Event ID `7045` was also reviewed as part of endpoint process and service investigation.

The following service installations were examined:

### KslD

```text
Service: KslD
Path:    system32\drivers\wd\KslD.sys
Type:    Kernel Driver
Start:   Demand
```

This was assessed as a legitimate Microsoft Defender component.

### Microsoft Defender Core Service

```text
Service: Microsoft Defender Core Service
Path:    C:\ProgramData\Microsoft\Windows Defender\platform\4.18.26080.3-0\MpDefenderCoreService.exe
Type:    User Mode
Start:   Auto
Account: LocalSystem
```

This was assessed as legitimate Microsoft Defender activity.

### Microsoft WdAiNisDrv Driver

```text
Service: Microsoft WdAiNisDrv Driver
Path:    system32\drivers\wd\WdAiNisDrv.sys
Type:    Kernel Driver
Start:   Demand
```

This was assessed as a legitimate Microsoft Defender component.

### Microsoft Update Health Service

```text
Service: Microsoft Update Health Service
Path:    C:\Program Files\Microsoft Update Health Tools\uhssvc.exe
Type:    User Mode
Start:   Disabled
Account: LocalSystem
```

This was assessed as a legitimate Microsoft update-related component.

---

# 7. Process Creation Investigation

Windows Event ID `4688` process creation events were reviewed to establish whether newly created processes followed expected Windows execution patterns.

Investigated processes included:

```text
lsass.exe
services.exe
winlogon.exe
wininit.exe
csrss.exe
autochk.exe
Registry
```

The investigation considered:

- Process name
- Executable path
- Parent process
- Execution context
- Relationship with normal Windows startup/system activity

The observed processes were consistent with legitimate Windows operation.

---

# Investigation Findings

The process investigation established:

- `svchost.exe` was running from the expected System32 path.
- Its parent process relationship with `services.exe` was normal.
- Hosted services were consistent with legitimate Windows services.
- Network connections were reviewed in process context.
- Wazuh Agent network traffic to the Wazuh Manager was expected.
- Investigated service installations were associated with legitimate Microsoft components.
- Process creation events reviewed were consistent with normal Windows system activity.
- No confirmed malicious process was identified.
- No confirmed malicious persistence mechanism was identified.

---

# SOC Investigation Perspective

The investigation demonstrated why process names alone should not be treated as indicators of compromise.

For example:

```text
svchost.exe
```

is a legitimate Windows process, but a SOC analyst should still validate:

```text
Process Name
     ↓
Executable Path
     ↓
Parent Process
     ↓
Hosted Services
     ↓
Network Connections
     ↓
Related Security Events
```

This contextual approach reduces false positives and provides stronger evidence for determining whether endpoint activity is suspicious.

---

# Result

The endpoint process and service investigation established that the observed activity was consistent with legitimate Windows behavior.

No malicious executable, suspicious persistence mechanism, or confirmed malicious network activity was identified.

The investigation provided additional endpoint context for the later **Wazuh detection engineering and incident response** stages.
