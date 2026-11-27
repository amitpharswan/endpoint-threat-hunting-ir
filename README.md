# Endpoint Threat Hunting & Incident Response Platform

A practical multi-endpoint cybersecurity lab for security monitoring, threat detection, endpoint investigation, digital forensics and incident response across Windows and Linux environments.

## Overview

This project combines **Splunk** and **Velociraptor** to build a centralized security monitoring and endpoint investigation environment.

Using Kali Linux, controlled attack scenarios are executed against Windows 11 ARM64 and Ubuntu Linux ARM64 endpoints. The resulting activity is monitored in Splunk, investigated using Velociraptor, and analyzed through a complete security operations workflow.

Each attack is investigated individually, from execution and detection through forensic evidence collection, containment, recovery and incident reporting.

## Architecture

The lab consists of four machines running on an Apple Silicon Mac environment using UTM virtual machines.

| Machine | Role | Primary Components |
|---|---|---|
| macOS (Apple M1) | Central security server | Splunk , Velociraptor Server |
| Windows 11 ARM64 | Target endpoint | Splunk Universal Forwarder, Sysmon, Windows Security Auditing, PowerShell Logging, Velociraptor Client |
| Ubuntu Linux ARM64 | Target endpoint | Splunk Universal Forwarder, auditd, Velociraptor Client |
| Kali Linux | Adversary simulation | Netcat, Python, SSH testing utilities, Hydra and other scenario-specific tools |

Kali Linux is used exclusively for controlled attack simulation. It is not configured as a monitored target endpoint.

## Technology Stack

| Technology | Purpose |
|---|---|
| Splunk  | Centralized log collection, analysis, correlation, detection and alerting |
| Splunk Universal Forwarder | Collects and forwards configured endpoint logs |
| Velociraptor | Endpoint investigation, threat hunting and forensic evidence collection |
| Sysmon | Detailed Windows process, network, registry and file activity telemetry |
| Windows Security Auditing | Security events, process creation, authentication and account activity |
| PowerShell Logging | Visibility into PowerShell execution and script activity |
| Linux auditd | Security auditing, process execution and selected system changes |
| Kali Linux | Controlled adversary and attack simulation |
| MITRE ATT&CK | Classification and mapping of simulated adversary techniques |

## How It Works

The platform uses two complementary security technologies.

**Splunk — Security Monitoring and Detection**

Splunk receives endpoint telemetry through Universal Forwarders, indexes the events and makes them available for investigation.

Security analysts use Search Processing Language (SPL) to identify suspicious activity, correlate events, develop detection logic and create alerts and dashboards.

**Velociraptor — Endpoint Investigation and Threat Hunting**

Velociraptor provides visibility into monitored endpoints through its server-client architecture.

Using VQL and forensic artifacts, analysts can collect endpoint information, investigate processes and network connections, examine persistence mechanisms and preserve relevant forensic evidence.

**Combined Investigation Workflow**

1. An attack is initiated from Kali Linux.
2. The target endpoint generates security telemetry.
3. Sysmon, Windows auditing or Linux auditd records relevant activity.
4. Splunk Universal Forwarder forwards the configured logs to Splunk .
5. SPL searches identify and investigate suspicious activity.
6. Velociraptor collects additional endpoint information.
7. Findings from both platforms are correlated.
8. Forensic evidence is collected and preserved.
9. Appropriate containment and incident response actions are performed.
10. The endpoint is recovered and the investigation is documented.

## Attack Scenarios

The project includes controlled attack simulations across Windows and Linux.

### Windows Scenarios

| ID | Attack Scenario | MITRE ATT&CK |
|---|---|---|
| WIN-01 | Encoded PowerShell Execution | T1059.001 |
| WIN-02 | System and User Discovery | T1082, T1033 |
| WIN-03 | Scheduled Task Persistence | T1053.005 |
| WIN-04 | Registry Run Key Persistence | T1547.001 |
| WIN-05 | Controlled C2 Callback | T1071.001, where applicable |

### Linux Scenarios

| ID | Attack Scenario | MITRE ATT&CK |
|---|---|---|
| LNX-01 | SSH Authentication Brute-Force Simulation | T1110.001 |
| LNX-02 | Controlled Shell Execution and Callback | T1059.004, where applicable |
| LNX-03 | Cron-Based Persistence | T1053.003 |
| LNX-04 | Unauthorized SSH Key Modification | T1098.004 |

Final technique mappings are verified against the actual execution methods.

## Investigation and Incident Response

Every attack follows the same structured investigation lifecycle:

1. **Attack Understanding:** Study the attack, its purpose, execution method and potential impact.
2. **Environment Preparation:** Identify the attacker, target, prerequisites, permissions and expected telemetry.
3. **Attack Execution:** Perform the controlled simulation from Kali Linux.
4. **Attack Verification:** Confirm the activity and its effects on the target endpoint.
5. **SIEM Investigation:** Examine generated events and identify relevant telemetry in Splunk.
6. **Detection Engineering:** Develop, test and validate SPL detection searches.
7. **Alerting:** Configure and test alerts for suitable detection scenarios.
8. **Endpoint Investigation:** Use Velociraptor artifacts and VQL to investigate suspicious endpoint activity.
9. **Event Correlation:** Correlate SIEM and endpoint findings to establish an investigation timeline.
10. **Forensic Evidence Collection:** Preserve relevant logs, artifacts and investigation results with SHA-256 hashes.
11. **Containment and Response:** Perform appropriate response actions and remove simulated attack artifacts.
12. **Recovery:** Verify that the endpoint and security monitoring components are operating normally.
13. **Incident Reporting:** Document the investigation, findings, evidence and response.

