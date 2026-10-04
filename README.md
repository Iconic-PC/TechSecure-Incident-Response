# TechSecure Incident Response & Digital Forensics

> **Simulated Incident Response Case Study**

A simulated incident-response and digital-forensics investigation involving the compromise of a public-facing Microsoft IIS web server.

The investigation reconstructs an attack involving web-application exploitation, ASP.NET web-shell deployment, Windows command execution, persistence, command-and-control communication, data collection, and approximately 2.3 GB of identified data exfiltration.

## Overview

**Organization:** TechSecure Solutions
**Incident ID:** TECHSECURE-2025-001
**Affected Host:** `web-server-01.techsecure.local`
**IP Address:** `192.168.1.50`
**Severity:** High
**Incident Date:** November 5–6, 2025
**Investigation Type:** Incident Response & Digital Forensics
**Status:** Simulated Case Study

## Investigation Objective

The objective of this investigation was to:

* Determine how the system was compromised.
* Establish a reliable incident timeline.
* Identify attacker activity and persistence mechanisms.
* Analyze malware and command-and-control indicators.
* Identify potentially compromised or exfiltrated information.
* Enrich indicators using threat intelligence.
* Map observed activity to MITRE ATT&CK.
* Assess business and security impact.
* Develop containment, remediation, detection, and prevention recommendations.

## Key Findings

The investigation identified:

* Compromise of a public-facing IIS server.
* Deployment of an ASP.NET web shell.
* Remote command execution through `cmd.exe`.
* Reconnaissance of the compromised Windows system.
* Activity consistent with privilege escalation and lateral movement.
* Registry-based persistence.
* Suspicious execution of `svchost.exe` from an IIS process context.
* HTTPS command-and-control communication.
* Periodic beaconing to `185.220.101.45:443`.
* Approximately 2.3 GB of identified external data transfer.
* Exposure of customer, financial, credential, and source-code data.
* Malware characteristics identified in the supplied case evidence as consistent with Cobalt Strike Beacon.

## Attack Chain

```text
Reconnaissance
       ↓
Public-Facing IIS Exploitation
       ↓
ASP.NET Web Shell
       ↓
Command Execution
       ↓
System Discovery
       ↓
Privilege Escalation Attempt
       ↓
Lateral-Movement Activity
       ↓
Registry Persistence
       ↓
Data Collection
       ↓
C2 Communication
       ↓
Data Exfiltration
       ↓
EDR Detection
```

## Evidence Sources

The investigation was based on a supplied simulated evidence package containing:

* Memory-dump information
* Disk-image information
* IIS logs
* Network PCAP information
* Malware sample information
* Windows event information
* YARA detection results
* Network and DNS indicators
* Threat-intelligence information
* Incident timeline data

## Important Evidence Disclaimer

This is a **simulated educational investigation**.

The evidence used in this project was supplied as part of a practical training exercise. No claim is made that the analyst independently acquired the original memory image, disk image, network capture, or other forensic evidence from a live production environment.

Where evidence was supplied by the case rather than independently validated, it is explicitly identified as **case-provided**.

## Investigation Frameworks

The investigation uses concepts from:

* NIST Incident Response
* SANS Incident Response
* MITRE ATT&CK
* Digital-forensics principles
* Threat-intelligence analysis

## Project Structure

| Directory                  | Purpose                                                  |
| -------------------------- | -------------------------------------------------------- |
| `01-incident-assessment`   | Initial triage, scope, severity and timeline             |
| `02-evidence-preservation` | Evidence inventory, preservation and imaging             |
| `03-forensic-analysis`     | Malware, web-shell, network and log analysis             |
| `04-threat-intelligence`   | IOC enrichment, threat actor analysis and ATT&CK mapping |
| `05-incident-response`     | Executive report, remediation and detection strategy     |
| `diagrams`                 | Investigation and attack-chain visualizations            |
| `references`               | External sources and research references                 |

## Major Indicators

| Type               | Indicator                                            |
| ------------------ | ---------------------------------------------------- |
| C2 IP              | `185.220.101.45`                                     |
| C2 Domain          | `attacker-c2.ru`                                     |
| Web Shell          | `shell.aspx`                                         |
| Suspicious Process | `svchost.exe`                                        |
| Parent Process     | `w3wp.exe`                                           |
| Persistence        | `WdiServiceHost`                                     |
| Registry           | `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` |
| Web Identity       | `IUSR_TECHSECURE`                                    |
| Exfiltration       | Approximately 2.3 GB                                 |

## Outcome

The investigation concluded that the simulated environment experienced a high-severity compromise involving unauthorized web-server access, command execution, persistence, C2 activity, sensitive-data collection, and external data transfer.

The recommended response focuses on:

1. Containment and evidence preservation
2. Eradication of attacker persistence
3. Credential rotation
4. Investigation of potential lateral movement
5. Rebuilding affected systems from known-clean sources
6. Vulnerability remediation
7. Improved endpoint and network detection
8. Stronger access control and segmentation
9. Continuous threat hunting and monitoring

## Disclaimer

This project is intended for educational and portfolio purposes. All organization names, systems, evidence, indicators, and incident details should be understood within the context of the supplied simulated case unless explicitly identified as externally validated threat intelligence.
