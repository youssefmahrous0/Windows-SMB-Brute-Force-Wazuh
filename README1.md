# 🔐 Detecting SMB Brute-Force Activity with Wazuh

A hands-on **SOC home lab** demonstrating SMB authentication attack simulation, Windows Security Event Log analysis, Wazuh SIEM detection, event correlation, investigation, evidence collection, and MITRE ATT&CK mapping.

> **Attack Simulation → Detection → Investigation → Evidence → Correlation → MITRE ATT&CK → Documentation**

---

## 📌 Project Overview

In this SOC home lab, I simulated repeated failed **SMB authentication attempts** against a Windows 7 endpoint and investigated the resulting activity using **Wazuh SIEM**.

The investigation focused on:

* SMB service verification
* Authentication attack simulation
* Windows Security Event ID `4625`
* Windows Security Event ID `4624`
* Wazuh alert detection
* Source and target identification
* Authentication event correlation
* Timeline reconstruction
* MITRE ATT&CK mapping
* Evidence-based investigation
* SOC investigation documentation

The primary objective was to understand how a SOC analyst can detect, investigate, and correlate authentication activity instead of analyzing individual alerts in isolation.

---

## 🎯 Objectives

The main objectives of this lab were to:

* Simulate SMB authentication failures in a controlled environment.
* Verify SMB availability on the Windows endpoint.
* Analyze Windows Security Event ID `4625`.
* Identify the source IP, workstation, targeted account, and logon type.
* Validate Wazuh detection of failed authentication activity.
* Investigate Wazuh Rule ID `60122`.
* Correlate failed authentication events with a subsequent successful authentication.
* Analyze Windows Security Event ID `4624`.
* Build an authentication activity timeline.
* Map the observed behavior to MITRE ATT&CK.
* Document the investigation using collected evidence.

---

# 🏗️ Lab Environment

| Machine    | Role              | IP Address     |
| ---------- | ----------------- | -------------- |
| Kali Linux | Attacker / Source | `192.168.1.7`  |
| Windows 7  | Target / Endpoint | `192.168.1.13` |
| Ubuntu     | Wazuh Manager     | `192.168.1.12` |

### Technologies

* Kali Linux
* Windows 7
* Ubuntu
* Wazuh SIEM
* SMB
* Windows Security Event Logs
* Nmap
* smbclient
* MITRE ATT&CK

---

# 🔄 Investigation Workflow

```text
┌──────────────────────┐
│   SMB Verification   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Authentication       │
│ Simulation           │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Windows Event ID     │
│ 4625 - Failed Logon  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Wazuh Detection      │
│ Rule ID 60122        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Source & Target      │
│ Investigation        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Event ID 4624        │
│ Successful Logon     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Event Correlation    │
│ & Timeline           │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ MITRE ATT&CK Mapping │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Findings & Report    │
└──────────────────────┘
```

---

# 1. SMB Service Verification

Before starting the authentication simulation, I verified that the SMB service was accessible on the Windows 7 endpoint.

### Command

```bash
nmap -Pn -p 445 192.168.1.13
```

### Result

```text
445/tcp open microsoft-ds
```

The scan confirmed that TCP port `445` was open and that the Microsoft-DS/SMB service was accessible from the Kali Linux machine.

### Evidence

![SMB Service Verification](screenshots/01-smb-port-445.png)

### SOC Relevance

Verifying the exposed service provides initial network context before investigating authentication activity.

---

# 2. SMB Authentication Simulation

After confirming SMB availability, I used `smbclient` to interact with the Windows SMB service.

The lab intentionally generated multiple failed authentication attempts against the test account:

```text
labuser
```

### Command

```bash
smbclient //192.168.1.13/IPC$ -U 'labuser%Wrong Password123!'
```

### Response

```text
NT_STATUS_LOGON_FAILURE
```

The authentication attempt failed as expected.

This generated Windows Security Event ID `4625`, which was subsequently collected and detected by Wazuh.

### Evidence

![SMB Authentication Simulation](screenshots/02-smb-authentication.png)

---

# 3. Windows Security Event ID 4625

Windows recorded the failed authentication attempt as:

```text
Event ID: 4625
```

The event provided important investigation details.

| Field          | Value         |
| -------------- | ------------- |
| Event ID       | `4625`        |
| Account        | `labuser`     |
| Source IP      | `192.168.1.7` |
| Workstation    | `KALI`        |
| Logon Type     | `3 - Network` |
| Authentication | `NTLM`        |

### Logon Type

`Logon Type 3` represents a **network logon**, which is consistent with the SMB authentication activity performed in this lab.

### Evidence

![Windows Event ID 4625](screenshots/03-windows-event-4625.png)

### Investigation Value

Event ID `4625` provided the key information required to continue the investigation:

* Who was targeted?
* Where did the authentication originate?
* Which workstation generated the activity?
* What type of logon occurred?
* Which authentication mechanism was used?

---

# 4. Wazuh Detection

The Windows Security Events were collected by the Wazuh Agent and forwarded to the Wazuh Manager.

Wazuh detected the failed authentication activity using:

