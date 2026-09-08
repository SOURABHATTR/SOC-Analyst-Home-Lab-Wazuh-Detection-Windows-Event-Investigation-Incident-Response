# 02 — Windows Agent Setup

## Objective

The second stage of the SOC home lab was to onboard a **Windows 10 endpoint** to the Wazuh server using the **Wazuh Agent**.

The objective was to establish communication between the endpoint and Wazuh so that Windows security telemetry could be collected centrally for SOC investigation.

---

## Lab Environment

| Component | Details |
|---|---|
| Endpoint | Windows 10 |
| Wazuh Agent | 4.14.7 |
| Wazuh Server | `192.168.244.128` |
| Windows Endpoint IP | `192.168.244.130` |
| Agent Name | `DESKTOP-2LUO2PD` |
| Agent ID | `001` |
| Agent Service | `WazuhSvc` |

---

## Network Configuration

The Windows endpoint and Wazuh server were connected through the virtual lab network.

```text
Wazuh Server
192.168.244.128
        │
        │ Wazuh Agent Communication
        │ TCP
        ▼
Windows 10
192.168.244.130
```

Connectivity to the Wazuh communication ports was verified during setup.

The endpoint was configured to communicate with the Wazuh Manager using:

```text
Manager: 192.168.244.128
Port: 1514
Protocol: TCP
```

---

## Wazuh Agent Configuration

The Windows Wazuh Agent was configured to communicate with the Wazuh Manager.

The relevant configuration was:

```xml
<ossec_config>
  <client>
    <server>
      <address>192.168.244.128</address>
      <port>1514</port>
      <protocol>tcp</protocol>
    </server>
    <config-profile>windows, windows10</config-profile>
    <crypto_method>aes</crypto_method>
    <notify_time>20</notify_time>
    <time-reconnect>60</time-reconnect>
    <auto_restart>yes</auto_restart>
  </client>

  <client_buffer>
    <disabled>no</disabled>
    <queue_size>5000</queue_size>
    <events_per_second>500</events_per_second>
  </client_buffer>

  <localfile>
    <location>Security</location>
    <log_format>eventchannel</log_format>
  </localfile>
</ossec_config>
```

### Configuration Purpose

The configuration established:

- Wazuh Manager address
- TCP communication
- Windows endpoint profile
- Agent encryption method
- Automatic agent restart
- Client event buffering
- Windows Security Event Channel collection

---

## Windows Security Event Collection

The following configuration enabled collection of the Windows **Security** Event Channel:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>
```

This was important because the later investigation depended on Windows authentication and security events such as:

- Event ID 4624 — Successful logon
- Event ID 4625 — Failed logon
- Event ID 4672 — Special privileges assigned
- Event ID 4634 — Logoff
- Event ID 4688 — Process creation
- Event ID 7045 — Service installation

---

## Agent Registration

The Windows endpoint was registered with the Wazuh Manager.

The resulting agent information was:

| Field | Value |
|---|---|
| Agent ID | `001` |
| Agent Name | `DESKTOP-2LUO2PD` |
| Version | `v4.14.7` |
| IP | `192.168.244.130` |
| Status | Active |

The agent appeared as active in the Wazuh environment, confirming successful onboarding.

---

## Service Verification

The Windows Wazuh Agent service was verified as running under:

```text
WazuhSvc
```

The running service confirmed that the endpoint-side Wazuh agent was operational and able to communicate with the Wazuh environment.

---

## Connectivity Verification

Connectivity between the Windows endpoint and Wazuh server was verified for the required Wazuh communication ports.

The lab confirmed communication with:

```text
TCP 1514
TCP 1515
```

TCP port `1514` was used for agent event communication, while port `1515` was used for agent enrollment/registration communication.

---

## Result

The Windows 10 endpoint was successfully onboarded to Wazuh.

The final communication path was:

```text
Windows 10
192.168.244.130
       │
       │ Wazuh Agent
       │
       ▼
Wazuh Manager
192.168.244.128
       │
       ▼
Wazuh Dashboard
```

The endpoint was active and configured to send Windows Security Event Channel telemetry to Wazuh.

This completed the endpoint onboarding stage and provided the telemetry required for the next stage: **Windows Event Investigation**.

---

## Evidence

Recommended evidence for this stage:

```text
screenshots/
├── wazuh-dashboard.png
└── windows-agent-active.png
```

Useful evidence includes:

- Wazuh Dashboard showing the Windows agent as active
- Windows Wazuh Agent service running
- Agent configuration
- Successful endpoint connectivity

---

## Key Takeaway

This stage established the connection between the monitored Windows endpoint and the central Wazuh SIEM.

The endpoint was not only registered with Wazuh, but also configured to send **Windows Security Event Channel telemetry**, enabling centralized SOC investigation in the following stages.
