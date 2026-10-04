# Initial Findings

## Assessment Summary

The initial assessment indicates that `web-server-01.techsecure.local` (`192.168.1.50`) was compromised through exploitation of a public-facing IIS web application.

The attacker subsequently deployed an ASP.NET web shell, executed operating-system commands, performed system and network reconnaissance, established persistence, collected sensitive information, and transferred approximately 2.3 GB of data to external infrastructure.

The incident is assessed as **HIGH severity** because the available evidence indicates compromise of a production-facing server and potential exposure of customer, financial, credential, and application-source data.

---

## Finding 1 — Confirmed Web Server Compromise

**Severity:** Critical
**Confidence:** High

The affected IIS server should be considered compromised.

Evidence includes:

* ASP.NET web shell located at `C:\inetpub\wwwroot\shell.aspx`
* Command execution through the web shell
* Suspicious `svchost.exe` spawned by `w3wp.exe`
* External C2 communication
* Persistence through a Registry Run key
* Large outbound data transfers

### Assessment

The combination of these indicators is sufficient to treat the host as compromised rather than merely suspicious.

---

## Finding 2 — Public-Facing Application Was the Likely Initial Access Vector

**Severity:** High
**Confidence:** High

The supplied incident timeline identifies exploitation of the public-facing IIS application immediately before the initial compromise.

The subsequent presence of an ASP.NET web shell in:

```text
C:\inetpub\wwwroot\
```

supports the assessment that the web application was used as the initial access pathway.

### Assessment

The precise application vulnerability has not been established from the supplied evidence. Further application and IIS log analysis would be required to determine whether the root cause was:

* insecure file upload,
* command injection,
* remote code execution,
* authentication bypass,
* or another application-layer vulnerability.

The evidence supports **public-facing application exploitation**, but the exact CVE or application vulnerability should not be claimed without additional evidence.

---

## Finding 3 — Web Shell Provided Remote Command Execution

**Severity:** Critical
**Confidence:** High

The file:

```text
C:\inetpub\wwwroot\shell.aspx
```

contains functionality allowing HTTP requests to invoke operating-system commands through `cmd.exe`.

Observed commands include:

```text
whoami
ipconfig
dir C:\Users
net user
tasklist
systeminfo
netstat -ano
wmic logicaldisk get name
```

### Assessment

The web shell provided the attacker with persistent application-level access and the ability to execute commands on the underlying Windows server.

This represents a major loss of host integrity.

---

## Finding 4 — Attacker Performed System Reconnaissance

**Severity:** High
**Confidence:** High

The attacker executed commands to identify:

* Current user context
* Network configuration
* Local users
* Running processes
* Operating-system information
* Network connections
* Available logical drives
* Potentially accessible client data

Examples include:

```text
whoami
ipconfig
net user
tasklist
systeminfo
netstat -ano
wmic logicaldisk get name
```

### Assessment

The command sequence demonstrates deliberate post-compromise reconnaissance rather than accidental or benign process execution.

---

## Finding 5 — Persistence Was Established

**Severity:** Critical
**Confidence:** High

Windows Event ID 4657 recorded modification of:

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

### Assessment

The registry modification indicates an attempt to maintain access across system restarts.

This finding is particularly significant because it occurred shortly before the final EDR detection.

---

## Finding 6 — Suspicious `svchost.exe` Execution

**Severity:** Critical
**Confidence:** High

The EDR alert identified:

```text
svchost.exe
```

with:

```text
Parent Process: w3wp.exe
PID: 2847
User: IUSR_TECHSECURE
```

The process was associated with outbound HTTPS communication to:

```text
185.220.101.45:443
```

### Assessment

The parent-child relationship is anomalous.

`w3wp.exe` is the IIS worker process. A Windows service-host process launched directly from the IIS worker context warrants immediate investigation, particularly when combined with a web shell and external C2 communication.

The case evidence identifies the binary as consistent with a **Cobalt Strike Beacon**.

---

## Finding 7 — Command and Control Communication Identified

**Severity:** High
**Confidence:** High

Network evidence identifies communication between:

```text
192.168.1.50:52847
        ↓
185.220.101.45:443
```

The supplied case evidence also identifies:

```text
attacker-c2.ru
```

as associated C2 infrastructure.

The communication reportedly used HTTPS with periodic beaconing approximately every 60 seconds.

### Assessment

The recurring outbound communication pattern is consistent with command-and-control activity.

However, infrastructure ownership and threat-actor attribution must be assessed separately from the existence of C2 communication.

---

## Finding 8 — Sensitive Data Was Collected and Exfiltrated

**Severity:** Critical
**Confidence:** High

The attacker accessed:

```text
D:\ClientData
```

and collected approximately 2.3 GB of data, including:

* Customer database backup
* Client records
* Transaction information
* Credentials
* Application source code

The main exfiltration window was:

```text
2025-11-05 22:00 UTC
        ↓
2025-11-06 01:00 UTC
```

### Assessment

The incident should be treated as a potential data-breach event pending confirmation of exactly what information was successfully transferred and whether the affected data contains regulated or legally protected information.

---

## Finding 9 — Potential Lateral Movement

**Severity:** High
**Confidence:** Medium

The supplied timeline records activity consistent with:

* Network scanning
* Lateral movement
* SMB/Windows administrative-share activity

