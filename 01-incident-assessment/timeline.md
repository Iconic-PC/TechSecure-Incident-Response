# Incident Timeline

## Overview

The following timeline reconstructs the simulated attack lifecycle against `web-server-01.techsecure.local` using the supplied web-server, EDR, Windows event, network, malware, and data-exfiltration evidence.

All timestamps are in **UTC**.

---

## Attack Timeline

| Date / Time             | Event                               | Evidence / Significance                                                                                                                                                |
| ----------------------- | ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **2025-11-05 12:00**    | Reconnaissance begins               | Attacker activity targeting the public-facing environment is recorded in the supplied incident timeline.                                                               |
| **2025-11-05 13:45**    | Exploitation attempt                | Activity consistent with exploitation of the public-facing IIS application begins.                                                                                     |
| **2025-11-05 14:00**    | **Initial compromise**              | Case timeline identifies this as the estimated initial compromise point.                                                                                               |
| **2025-11-05 14:15**    | Web shell execution                 | `shell.aspx` appears in the IIS web root and is associated with execution through the IIS worker process.                                                              |
| **2025-11-05 14:20**    | Command execution                   | `cmd.exe /c whoami` executed through the web shell.                                                                                                                    |
| **2025-11-05 14:25**    | Infrastructure discovery            | DNS activity associated with `attacker-c2.ru` resolving to `185.220.101.45`.                                                                                           |
| **2025-11-05 15:00**    | System reconnaissance               | Attacker performs host, user, process, network, and system discovery.                                                                                                  |
| **2025-11-05 16:00**    | Privilege escalation attempt        | Activity consistent with a UAC bypass is identified in the supplied case evidence.                                                                                     |
| **2025-11-05 17:00**    | Lateral movement / network scanning | Activity consistent with network discovery and possible lateral movement is recorded. No second compromised host is conclusively established by the supplied evidence. |
| **2025-11-05 18:00**    | Persistence established             | Registry Run key `WdiServiceHost` is used to establish persistence.                                                                                                    |
| **2025-11-05 20:00**    | Data collection                     | Sensitive information is collected from `D:\ClientData`.                                                                                                               |
| **2025-11-05 22:00**    | **Data exfiltration begins**        | Large outbound transfers to the external C2 infrastructure begin.                                                                                                      |
| **2025-11-06 01:00**    | **Data exfiltration ends**          | Approximately 2.3 GB of identified data has been transferred externally.                                                                                               |
| **2025-11-06 02:10**    | Web shell modified                  | `shell.aspx` modification timestamp recorded shortly before final detection activity.                                                                                  |
| **2025-11-06 02:12:45** | Suspicious `svchost.exe` execution  | Windows Event ID 4688 records `svchost.exe` as a child of `w3wp.exe`, running as `IUSR_TECHSECURE`.                                                                    |
| **2025-11-06 02:13**    | Persistence registry modification   | Windows Event ID 4657 records modification of the `WdiServiceHost` Run key.                                                                                            |
| **2025-11-06 02:14**    | Web shell accessed                  | Supplied web-shell metadata records an access event shortly before EDR detection.                                                                                      |
| **2025-11-06 02:15**    | **EDR alert generated**             | High-severity Process Execution Anomaly detected on the IIS server.                                                                                                    |
| **2025-11-06 02:15**    | C2 connection terminated            | IDS terminates the identified external connection.                                                                                                                     |
| **2025-11-06 02:30**    | **Incident response activated**     | Security/IR team begins formal response activities.                                                                                                                    |

---

## Attack Lifecycle

The incident can be reconstructed into the following stages:

```text
Reconnaissance
      │
      ▼
Public-Facing IIS Exploitation
      │
      ▼
ASP.NET Web Shell Deployment
      │
      ▼
Command Execution
      │
      ▼
System & Network Discovery
      │
      ▼
Privilege Escalation Attempt
      │
      ▼
Lateral Movement / Network Scanning
      │
      ▼
Persistence
      │
      ▼
Data Collection
      │
      ▼
Data Exfiltration
      │
      ▼
Cobalt Strike Beacon / C2
      │
      ▼
EDR Detection
      │
      ▼
Incident Response
```

---

## Key Timeline Events

### 1. Initial Compromise

The case identifies **14:00 UTC on November 5, 2025** as the estimated initial compromise.

