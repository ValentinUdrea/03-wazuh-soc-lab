# 03 - Wazuh SOC Lab

A hands-on **Security Operations Center (SOC) lab** built on top of the virtual network and Active Directory environments created in the previous CyberHomeLab projects.

The project focuses on **security monitoring, Windows event collection, detection, investigation, and incident response** using Wazuh.

---

## Lab Architecture

```text
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
        DC01         CLT-WIN11-01            Wazuh
     10.10.20.10      10.10.20.120         10.10.20.110
     Active            Windows 11        Ubuntu Server
     Directory                            Wazuh Manager
```

The network itself was created in the previous CyberHomeLab projects.

---

## Project Goals

The main goal is to build a small but realistic SOC environment where security activity on Windows systems can be:

* collected
* monitored
* detected
* investigated
* documented
* used to improve security detections

The project will gradually move from basic event visibility toward a complete detection and incident-response workflow.

---

## 01. Infrastructure ✅

The infrastructure used by this project was built in the previous CyberHomeLab projects.

Current environment:

* OPNsense firewall
* LAB-LAN: `10.10.20.0/24`
* ATTACK-LAN: `10.10.50.0/24`
* DC01: `10.10.20.10`
* CLT-WIN11-01: `10.10.20.120`
* Ubuntu / Wazuh: `10.10.20.110`
* Kali Linux: `10.10.50.10`

The environment is intentionally segmented between the laboratory network and attacker network.

---

## 02. Logging & Auditing ✅

Windows security auditing has been configured to provide the telemetry required for SOC investigation.

Implemented:

* Windows Advanced Audit Policy
* Windows Security Event Log collection
* Authentication auditing
* Account-related security events
* Process-related security events
* Active Directory security auditing
* Wazuh Agent deployment
* Wazuh endpoint monitoring

Windows security events are successfully being received by Wazuh.

---

## 03.Detection Scenarios 🚧

This phase is currently in progress.

Controlled security scenarios will be performed inside the isolated lab to generate realistic security events and test the detection capabilities of the SOC environment.

Planned scenarios include:

* Failed logons
* Account discovery
* PowerShell activity
* Process creation
* Privilege escalation
* Persistence
* Active Directory changes
* Lateral movement

The purpose of these scenarios is to generate observable activity that can later be investigated from the defensive side.

---

## 04. Wazuh Detection 🚧

Each security scenario will be analyzed from the Wazuh perspective.

The investigation will follow:

```text
Security Activity
       ↓
Windows Event / Artifact
       ↓
Wazuh Agent
       ↓
Wazuh Manager
       ↓
Wazuh Alert
       ↓
Investigation
```

For each scenario, the objective is to determine:

* What happened?
* Which host was involved?
* Which user was involved?
* Which Windows event was generated?
* Did Wazuh detect the activity?
* Which Wazuh rule generated the alert?
* What information is available for investigation?
* What additional evidence would be useful?

---

## 05. Detection Engineering & Tuning 🚧

After testing the native Wazuh detections, the project will be extended with detection engineering and alert tuning.

Planned work includes:

* Custom Wazuh rules
* Alert severity tuning
* Improved alert context
* Clearer event fields
* Source IP and hostname context
* Alert categorization
* Noise reduction
* False-positive reduction
* SOC-focused dashboards

---

## 06. Investigation & Incident Response 🚧

The final stage will focus on developing the investigation process rather than simply identifying individual alerts.

The intended workflow is:

```text
Alert
  ↓
Triage
  ↓
Validation
  ↓
Evidence Collection
  ↓
Timeline Reconstruction
  ↓
Scoping
  ↓
Containment
  ↓
Eradication
  ↓
Recovery
  ↓
Lessons Learned
```

For each incident, the investigation will focus on understanding:

```text
What happened?
      ↓
Where did it happen?
      ↓
When did it happen?
      ↓
Who was involved?
      ↓
What happened before and after?
      ↓
How far did the activity spread?
      ↓
What should be contained?
      ↓
How should the environment be recovered?
```

---

## Investigation Documentation

Each completed scenario will be documented using:

```text
Attack
  ↓
Windows Event / Artifact
  ↓
Wazuh Alert
  ↓
Investigation
  ↓
Detection
  ↓
Mitigation
```

The goal is to document not only the final alert, but also the reasoning used to investigate the activity and determine the appropriate response.

---

## Technologies

* Wazuh
* Windows Server
* Active Directory
* Windows 11
* Ubuntu Server
* OPNsense
* PowerShell
* Windows Event Viewer

---

## Related CyberHomeLab Projects

This project is part of the larger **CyberHomeLab**.

### 01 - Virtual Network

Builds the segmented virtual network, OPNsense firewall, routing, NAT, DHCP, DNS and network isolation.

### 02 - Active Directory PAM Lab

Builds the Windows Active Directory environment with users, groups, OUs, Group Policy, privileged accounts, service accounts and least-privilege administration.

### 03 - Wazuh SOC Lab

Adds security monitoring, Windows telemetry, detection, investigation and incident response capabilities on top of the previous projects.

### 04 - Adversary Simulation Lab

Planned future project focused on the attacker perspective and full attack-path simulation.

---

## Current Status

🚧 **In Progress**

### Completed

* Wazuh server deployed
* Wazuh Dashboard configured
* Windows 11 Wazuh agent deployed
* Active Directory environment integrated
* Windows Advanced Audit Policy configured
* Windows security events successfully ingested into Wazuh
* Wazuh endpoints verified as active
* Initial event visibility confirmed

### Current Focus

* Build the first attack/detection scenario
* Validate Wazuh detection coverage
* Investigate generated security events
* Build the SOC investigation workflow
* Add custom detections and tuning
* Document completed scenarios

---

## Project Status Flow

```text
Infrastructure              ✅
Logging & Auditing          ✅
Attack / Detection         🚧
Wazuh Detection             🚧
Detection Engineering      🚧
Investigation / IR          🚧
Documentation               🚧
```