However, the available evidence does not conclusively identify a second compromised host.

### Assessment

Lateral movement should therefore remain an **investigative hypothesis**, not a confirmed compromise.

Threat hunting should be performed across internal systems for:

* Authentication events
* SMB connections
* Remote-service execution
* Administrative-share access
* Repeated attacker infrastructure connections
* Matching malware indicators

---

## Finding 10 — Threat Actor Attribution Requires Qualification

**Severity:** High
**Confidence:** Medium

The supplied case identifies the activity as associated with:

```text
APT28 / Fancy Bear
```

and describes the malware as Cobalt Strike Beacon.

### Assessment

APT28 attribution should be treated as **case-provided attribution** rather than independently established attribution.

Cobalt Strike is a widely used legitimate adversary-simulation and penetration-testing platform and is not, by itself, unique to APT28.

Attribution should therefore be based on a combination of:

* Infrastructure
* Malware characteristics
* Tactics, techniques, and procedures
* Victimology
* Historical targeting
* Operational patterns
* Independent threat-intelligence reporting

---

# Initial IOC Set

The following indicators should be added to defensive monitoring and threat-hunting activities.

| Indicator Type | Indicator                                                           | Confidence    |
| -------------- | ------------------------------------------------------------------- | ------------- |
| IP Address     | `185.220.101.45`                                                    | High          |
| Domain         | `attacker-c2.ru`                                                    | High          |
| File           | `C:\inetpub\wwwroot\shell.aspx`                                     | High          |
| Process        | `svchost.exe` launched by `w3wp.exe`                                | High          |
| User           | `IUSR_TECHSECURE`                                                   | Medium        |
| Registry       | `HKLM\Software\Microsoft\Windows\CurrentVersion\Run\WdiServiceHost` | High          |
| C2 Port        | TCP/443                                                             | Medium        |
| Related Domain | `backup-c2.ru`                                                      | Case-provided |
| Related Domain | `staging-c2.ru`                                                     | Case-provided |
| Related IP     | `185.220.102.8`                                                     | Case-provided |
| Related IP     | `185.220.103.25`                                                    | Case-provided |

---

# Initial MITRE ATT&CK Assessment

The observed activity maps to several ATT&CK techniques.

| Tactic               | Technique                              | ID        | Evidence                                |
| -------------------- | -------------------------------------- | --------- | --------------------------------------- |
| Initial Access       | Exploit Public-Facing Application      | T1190     | IIS application compromise              |
| Execution            | Windows Command Shell                  | T1059.003 | `cmd.exe /c` execution                  |
| Persistence          | Registry Run Keys / Startup Folder     | T1547.001 | `WdiServiceHost` Run key                |
| Privilege Escalation | Bypass User Account Control            | T1548.002 | Case timeline                           |
| Defense Evasion      | Masquerading                           | T1036     | Suspicious `svchost.exe` naming/context |
| Discovery            | System Information Discovery           | T1082     | `systeminfo`                            |
| Discovery            | System Network Configuration Discovery | T1016     | `ipconfig`                              |
| Discovery            | Process Discovery                      | T1057     | `tasklist`                              |
| Discovery            | System Owner/User Discovery            | T1033     | `whoami`                                |
| Credential Access    | Credentials from Files                 | T1552.001 | `passwords.txt`                         |
| Collection           | Data from Local System                 | T1005     | Client data collection                  |
| Command and Control  | Web Protocols                          | T1071.001 | HTTPS C2                                |
| Exfiltration         | Exfiltration Over C2 Channel           | T1041     | Large outbound transfer                 |

---

# Risk Assessment

## Confidentiality

**Critical**

Sensitive customer, transaction, credential, and application-source information was accessed and potentially exfiltrated.

## Integrity

**Critical**

The attacker obtained command execution and modified the host by deploying a web shell and establishing persistence.

## Availability

**High**

Although the supplied evidence does not demonstrate prolonged service disruption, compromise of a public-facing production server creates a significant availability and operational risk.

---

# Overall Initial Assessment

The available evidence supports the conclusion that TechSecure Solutions experienced a **high-severity server compromise involving web-application exploitation, web-shell access, command execution, persistence, C2 communication, and data exfiltration**.

The primary affected asset is:

```text
web-server-01.techsecure.local
192.168.1.50
```

The investigation should proceed under the assumption that the host is fully compromised until forensic analysis demonstrates otherwise.

The next investigative priorities are:

1. Preserve the compromised system and associated evidence.
2. Analyze the web shell.
3. Analyze the suspicious executable.
4. Review IIS and Windows logs.
5. Reconstruct network communications.
6. Identify all persistence mechanisms.
7. Determine the exact data accessed and transferred.
8. Hunt for indicators across the wider environment.
9. Validate whether additional hosts were compromised.
10. Determine the root cause of the IIS application compromise.

---

## Analyst Conclusion

The incident was not limited to a single malicious process.

The evidence indicates a complete intrusion lifecycle:

```text
Initial Access
      ↓
Execution
      ↓
Discovery
      ↓
Privilege Escalation
      ↓
Persistence
      ↓
Collection
      ↓
Command & Control
      ↓
Exfiltration
      ↓
Detection
```

This assessment establishes the baseline for the subsequent **Evidence Preservation** and **Forensic Analysis** phases of the investigation.