The suspected attack path was exploitation of the public-facing IIS application followed by deployment of an ASP.NET web shell.

The web-shell file metadata records creation at **14:15 UTC**. This distinction is important:

* **14:00 UTC** — estimated initial compromise according to the case timeline.
* **14:15 UTC** — web-shell file creation/execution evidence.

The two timestamps should not be treated as contradictory; the initial exploitation could have occurred before the web shell was written to disk.

---

### 2. Web Shell Command Execution

At approximately **14:20 UTC**, the attacker executed:

```text
cmd.exe /c whoami
```

through the web shell.

Additional commands included:

```text
ipconfig
dir C:\Users
net user
tasklist
systeminfo
netstat -ano
wmic logicaldisk get name
```

These commands demonstrate systematic host reconnaissance.

---

### 3. Persistence

At approximately **02:13 UTC on November 6**, Windows Event ID 4657 recorded modification of the following registry location:

```text
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

The persistence value was:

```text
WdiServiceHost
```

with the associated command:

```text
C:\Windows\System32\svchost.exe -k netsvcs -p -s WdiServiceHost
```

This occurred shortly before the EDR alert and is therefore a high-priority forensic event.

---

### 4. Data Exfiltration

Large-scale data transfer began at approximately:

```text
2025-11-05 22:00 UTC
```

and continued until approximately:

```text
2025-11-06 01:00 UTC
```

The supplied evidence identifies approximately **2.3 GB** of potentially sensitive information transferred externally.

The collected data included:

* Customer database backup
* Client records
* Transaction information
* Credentials
* Application source code

---

### 5. Detection

The decisive detection event occurred at:

```text
2025-11-06 02:15 UTC
```

The EDR identified an anomalous `svchost.exe` process whose parent was:

```text
w3wp.exe
```

This process relationship was highly suspicious because `w3wp.exe` is the IIS worker process and should not normally be spawning an arbitrary Windows service-host process in this context.

The process was also associated with:

```text
IUSR_TECHSECURE
```

and external HTTPS communication with:

```text
185.220.101.45:443
```

---

## Dwell Time

Using the case's estimated initial compromise time of **14:00 UTC on November 5**:

### Time to EDR Detection

```text
Initial compromise: 2025-11-05 14:00
EDR detection:      2025-11-06 02:15

Dwell time:         12 hours 15 minutes
```

### Time to Incident Response Activation

```text
Initial compromise: 2025-11-05 14:00
IR activation:      2025-11-06 02:30

Elapsed time:       12 hours 30 minutes
```

This indicates that the attacker had approximately **12 hours of operational access** before the security team formally activated the incident-response process.

---

## Critical Evidence Sequence

The strongest sequence of events is:

```text
Public-facing IIS application
            │
            ▼
Initial compromise
14:00 UTC
            │
            ▼
ASP.NET web shell
shell.aspx
            │
            ▼
cmd.exe execution
            │
            ▼
Reconnaissance
            │
            ▼
Privilege escalation attempt
            │
            ▼
Persistence
Registry Run Key
            │
            ▼
Sensitive data collection
            │
            ▼
~2.3 GB external transfer
            │
            ▼
Suspicious svchost.exe
            │
            ▼
EDR HIGH alert
02:15 UTC
            │
            ▼
Incident response
02:30 UTC
```

---

## Timeline Assessment

The timeline indicates a multi-stage intrusion rather than an isolated malware execution event.

The attacker appears to have progressed from initial web-application compromise to command execution, reconnaissance, persistence, collection, and data exfiltration before the endpoint detection mechanism identified the anomalous process.

The most significant defensive observation is that **the attacker had already performed substantial post-compromise activity before the final EDR alert**.

This makes the incident a full compromise requiring host containment, forensic investigation, credential review, threat hunting, and validation of the affected web application—not simply removal of the detected `svchost.exe` process.

---

## Investigation Notes

The following distinctions should be preserved during further analysis:

1. **Initial compromise at 14:00 UTC** is based on the supplied case timeline.
2. **Web shell creation at 14:15 UTC** is based on supplied file metadata.
3. **EDR process-level traffic of approximately 2.3 MB** should not be confused with the approximately **2.3 GB incident-wide data transfer** identified in the network evidence.
4. **Lateral movement is indicated but not conclusively proven against a second host.**
5. Threat-actor attribution should be treated separately from technical evidence and should not be based solely on the presence of Cobalt Strike.
