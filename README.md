
# 03 - Wazuh SOC Lab

Wazuh lab for Windows security monitoring and detection.

The lab uses the existing CyberLab network and Active Directory environment and focuses on collecting Windows events, testing detections and investigating the generated data in Wazuh.

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

- Windows event collection
- Security event detection
- PowerShell logging
- File Integrity Monitoring
- Scheduled Task detection
- Basic investigation in Wazuh

All of the above were tested and are working.

---

## 01. Infrastructure 

Wazuh was deployed on an Ubuntu Server and connected to the Windows 11 endpoint.

Current setup:

- OPNsense
- LAB-LAN `10.10.20.0/24`
- ATTACK-LAN `10.10.50.0/24`
- DC01 `10.10.20.10`
- CLT-WIN11-01 `10.10.20.120`
- Wazuh Server `10.10.20.110`
- Kali Linux `10.10.50.10`

The Windows endpoint is running the Wazuh Agent and reporting events to the Wazuh server.

---

## 02. Logging & Auditing 

Windows Advanced Audit Policy was configured and the required event logs were added to the Wazuh Agent.

Collected logs:

- Security
- System
- Application
- PowerShell Operational
- Task Scheduler Operational

The events are visible in Wazuh and can be searched from the dashboard.

---

## 03. Detection Scenarios 

The following events were generated manually on the Windows endpoint and checked in Wazuh:

- `4625` — Failed Logon
- `4688` — Process Creation
- `4698` — Scheduled Task
- `7045` — Windows Service Creation
- `4672` — Special Privileges Assigned
- `4719` — Audit Policy Change
- `4104` — PowerShell Script Block Logging
- File creation
- File deletion
- Account management events

---

## 04. Wazuh Detection 

The event flow was verified from Windows to Wazuh:

~~~text
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

Some of the rules verified during testing:

- Event `4688` → Rule `67027`
- Event `4698` → Rule `60228`
- Event `7045` → Rule `61138`
- File added → Rule `554`

Wazuh also displays MITRE ATT&CK mappings where they are available for the detected event.

A custom rule was added for the scheduled task test:

~~~text
Rule ID: 100101
Level: 10
Description: CyberLab: Scheduled Task persistence detected
MITRE: T1053 - Scheduled Task/Job
~~~

---

## 05. PowerShell Logging & Archives 

PowerShell Script Block Logging was enabled through Local Group Policy.

The Wazuh Agent collects:

~~~text
Microsoft-Windows-PowerShell/Operational
~~~

Event ID `4104` was generated and verified in Wazuh.

Wazuh archives were also enabled so that events can be checked even when they do not generate an alert.

Archive index:

~~~text
wazuh-archives-*
~~~

PowerShell Script Block content was visible in the archive data.

---

## 06. File Integrity Monitoring 

Wazuh Syscheck was tested with a file created in the Windows Startup folder.

Verified:

- File creation detected
- File deletion detected
- Syscheck events received by Wazuh
- Rule `554` triggered for the file creation

---

## 07. Scheduled Task Detection 

A test Scheduled Task was created on the Windows endpoint.

The task:

- appeared in Task Scheduler
- executed after logon
- generated Event ID `4698`
- was received by Wazuh
- triggered Rule `60228`

A custom Wazuh rule was also created for this specific test (`100101`).

---

## Challenges & Troubleshooting ⚠️⚠️⚠️⚠️⚠️

### Network connectivity

At one point, the attack network had no internet access and traffic was not behaving as expected.

I used:

~~~text
ping
nmap
traceroute
nc
~~~

to check where the problem was.

After checking the OPNsense rules, some firewall rules were changed and connectivity started working again.

The lab networks were kept separated while allowing the required internet traffic.

---

### Lab hosts looked unreachable

Nmap showed the lab hosts as up, but the tested ports were filtered:

~~~text
100 filtered tcp ports (no-response)
~~~

I checked the traffic with `traceroute` and `nc` and then checked the firewall rules.

This turned out to be filtering rather than the machines being offline.

---

### PowerShell 4104 not showing in Wazuh

PowerShell was generating Event ID `4104` on Windows, but I couldn't see it in Wazuh.

I first checked Windows with Event Viewer / `Get-WinEvent` and confirmed that the events existed.

Then I checked the Wazuh Agent configuration and added:

~~~text
Microsoft-Windows-PowerShell/Operational
~~~

to `ossec.conf`.

After restarting the agent and running PowerShell commands again, the events appeared in Wazuh.

---

### Task Scheduler events

The Scheduled Task test generated Event `4698`, but I also wanted the Task Scheduler operational events.

I added:

~~~text
Microsoft-Windows-TaskScheduler/Operational
~~~

to the Wazuh Agent configuration.

After that, task start and action events were also available.

---

### Event was collected but no alert

Some events were reaching Wazuh without generating a normal alert.

Instead of only checking `wazuh-alerts-*`, I checked the archives as well.

This showed that the event was actually being collected even when no alert was created.

---

### Archives missing from Discover

Wazuh was already writing archive data on the server, but `wazuh-archives-*` was not showing up in Discover.

I checked the Filebeat configuration and found that archives were disabled.

I enabled:

~~~text
archives:
  enabled: true
~~~

and restarted Filebeat.

After that, the archive index appeared in Discover.

---

### FIM test

For the FIM test, I created a file in the Windows Startup folder using PowerShell.

When searching for the filename, the first event I found was the PowerShell `4104` event.

I checked the event data again and found the Syscheck event separately:

~~~text
event: added
mode: realtime
~~~

The file was then deleted and the deletion was also detected.

---

### Custom Wazuh rule

The Scheduled Task test was already detected by the built-in Rule `60228`.

I added a separate custom rule for the lab:

~~~text
Rule ID: 100101
Level: 10
Description: CyberLab: Scheduled Task persistence detected
~~~

The rule was loaded and triggered correctly.

---

### Linux permissions

Some Wazuh and Filebeat files could not be read with the normal user.

The command returned:

~~~text
Permission denied
~~~

I used `sudo` when checking the configuration and logs.

---

### SSH access

Managing the Ubuntu server from the VM console was inconvenient, so I used SSH instead.

There was initially an authentication error:

~~~text
Permission denied (publickey,password)
~~~

After checking the SSH credentials and connection details, SSH access worked normally.

---

### How I checked problems

When something wasn't showing up in Wazuh, I generally checked it in this order:

~~~text
Windows event
    ↓
Wazuh Agent config
    ↓
Wazuh logs
    ↓
wazuh-alerts-*
    ↓
wazuh-archives-*
~~~

This helped determine whether the problem was with the Windows event itself, the agent, Wazuh processing, or just the alerting.
```


---

## Investigation Documentation

The lab can be followed from the original Windows activity through to the Wazuh alert or archive:

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
Investigation
~~~

The collected data can be used to investigate logons, process execution, PowerShell activity, persistence and file changes.

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

 Completed

The Wazuh SOC Lab is operational and the main objectives have been reached.

Windows events are being collected, tested detections are visible in Wazuh, PowerShell logging and archives are working, File Integrity Monitoring is working and Scheduled Task persistence was successfully detected.

The lab is now usable for Windows security monitoring and SOC investigation.
```
