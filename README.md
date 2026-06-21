<img width="1280" height="714" alt="SOC LAB" src="https://github.com/user-attachments/assets/96d952af-e53b-47bc-a6ab-fa7f994b0c63" />


# 🛡️ DLL Injection Detection Lab with Wazuh SIEM
## Threat Monitoring, Detection Engineering, and Incident Investigation for MITRE ATT&CK T1055

---

# 📌 Project Overview

This project demonstrates how a SOC Engineer can detect, monitor, investigate, and respond to DLL Injection-related activities using:

* Wazuh SIEM
* Sysmon
* Windows 11 Endpoint
* Ubuntu 22.04 Wazuh Server
* Elastic Stack
* MITRE ATT&CK Framework

The objective is to build a detection pipeline capable of identifying suspicious process access, remote thread creation, and malicious DLL loading behaviors commonly associated with Process Injection (T1055).

---

# 🎯 Learning Objectives

After completing this project, you will be able to:

✅ Deploy a Wazuh-based SOC Lab

✅ Collect advanced endpoint telemetry

✅ Monitor Sysmon security events

✅ Detect DLL Injection indicators

✅ Create custom Wazuh detection rules

✅ Investigate security alerts

✅ Map detections to MITRE ATT&CK

---

# 🏗️ Lab Architecture

```text
┌─────────────────────┐
│ Windows 11 Endpoint │
│  Sysmon Installed   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│    Wazuh Agent      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Wazuh Manager     │
│   Ubuntu 22.04      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│     Filebeat        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Elasticsearch       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Wazuh Dashboard     │
└─────────────────────┘
```

---

# 🖥️ Environment Setup

| Component        | Version    |
| ---------------- | ---------- |
| Ubuntu Server    | 22.04      |
| Windows Endpoint | Windows 11 |
| Wazuh            | 4.x        |
| Sysmon           | Latest     |
| Elastic Stack    | 8.x        |
| VirtualBox       | Latest     |

---

# Phase 1: Install Sysmon

## Download Sysmon

Official Source:

https://learn.microsoft.com/sysinternals

Extract Sysmon package.

Install:

```powershell
Sysmon64.exe -accepteula -i
```

Verify Installation:

```powershell
Get-Service Sysmon64
```

Expected Output:

```text
Status : Running
```

---

# Phase 2: Deploy SwiftOnSecurity Configuration

Download Sysmon configuration:

```powershell
git clone https://github.com/SwiftOnSecurity/sysmon-config.git
```

Apply Configuration:

```powershell
Sysmon64.exe -c sysmonconfig-export.xml
```

Verify:

```powershell
Sysmon64.exe -s
```

---

# Phase 3: Install Wazuh Agent

Download Agent:

```powershell
Invoke-WebRequest `
-Uri https://packages.wazuh.com/4.x/windows/wazuh-agent.msi `
-OutFile wazuh-agent.msi
```

Install:

```powershell
msiexec.exe /i wazuh-agent.msi
```

Configure Manager IP:

```text
192.168.1.100
```

Start Service:

```powershell
NET START WazuhSvc
```

---

# Phase 4: Enable Sysmon Log Collection

Edit:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Add:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Restart Agent:

```powershell
Restart-Service WazuhSvc
```

---

# Phase 5: Validate Event Collection

Generate normal activity:

```powershell
notepad.exe
calc.exe
cmd.exe
```

Verify logs reach Wazuh Dashboard.

Navigate:

```text
Security Events
```

Search:

```text
rule.groups : sysmon
```

---

# Phase 6: Monitor DLL Injection Indicators

## Sysmon Event ID 8

CreateRemoteThread

Indicates one process attempting to create a thread inside another process.

Investigation Fields:

* Source Process
* Target Process
* User Account
* Parent Process

---

## Sysmon Event ID 10

Process Access

Monitor:

```text
GrantedAccess
TargetImage
SourceImage
```

Suspicious Example:

```text
SourceImage:
powershell.exe

TargetImage:
lsass.exe
```

---

## Sysmon Event ID 7

Image Load

Monitor:

```text
ImageLoaded
Signed
Hashes
```

Look for:

* Unsigned DLLs
* DLLs from Temp folders
* DLLs from User profiles

---

# Phase 7: Create Wazuh Detection Rules

Custom Rule:

```xml
<group name="dll_injection">

<rule id="100500" level="12">

<if_sid>61613</if_sid>

<field name="win.eventdata.TargetImage">
lsass.exe
</field>

<description>
Potential DLL Injection Activity Detected
</description>

<mitre>
<id>T1055</id>
</mitre>

</rule>

</group>
```

Restart Manager:

```bash
sudo systemctl restart wazuh-manager
```

---

# Phase 8: Alert Investigation Workflow

```text
Alert Triggered
       │
       ▼
Review Rule Details
       │
       ▼
Identify Source Process
       │
       ▼
Check Parent Process
       │
       ▼
Review Command Line
       │
       ▼
Analyze User Context
       │
       ▼
Map to MITRE ATT&CK
       │
       ▼
Escalate Incident
```

---

# MITRE ATT&CK Mapping

| Technique | Description       |
| --------- | ----------------- |
| T1055     | Process Injection |
| T1106     | Native API        |
| T1059     | PowerShell        |
| T1003     | Credential Access |
| T1547     | Persistence       |

---

# Sample Dashboard

Create visualizations:

### Top Source Processes

* powershell.exe
* rundll32.exe
* cmd.exe

### Top Target Processes

* lsass.exe
* explorer.exe
* svchost.exe

### Injection Alerts by Host

Track affected endpoints.

### MITRE ATT&CK Heatmap

Visualize attack coverage.

---

# SOC Analyst Investigation Checklist

✅ Verify alert legitimacy

✅ Review parent-child processes

✅ Check process hashes

✅ Validate digital signatures

✅ Identify affected host

✅ Search related alerts

✅ Document findings

✅ Escalate if required

---

# Skills Demonstrated

* SOC Operations
* Threat Detection
* Endpoint Monitoring
* Detection Engineering
* Log Analysis
* Incident Investigation
* Wazuh SIEM
* Sysmon Monitoring
* MITRE ATT&CK Mapping

---

# Author

**Bappy Sharma**
IT Audit • Cybersecurity • SOC Operations •

---

# Project Outcome

Successfully implemented a SOC detection lab capable of identifying suspicious process-access and DLL-loading behaviors, providing visibility into potential Process Injection techniques while supporting threat hunting, incident response, and detection engineering activities.
