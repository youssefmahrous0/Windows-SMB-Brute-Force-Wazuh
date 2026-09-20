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