| Detection Field | Value                                          |
| --------------- | ---------------------------------------------- |
| Rule ID         | `60122`                                        |
| Level           | `5`                                            |
| Description     | `Logon failure - Unknown user or bad password` |

Multiple authentication failure events were observed during the simulation.

### Evidence

![Wazuh Security Events](screenshots/04-wazuh-alert.png)

### Detection Result

Wazuh successfully received and detected the Windows Security Event ID `4625`.

The alert was triggered by:

```text
Rule ID: 60122
Level: 5
```

This confirmed that the Windows authentication events were successfully collected and processed by the Wazuh Manager.

---

# 5. Source and Target Investigation

The investigation established the following:

### Source

```text
Host: KALI
IP:   192.168.1.7
```

### Target

```text
Host: Windows 7
IP:   192.168.1.13
```

### Target Account

```text
labuser
```

### Investigation Summary

| Investigation Item | Result         |
| ------------------ | -------------- |
| Source Host        | `KALI`         |
| Source IP          | `192.168.1.7`  |
| Target Host        | Windows 7      |
| Target IP          | `192.168.1.13` |
| Target Account     | `labuser`      |
| Protocol           | SMB            |
| Destination Port   | `445/TCP`      |
| Failed Event       | `4625`         |
| Logon Type         | `3 - Network`  |
| Authentication     | `NTLM`         |

### Finding

The source host `KALI` (`192.168.1.7`) was identified as the origin of the SMB authentication attempts against the Windows 7 endpoint (`192.168.1.13`).

The targeted account was:

```text
labuser
```

The activity occurred over:

```text
SMB / TCP 445
```

### Evidence

![Wazuh Event Details](screenshots/05-wazuh-event-details.png)

---

# 6. Successful Authentication — Event ID 4624

After the failed authentication attempts, a valid authentication was performed using the test account.

Windows generated:

```text
Event ID: 4624
```

The event showed:

| Field          | Value         |
| -------------- | ------------- |
| Account        | `labuser`     |
| Logon Type     | `3`           |
| Workstation    | `KALI`        |
| Source IP      | `192.168.1.7` |
| Authentication | `NTLM`        |
| Package        | `NTLM V2`     |

This allowed the authentication activity to be correlated as:

```text
4625 - Failed Logon
        ↓
4625 - Failed Logon
        ↓
4625 - Failed Logon
        ↓
4624 - Successful Logon
```

### Evidence

![Windows Event ID 4624](screenshots/06-windows-event-4624.png)

### Investigation Significance

The successful authentication event was correlated with the previous failed authentication attempts because the activity involved the same:

* Source IP
* Workstation
* Target account
* Network logon type
* SMB authentication context

---

# 7. Investigation Timeline

The documented sequence of failed authentication attempts began around:

```text
19 September 2026 – 2:10:43 PM
```

### Timeline Context

```text
Source:
192.168.1.7 / KALI

        │
        │ SMB / TCP 445
        ▼

Target:
192.168.1.13 / Windows 7

        │
        ▼

Account:
labuser

        │
        ▼

Multiple Failed Logons
Event ID 4625

        │
        ▼

Successful Authentication
Event ID 4624
```

### Timeline Fields

| Field                     | Value                      |
| ------------------------- | -------------------------- |
| Source                    | `192.168.1.7 / KALI`       |
| Target                    | `192.168.1.13 / Windows 7` |
| Account                   | `labuser`                  |
| Protocol                  | SMB                        |
| Port                      | `445/TCP`                  |
| Logon Type                | `3 - Network`              |
| Failed Authentication     | Event `4625`               |
| Successful Authentication | Event `4624`               |

### Evidence

![Investigation Timeline](screenshots/07-mitre-attack.png)

---

# 8. MITRE ATT&CK Mapping

The observed activity was mapped to the following MITRE ATT&CK techniques.

## T1110 — Brute Force

The lab generated repeated failed authentication attempts against the same test account, followed by a successful authentication.

This behavior was part of a controlled password-guessing simulation performed inside the lab environment.

```text
T1110 — Brute Force
```

---

## T1021.002 — SMB/Windows Admin Shares

The authentication activity involved:

```text
SMB
TCP/445
IPC$
```

This is relevant to:

```text
T1021.002 — SMB/Windows Admin Shares
```

The evidence collected in this lab demonstrates SMB authentication/access to `IPC$`.

It does **not** demonstrate:

* Remote command execution
* Remote code execution
* Confirmed lateral movement

This distinction is important because the investigation conclusions are based only on the evidence collected.

---

# 9. Evidence Collected

The investigation produced the following evidence.

### Evidence 1 — SMB Service Verification

```text
TCP/445 → Open
```

### Evidence 2 — SMB Authentication Activity

```text
Kali → Windows 7
SMB / TCP 445
```

### Evidence 3 — Windows Event ID 4625

```text
Failed Authentication

User:
labuser

Source:
192.168.1.7
```

### Evidence 4 — Wazuh Detection

```text
Rule ID: 60122
Level: 5
```

### Evidence 5 — Windows Event ID 4624

