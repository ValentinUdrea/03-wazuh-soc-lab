
# 03 - Wazuh SOC Lab

A hands-on SOC lab built around Wazuh, Windows and Active Directory.

The project focuses on collecting Windows security telemetry, detecting security-related activity and validating the full logging and detection pipeline.

---

## Lab Architecture

~~~text
                         ATTACK-LAN
                       10.10.50.0/24
                             │
                       Kali Linux
                      10.10.50.10
                             │
                             ▼
                       ┌───────────┐
                       │ OPNsense  │
                       │ Firewall  │
                       └─────┬─────┘
                             │
                      LAB-LAN 10.10.20.0/24
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
        DC01          CLT-WIN11-01           Wazuh
     10.10.20.10      10.10.20.120        10.10.20.110
     Active             Windows 11        Ubuntu Server
     Directory
~~~

---

## Project Goals

The main objective was to build and validate a functional SOC monitoring environment capable of:

- collecting Windows security events
- monitoring endpoint activity
- detecting common security events
- detecting persistence-related activity
- monitoring file changes
- collecting PowerShell activity
- investigating raw events and alerts in Wazuh

The objectives above were successfully achieved.

---

## 01. Infrastructure ✅

The Wazuh SOC environment was deployed on top of the existing CyberLab infrastructure.

Implemented and working:

- OPNsense firewall
- LAB-LAN segmentation
- ATTACK-LAN segmentation
- Ubuntu Server running Wazuh
- Wazuh Dashboard
- Windows 11 endpoint
- Wazuh Agent deployment
- Active Directory environment

Current endpoints:

- DC01 — `10.10.20.10`
- CLT-WIN11-01 — `10.10.20.120`
- Wazuh Server — `10.10.20.110`
- Kali Linux — `10.10.50.10`

---

## 02. Logging & Auditing ✅

Windows security auditing was configured and successfully integrated with Wazuh.

Working telemetry includes:

- Windows Security events
- Windows System events
- Windows Application events
- Authentication events
- Account management events
- Process creation events
- Privilege-related events
- Audit policy changes
- PowerShell Operational events
- Task Scheduler Operational events

Windows events are successfully received by the Wazuh Agent and processed by the Wazuh Manager.

---

## 03. Detection Scenarios ✅

Multiple controlled security scenarios were executed on the Windows endpoint and successfully detected or collected by Wazuh.

Tested events include:

- Failed logon — Event ID `4625`
- Process creation — Event ID `4688`
- Scheduled task creation — Event ID `4698`
- Windows service creation — Event ID `7045`
- Special privileges assigned — Event ID `4672`
- Audit policy change — Event ID `4719`
- PowerShell Script Block Logging — Event ID `4104`
- File creation
- File deletion
- User/account management activity

---

## 04. Wazuh Detection ✅

The detection pipeline was successfully validated:

~~~text
Windows Activity
      ↓
Windows Event
      ↓
Wazuh Agent
      ↓
Wazuh Manager
      ↓
Alert / Archive
      ↓
Wazuh Dashboard
~~~

Examples of verified Wazuh detections:

- Event `4688` → Rule `67027`
- Event `4698` → Rule `60228`
- Event `7045` → Rule `61138`
- File added → Rule `554`

Wazuh also displayed MITRE ATT&CK information for supported detections.

A custom rule was created for the scheduled task persistence test:

~~~text
Rule ID: 100101
Level: 10
Description: CyberLab: Scheduled Task persistence detected
MITRE: T1053 - Scheduled Task/Job
~~~

---

## 05. PowerShell Logging & Archives ✅

PowerShell Script Block Logging was enabled and tested successfully.

The Wazuh Agent collects:

~~~text
Microsoft-Windows-PowerShell/Operational
~~~

Event ID `4104` was successfully generated and received by Wazuh.

Wazuh archives were also enabled to retain events that do not necessarily generate alerts.

Archive data is available through:

~~~text
wazuh-archives-*
~~~

Raw PowerShell Script Block content was successfully observed in the Wazuh archive data.

---

## 06. File Integrity Monitoring ✅

Wazuh File Integrity Monitoring was configured and tested successfully.

Realtime monitoring was verified on the Windows endpoint, including the Startup folder.

Test results:

- File creation detected
- File deletion detected
- Syscheck events received by Wazuh
- Rule `554` triggered for file creation

---

## 07. Scheduled Task Detection ✅

A controlled scheduled task was created to simulate a persistence technique.

The task successfully:

- appeared in Windows Task Scheduler
- executed after logon
- generated Windows Event ID `4698`
- was received by Wazuh
- triggered built-in Wazuh detection Rule `60228`

A custom detection was also created specifically for the lab test using Rule ID `100101`.

---

## Challenges & Troubleshooting

During implementation, several issues were identified and resolved.

### PowerShell 4104 visibility

PowerShell events were confirmed locally in Windows but were initially not visible as Wazuh alerts.

The PowerShell Operational channel was added to the Wazuh Agent configuration and archive logging was enabled.

The `4104` events were subsequently verified in Wazuh archives.

### Wazuh Archives

Wazuh was writing archive data locally, but the archive index was initially not available in Discover.

The Filebeat Wazuh archives module was enabled:

~~~text
archives:
  enabled: true
~~~

After enabling the module, the `wazuh-archives-*` index became available in Discover.

### File Integrity Monitoring

During FIM testing, the PowerShell event responsible for creating the test file was initially found before the actual Syscheck event.

The Syscheck event was subsequently identified and confirmed as the actual file creation detection.

---

## Investigation Documentation

The lab successfully demonstrated the complete path from endpoint activity to security monitoring:

~~~text
Security Activity
      ↓
Windows Event
      ↓
Wazuh Agent
      ↓
Wazuh Manager
      ↓
Alert / Archive
      ↓
Investigation
~~~

The collected events provide the required telemetry to investigate authentication activity, process execution, PowerShell usage, persistence mechanisms, account activity and file changes.

---

## Technologies

- Wazuh
- Wazuh Agent
- Wazuh Manager
- Wazuh Dashboard
- OpenSearch
- Filebeat
- Windows Server
- Active Directory
- Windows 11
- PowerShell
- OPNsense
- Ubuntu Server
- Kali Linux

---

## Current Status

✅ **Completed**

The Wazuh SOC Lab is fully operational.

The project objectives were achieved:

- Windows endpoint successfully integrated with Wazuh
- Security auditing successfully configured
- Windows security events successfully collected
- Detection scenarios successfully tested
- Wazuh native detections verified
- Custom detection rule created
- PowerShell `4104` logging verified
- Wazuh archives configured and verified
- File Integrity Monitoring verified
- Scheduled Task persistence detection verified
- Security events investigated through Wazuh Dashboard

The lab provides a functional environment for Windows security monitoring and SOC-oriented detection work.
```


