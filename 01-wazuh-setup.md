# 01 — Wazuh Setup

## Objective

The first stage of the SOC home lab was to deploy a **Wazuh 4.14.7** environment in VMware and verify that the Wazuh platform was accessible and ready to receive endpoint telemetry.

The Wazuh deployment served as the central monitoring and investigation platform for the Windows endpoint used later in the lab.

---

## Lab Environment

| Component | Details |
|---|---|
| Virtualization | VMware |
| Wazuh Version | 4.14.7 |
| Wazuh Server IP | `192.168.244.128` |
| Dashboard | Wazuh Dashboard |
| Deployment Type | Wazuh virtual appliance |

---

## Wazuh Components

The deployed Wazuh environment provided the following components:

### Wazuh Manager

The Wazuh Manager acted as the central security monitoring component. It received endpoint telemetry, processed events, applied detection rules, and generated alerts.

### Wazuh Indexer

The Wazuh Indexer stored and indexed security events so that they could be searched and investigated through the dashboard.

### Wazuh Dashboard

The Wazuh Dashboard provided the primary interface for:

- Monitoring endpoint status
- Searching security events
- Investigating Windows telemetry
- Reviewing Wazuh alerts
- Analyzing detection results

---

## Network Configuration

The Wazuh server was configured with the following lab IP address:

```text
Wazuh Server
IP: 192.168.244.128
```

The lab used an isolated virtual network for communication between the Wazuh server and the Windows endpoint.

---

## Dashboard Verification

After deployment, the Wazuh Dashboard was accessed using the Wazuh server address:

```text
https://192.168.244.128
```

Successful dashboard access confirmed that the Wazuh platform was available for subsequent endpoint onboarding and investigation activities.

---

## Initial Verification

The following items were verified during the setup stage:

- Wazuh virtual appliance was running
- Wazuh Manager was available
- Wazuh Indexer was available
- Wazuh Dashboard was accessible
- Wazuh server had the expected lab IP address
- The environment was ready for Windows endpoint onboarding

---

## Result

The Wazuh 4.14.7 environment was successfully deployed in VMware and made available as the central SOC monitoring platform.

The completed setup provided the foundation for the next stage:

```text
Wazuh Deployment
        ↓
Windows Endpoint Onboarding
        ↓
Security Telemetry Collection
        ↓
Event Investigation
```

---

## Evidence

Recommended evidence for this stage:

```text
screenshots/
└── wazuh-dashboard.png
```

The dashboard screenshot should demonstrate successful access to the deployed Wazuh environment.

---

## Key Takeaway

This stage established the **central SIEM and endpoint-monitoring infrastructure** required for the rest of the SOC lab. No detection or incident response activity was performed at this stage; those activities were completed in later stages of the project.