```text
Successful Authentication

User:
labuser

Source:
192.168.1.7
```

### Evidence 6 — MITRE ATT&CK Mapping

```text
T1110
T1021.002
```

---

# 🔎 Investigation Summary

| Category         | Finding                                           |
| ---------------- | ------------------------------------------------- |
| Attack Type      | SMB Authentication / Password Guessing Simulation |
| Source           | Kali Linux                                        |
| Source IP        | `192.168.1.7`                                     |
| Target           | Windows 7                                         |
| Target IP        | `192.168.1.13`                                    |
| Account          | `labuser`                                         |
| Protocol         | SMB                                               |
| Port             | `445/TCP`                                         |
| Failed Event     | `4625`                                            |
| Successful Event | `4624`                                            |
| Logon Type       | `3 - Network`                                     |
| Authentication   | NTLM                                              |
| Wazuh Rule       | `60122`                                           |
| Wazuh Level      | `5`                                               |
| MITRE ATT&CK     | `T1110`, `T1021.002`                              |

---

# 🚨 Key Findings

* Multiple failed SMB authentication attempts were observed against the Windows 7 endpoint.
* The attempts targeted the `labuser` account.
* The source workstation was identified as `KALI`.
* The source IP address was `192.168.1.7`.
* Windows generated Event ID `4625` for the failed authentication attempts.
* Wazuh successfully detected the events using Rule ID `60122`.
* A subsequent successful authentication generated Event ID `4624`.
* The failed and successful authentication events were correlated using common investigation fields.
* The observed activity was consistent with the controlled password-guessing simulation performed in the lab.
* SMB/`IPC$` access was observed.
* No remote command execution or confirmed lateral movement was demonstrated by the collected evidence.

---

# 🛡️ SOC Analyst Investigation Workflow

This lab demonstrates a practical SOC investigation workflow:

### 1. Alert Triage

Identify the authentication alert and determine whether it requires further investigation.

### 2. Event Analysis

Analyze the Windows Security Event ID `4625` and extract relevant fields.

### 3. Source Identification

Determine the source IP address and originating workstation.

### 4. Target Identification

Identify the targeted endpoint and account.

### 5. Context Analysis

Review the protocol, destination port, logon type, and authentication package.

### 6. Event Correlation

Search for related authentication events, including successful Event ID `4624`.

### 7. Timeline Reconstruction

Place the authentication events into chronological order.

### 8. MITRE Mapping

Map the observed behavior to relevant ATT&CK techniques.

### 9. Evidence-Based Conclusion

Document what the evidence confirms and clearly distinguish it from activity that was not demonstrated.

---

# 🧰 Skills Demonstrated

* SOC Alert Investigation
* SIEM Monitoring with Wazuh
* Windows Event Log Analysis
* Windows Authentication Analysis
* SMB Investigation
* Network Service Verification
* Source IP Identification
* Authentication Event Correlation
* Timeline Reconstruction
* MITRE ATT&CK Mapping
* Security Event Investigation
* Evidence Collection
* Incident Investigation Documentation

---

# 📸 Screenshots

The `screenshots/` directory contains investigation evidence collected during the lab.

Examples include:

* SMB service verification
* SMB authentication simulation
* Windows Event ID `4625`
* Wazuh security alerts
* Wazuh event details
* Windows Event ID `4624`
* Investigation timeline

---

# 📚 Related Article

A detailed investigation write-up is also available on Medium:

**Detecting SMB Brute-Force Activity with Wazuh: A Windows SOC Investigation**

[Read the full investigation on Medium](https://medium.com/@jomahrous0/detecting-smb-brute-force-activity-with-wazuh-a-windows-soc-investigation-8a186203d5fe)

---

# ⚠️ Disclaimer

This project was conducted in an isolated home lab using intentionally configured test systems and accounts.

All authentication activity was simulated for educational and defensive security purposes.

Do not perform authentication attacks or security testing against systems without explicit authorization.

---

# 📌 Conclusion

This lab demonstrated a complete SOC investigation workflow for SMB authentication activity.

I successfully:

* Verified SMB availability.
* Simulated failed SMB authentication attempts.
* Analyzed Windows Security Event ID `4625`.
* Investigated the source IP and targeted account.
* Validated Wazuh detection.
* Analyzed Wazuh Rule ID `60122`.
* Correlated failed authentication events with Event ID `4624`.
* Reconstructed the authentication timeline.
* Mapped the observed behavior to MITRE ATT&CK.
* Documented the investigation using collected evidence.

The exercise reinforced an important SOC investigation principle:

> **A single authentication failure may provide limited context, but correlating repeated failures with subsequent authentication activity can reveal a much clearer picture of the incident.**

---

## ⭐ Project Focus

```text
SOC
│
├── Wazuh SIEM
├── Windows Security Logs
├── Authentication Monitoring
├── SMB Investigation
├── Event Correlation
├── Timeline Analysis
├── MITRE ATT&CK
└── Incident Documentation
```

---

**Author:** Youssef Mahrous
**Focus:** SOC Analyst | Blue Team | Cybersecurity