Each scenario must be completed and documented before moving to the next.

## Detection Engineering

Detection logic is developed using telemetry generated by the controlled attack scenarios.

The project focuses on:

- Windows process execution and PowerShell activity.
- Suspicious process and parent-child relationships.
- Scheduled task creation.
- Registry persistence modifications.
- Linux authentication failures.
- Cron configuration changes.
- SSH authorized-key modifications.
- Suspicious network connections.

Detection quality is evaluated through benign and simulated malicious activity.

False-positive tuning includes examining user context, process information, command lines, network activity and authorized administrative behavior.

Every detection must be tested, and its limitations must be documented.

## Dashboards and Alerting

Splunk dashboards will provide visibility into:

- Endpoint activity.
- Windows process execution.
- PowerShell activity.
- Authentication failures.
- Scheduled task creation.
- Registry persistence indicators.
- Linux SSH authentication activity.
- Linux cron modifications.
- Suspicious network connections.
- Detection alerts.

Alerts are created for validated detection scenarios, with documented search logic, schedules, trigger conditions, severity and test results.

## Repository Structure

```text
endpoint-threat-hunting-ir-platform/
│
├── README.md
│
├── architecture/
│   ├── architecture.md
│   ├── technology-stack.md
│   ├── network-topology.md
│   ├── data-flow.md
│   └── lab_topology.png
│
├── setup/
│   ├── splunk-setup.md
│   ├── velociraptor-server-setup.md
│   ├── windows-endpoint-setup.md
│   ├── ubuntu-endpoint-setup.md
│   ├── kali-setup.md
│   ├── telemetry-validation.md
│   └── baseline-results.md
│
├── attack-scenarios/
│   ├── attack-scenario-catalogue.md
│   ├── windows/
│   │   ├── WIN-01-encoded-powershell.md
│   │   ├── WIN-02-system-discovery.md
│   │   ├── WIN-03-scheduled-task.md
│   │   ├── WIN-04-registry-run-key.md
│   │   └── WIN-05-c2-callback.md
│   └── linux/
│       ├── LNX-01-ssh-brute-force.md
│       ├── LNX-02-shell-callback.md
│       ├── LNX-03-cron-persistence.md
│       └── LNX-04-ssh-authorized-keys.md
│
├── detections/
│   ├── splunk-detections.md
│   ├── velociraptor-hunts.md
│   ├── false-positive-tuning.md
│   └── validation-results.md
│
├── investigation-reports/
│   ├── windows/
│   └── linux/
│
├── evidence/
│   ├── evidence-inventory.md
│   ├── hashes/
│   ├── windows/
│   └── linux/
│
├── dashboards/
│   ├── dashboard-documentation.md
│   └── screenshots/
│
├── screenshots/
│   ├── architecture/
│   ├── deployment/
│   ├── attack-execution/
│   ├── splunk-detection/
│   ├── velociraptor-investigation/
│   └── incident-response/
│
└── final-report/
    ├── project-summary.md
    ├── lessons-learned.md
    └── limitations-and-future-work.md
```

## Project Phases

The implementation follows seven structured phases:

| Phase | Description |
|---|---|
| Phase 0 | Architecture and Technology Fundamentals |
| Phase 1 | Environment Preparation and Deployment |
| Phase 2 | Communication, Telemetry and Baseline Validation |
| Phase 3 | Attack Scenario Design |
| Phase 4 | Individual Attack Execution and Complete SOC Lifecycle |
| Phase 5 | Detection Engineering and False-Positive Tuning |
| Phase 6 | Dashboards, Alerting and Project Consolidation |

Each phase has defined deliverables and completion criteria. Attack execution begins only after the environment and telemetry have been validated.

## Safety and Lab Scope

This project is designed exclusively for controlled cybersecurity testing within an isolated virtual lab.

- Only authorized lab machines are targeted.
- Dedicated test accounts are used.
- Real credentials and third-party systems are not targeted.
- Attack simulations use controlled, non-destructive methods.
- Evidence is preserved before cleanup.
- Response and recovery actions are verified.
- Credentials, private keys and sensitive forensic data are excluded from the public repository.

## Project Scope and Limitations

This project demonstrates selected  security operations, threat hunting, endpoint investigation and incident response workflows.

It is not a production EDR deployment and is not intended to replace commercial endpoint detection and response platforms such as CrowdStrike Falcon.

Splunk provides centralized security monitoring and detection, while Velociraptor provides endpoint investigation, forensic collection and supported remediation capabilities.

Together, they demonstrate selected capabilities associated with broader EDR and DFIR workflows.

## Official References

- [Splunk Documentation](https://docs.splunk.com/Documentation/Splunk/latest)
- [Splunk Help](https://help.splunk.com/)
- [Velociraptor Documentation](https://docs.velociraptor.app/)
- [Velociraptor GitHub](https://github.com/Velocidex/velociraptor)
- [Microsoft Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Linux auditd Manual](https://man7.org/linux/man-pages/man8/auditd.8.html)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Kali Linux Documentation](https://www.kali.org/docs/)

## Final Outcome

The final deliverable is a working, tested and documented multi-endpoint security investigation environment.

It demonstrates how controlled adversary activity can be monitored, detected, investigated and responded to using centralized SIEM monitoring and endpoint forensic investigation.

The project is supported by reproducible attack scenarios, validated SPL detections, Velociraptor investigation procedures, preserved forensic evidence, individual incident reports, security dashboards and technical documentation.
