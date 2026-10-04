# Incident Overview

## Incident Identification

| Field                            | Details                                         |
| -------------------------------- | ----------------------------------------------- |
| **Incident ID**                  | TECHSECURE-2025-001                             |
| **Organization**                 | TechSecure Solutions                            |
| **Affected System**              | `web-server-01.techsecure.local`                |
| **IP Address**                   | `192.168.1.50`                                  |
| **Incident Severity**            | **HIGH**                                        |
| **Estimated Initial Compromise** | 2025-11-05 14:00 UTC                            |
| **Detection**                    | 2025-11-06 02:15 UTC                            |
| **Incident Response Activated**  | 2025-11-06 02:30 UTC                            |
| **Investigation Status**         | Under Investigation                             |
| **Investigation Type**           | Simulated Incident Response & Digital Forensics |

---

## Incident Description

TechSecure Solutions experienced a simulated high-severity cybersecurity incident involving the compromise of a public-facing Microsoft IIS web server.

The supplied case evidence indicates that the attacker initially compromised the web server at approximately **14:00 UTC on November 5, 2025** following exploitation of the public-facing web application.

An ASP.NET web shell named `shell.aspx` was subsequently deployed within the IIS web root:

```text
C:\inetpub\wwwroot\shell.aspx
```

The web shell provided the attacker with remote command-execution capabilities through `cmd.exe`.

Following initial access, the attacker performed system reconnaissance, attempted privilege escalation, conducted activity consistent with lateral movement, established persistence, collected sensitive information, and communicated with external command-and-control infrastructure.

---

## Initial Access

The initial access vector was assessed as exploitation of the public-facing IIS web application.

The attacker deployed an ASP.NET web shell that accepted commands through a query-string parameter:

```text
/shell.aspx?cmd=<command>
```

The supplied web-shell evidence indicates that commands were executed through:

```text
cmd.exe /c
```

The web shell was associated with the IIS worker process:

```text
w3wp.exe
```

and executed under the IIS application identity:

```text
IUSR_TECHSECURE
```

---

## Confirmed Malicious Activity

The investigation identified multiple stages of attacker activity.

### Web Shell Activity

The web shell was used to execute commands including:

```text
whoami
ipconfig
dir C:\Users
net user
tasklist
systeminfo
netstat -ano
wmic logicaldisk get name
dir D:\ClientData
copy D:\ClientData\*.*
```

These commands indicate progressive reconnaissance of the compromised system, users, processes, network configuration, storage, and potentially sensitive data.

### Persistence

Persistence was established through the Windows Registry Run key:

```text
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

with the value:

```text
WdiServiceHost
```

pointing to:

```text
C:\Windows\System32\svchost.exe -k netsvcs -p -s WdiServiceHost
```

### Suspicious Process Execution

A suspicious `svchost.exe` process was executed as a child of the IIS worker process:

```text
w3wp.exe
    └── svchost.exe
```

This parent-child relationship was identified as anomalous and generated the EDR detection.

The process was executed under:

```text
IUSR_TECHSECURE
```

rather than through a normal Windows service-management context.

---

## Command and Control

Network evidence identified communication between the compromised server and:

```text
192.168.1.50:52847
        ↓
185.220.101.45:443
```

The communication used TCP/HTTPS and exhibited periodic beaconing at approximately 60-second intervals.

The associated domain was:

```text
attacker-c2.ru
```

DNS evidence showed the domain resolving to:

```text
185.220.101.45
```

The case evidence identifies the malware as consistent with a **Cobalt Strike Beacon**.

---

## Data Collection and Exfiltration

The investigation identified collection of sensitive information from:

```text
D:\ClientData
```

The supplied evidence identifies the following files:

| Data                        | Approximate Size |
| --------------------------- | ---------------: |
| `ClientData_2025-11-05.bak` |           1.2 GB |
| `clients.xlsx`              |            45 MB |
| `transactions_2025.csv`     |           320 MB |
| `passwords.txt`             |           2.3 MB |
| `application_source.zip`    |           850 MB |
| **Total**                   |      **~2.3 GB** |

Network evidence indicates approximately **2.3 GB of data was transferred externally** through the identified C2 connection.

The exfiltration period was approximately:

```text
2025-11-05 22:00 UTC
        ↓
2025-11-06 01:00 UTC
```

---

## Detection

The primary EDR detection occurred at:

```text
2025-11-06 02:15 UTC
```

The alert identified anomalous execution of:

```text
svchost.exe
```

with:

```text
Parent Process: w3wp.exe
PID: 2847
User: IUSR_TECHSECURE
```

The unusual process relationship, web-application context, command line, and external network communication contributed to the high-severity detection.

Incident response activities were activated at:

```text
2025-11-06 02:30 UTC
```

---

## Initial Scope Assessment

### Confirmed Compromised System

```text
web-server-01.techsecure.local
192.168.1.50
```

The evidence confirms malicious activity on this host.

### Potential Additional Compromise

The supplied timeline indicates activity consistent with lateral movement and network scanning.

However, the supplied evidence does **not conclusively identify a second compromised host**.

Therefore:

> Lateral movement is treated as an investigative finding requiring further validation rather than confirmation of additional system compromise.

---

## Initial Impact Assessment

The incident potentially affected:

* Customer database information
* Client contact information
* Financial records
* Employee credentials
* Application source code
* Production web-server availability and integrity

The incident therefore represents a significant confidentiality, integrity, and operational risk.

---

## Investigation Priorities

The investigation should prioritize:

1. Containing the compromised web server.
2. Preserving forensic evidence.
3. Determining the original web-application vulnerability.
4. Investigating the ASP.NET web shell.
5. Analyzing the suspicious `svchost.exe`.
6. Identifying all persistence mechanisms.
7. Determining the full extent of data access and exfiltration.
8. Investigating potential lateral movement.
9. Hunting for the identified IOCs across the environment.
10. Restoring affected systems from known-clean sources.

---

## Evidence Qualification

This project is based on **simulated educational evidence supplied as part of a practical incident-response exercise**.

Evidence that was supplied directly by the case is identified as case-provided. No claim is made that the analyst independently acquired the original memory image, disk image, network PCAP, or malware sample from a live production environment.

The supplied MD5 and SHA-256 strings for the malware and web shell are retained as case-provided indicators. They contain non-hexadecimal characters and were therefore not independently validated as standard cryptographic hashes.
