# Detecting SMB Brute-Force Activity with Wazuh

A SOC home lab demonstrating SMB authentication attack simulation, Windows Security Event Log analysis, Wazuh detection, investigation, event correlation, and MITRE ATT&CK mapping.

## 📌 Project Overview

In this hands-on SOC home lab, I simulated repeated failed SMB authentication attempts against a Windows 7 endpoint and investigated the activity using Wazuh SIEM.

The objective was to practice a complete SOC investigation workflow:

> Attack → Detection → Investigation → Evidence → MITRE ATT&CK → Documentation

The investigation focused on Windows Security Events `4625` and `4624`, identifying the source and targeted account, analyzing the authentication method, and correlating failed and successful authentication events.

---

## 🏗️ Lab Environment

| Machine | Role | IP Address |
|---|---|---|
| Kali Linux | Attacker / Source | `192.168.1.7` |
| Windows 7 | Target / Endpoint | `192.168.1.13` |
| Ubuntu | Wazuh Manager | `192.168.1.12` |

### Technologies

- Kali Linux
- Windows 7
- Wazuh SIEM
- SMB
- Windows Security Event Logs
- Nmap
- smbclient
- MITRE ATT&CK

---

# 1. SMB Service Verification

Before starting the authentication simulation, I verified that SMB was accessible on the Windows 7 endpoint.

### Command

```bash
nmap -Pn -p 445 192.168.1.13
```

The scan confirmed:
```text
445/tcp open microsoft-ds
```
This confirmed that the SMB service was reachable on the target.

### Evidence
The Nmap scan provided evidence that TCP port `445` was open on the Windows 7 endpoint and that the Microsoft-DS (SMB) service was accessible.

![SMB Service Verification](screenshots/01-smb-port-445.png)

### Result
TCP port 445 was open.
The Microsoft-DS (SMB) service was accessible on the Windows 7 endpoint.
The target was reachable from the Kali Linux machine.

Next Step
With SMB confirmed as accessible, the next step was to simulate SMB authentication attempts against the Windows 7 endpoint using the smbclient utility.

---

# 2. SMB Authentication Simulation

After confirming SMB availability, I used `smbclient` to interact with the Windows SMB service.

The lab intentionally generated multiple failed authentication attempts against the test account `labuser`.

### Command

```bash
smbclient //192.168.1.13/IPC$ -U 'labuser%Wrong Password123!'
```
The authentication attempt returned:

```text
NT_STATUS_LOGON_FAILURE
```
### Evidence

The Kali Linux terminal showed the SMB authentication attempt against the Windows 7 `IPC$` share. The failed authentication generated an `NT_STATUS_LOGON_FAILURE` response, confirming that the authentication attempt was unsuccessful.

![SMB Authentication Simulation](screenshots/02-smb-authentication.png)

### Result

The SMB authentication attempt failed as expected and generated a Windows Security Event ID `4625`, which was later detected by Wazuh.

### Next Step

The next step was to investigate the generated Windows Security Event ID `4625` and identify the source IP, targeted account, and logon type.

---

### 3. Windows Security Event ID 4625

Windows recorded the failed authentication attempts as Event ID 4625.

The event provided important investigation details:

Field	Value
Event ID	4625
Account	labuser
Source IP	192.168.1.7
Workstation	KALI
Logon Type	3 - Network
Authentication	NTLM

The Logon Type 3 indicates a network logon, which is consistent with the SMB authentication activity performed in the lab.

### Evidence

Windows Event Viewer captured Event ID `4625`, showing the failed authentication attempt against the `labuser` account. The event identified `KALI` as the source workstation with IP address `192.168.1.7` and recorded the logon type as `3 - Network`.

![Windows Security Event 4625](screenshots/03-windows-event-4625.png)

### Result

The Windows Security Event Log confirmed that the SMB authentication attempt from the Kali Linux machine failed for the `labuser` account.

### Next Step

The next step was to verify whether Wazuh successfully collected and detected the Event ID `4625` from the Windows 7 endpoint.

---
