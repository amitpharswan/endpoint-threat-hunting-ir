# Technology Stack

## 1. Purpose

This document describes the technologies used in the   Endpoint Threat Hunting & Incident Response Platform.

For each technology, it records:

- What it is.
- Why the project needs it.
- Where it will be installed or used.
- What it monitors or manages.
- What data it generates or collects.
- How it communicates.
- How its operation will be verified.
- Its limitations.

**Status:** Planned technology stack. Installation, configuration, version compatibility, and operational verification remain subject to Phase 1 and Phase 2.

## 2. Technology Overview

| Technology | Deployment location | Primary responsibility |
|---|---|---|
| Splunk   | MacBook Air M1 | Central SIEM and log analysis |
| Splunk Universal Forwarder | Windows and Ubuntu | Endpoint log collection and forwarding |
| Sysmon | Windows 11 ARM64 | Detailed Windows telemetry |
| Windows Security Auditing | Windows 11 ARM64 | Security and authentication events |
| PowerShell Logging | Windows 11 ARM64 | PowerShell activity visibility |
| Linux auditd | Ubuntu Linux ARM64 | Linux security auditing |
| Velociraptor Server | MacBook Air M1 | Endpoint investigation management |
| Velociraptor Client | Windows and Ubuntu | Endpoint collection and investigation |
| VQL and Artifacts | Velociraptor platform | Querying and reusable collection workflows |
| Kali Linux utilities | Kali Linux | Controlled adversary simulation |

---

## 3. Splunk  

### What it is

Splunk   is the central Security Information and Event Management (SIEM) platform in this lab.

### Why the project needs it

It provides centralized security monitoring and analysis across the Windows and Ubuntu endpoints.

### Installation location

MacBook Air M1 running macOS.

### What it monitors or manages

- Forwarded Windows security telemetry.
- Forwarded Ubuntu authentication and system logs.
- Forwarded Sysmon events.
- Configured Linux audit logs.
- Activity from multiple endpoints for correlation.

### Data handled

- Windows Security, System, PowerShell, and Sysmon events.
- Ubuntu authentication, syslog, and audit events.
- Search results and detection findings.
- Alert information and dashboard results.

### Communication

Endpoint Universal Forwarders send configured data to the Splunk receiving service on the Mac.

The planned receiving port is TCP 9997, subject to verification against the existing Splunk configuration.

### Verification

- Confirm the Splunk service is running.
- Confirm the web interface is accessible.
- Verify the receiving service is enabled.
- Confirm endpoint data is arriving.
- Search for Windows and Ubuntu events.
- Validate that expected fields and timestamps are available.
- Execute test SPL searches.

### Limitations

- Splunk does not automatically determine whether activity is malicious.
- Detection logic must be developed and validated.
- Missing endpoint telemetry cannot be recovered by a search.
- Detection quality depends on data quality, source configuration, field availability, and search logic.

---

## 4. Splunk Universal Forwarder

### What it is

Splunk Universal Forwarder is the endpoint log collection and forwarding component.

### Why the project needs it

It transfers configured endpoint telemetry to the central Splunk   instance.

### Installation location

- Windows 11 ARM64.
- Ubuntu Linux ARM64.

It will not be installed on Kali for this project.

### What it monitors or manages

It collects configured log sources rather than independently classifying activity as malicious.

Windows sources:

- Security events.
- System events.
- PowerShell events.
- Sysmon events.

Ubuntu sources:

- Authentication logs.
- System logs.
- auditd logs.

### Data handled

The forwarder collects and forwards configured event and log data to Splunk  .

### Communication

Windows and Ubuntu Universal Forwarders communicate with the Mac Splunk receiver.

Planned destination: Mac Splunk  , TCP 9997, subject to configuration verification.

### Verification

- Confirm the forwarder service is running.
- Check its configured destination.
- Verify connectivity to the Splunk receiver.
- Confirm the endpoint appears in received data.
- Search for events from each endpoint.
- Confirm expected source, host, and timestamp information.

### Limitations

- It only collects configured sources.
- Incorrect input configuration can result in missing telemetry.
- It does not replace Sysmon, Windows auditing, or Linux auditd.
- It does not independently perform full SIEM detection and investigation.

---

## 5. Sysmon

### What it is

Sysmon is a Windows system monitoring component that provides detailed system activity telemetry.

### Why the project needs it

It provides process and other configured event information needed for threat hunting and behavioural investigation.

### Installation location

Windows 11 ARM64.

### What it monitors

Depending on the selected configuration:

- Process creation.
- Network connections.
- Process termination.
- Process access.
- File creation.
- Registry activity.
- DNS queries.

### Data generated

Important planned event examples:

| Event ID | Activity |
|---|---|
| 1 | Process creation |
| 3 | Network connection |
| 5 | Process termination |
| 10 | Process access |
| 11 | File creation |
| 12–14 | Registry activity |
| 22 | DNS query |

Process information may include:

- Process name.
- Command line.
- Parent process.
- Process identifiers.
- Process GUID.
- Other configured execution details.

### Communication

Sysmon writes events to its Windows event log channel.

Splunk Universal Forwarder collects the configured Sysmon event source and forwards it to Splunk  .

### Verification

- Confirm Sysmon is installed and running.
- Inspect the active Sysmon configuration.
- Generate harmless test process activity.
- Confirm the expected events appear in Windows Event Viewer.
- Confirm those events reach Splunk.
- Validate relevant fields such as process name, command line, and parent process.

### Limitations

- Available telemetry depends on the active Sysmon configuration.
- Sysmon records activity; it does not independently classify activity as malicious.
- Unconfigured event categories will not provide the expected visibility.
- Event availability must be validated against the actual operating-system and Sysmon versions.

---

## 6. Windows Security Auditing

### What it is

Windows Security Auditing records security-related operating-system activity according to enabled audit policies.

### Why the project needs it

It supplements Sysmon with authentication, account, process, and other security events.

### Installation location

Windows 11 ARM64, using the operating system's auditing facilities.

### What it monitors

Depending on enabled audit policies:

- Process creation.
- Successful logons.
- Failed logons.
- Scheduled task creation.
- User account creation.
- Other configured security activity.

### Data generated

Example event IDs:

| Event ID | Activity |
|---|---|
| 4688 | Process creation |
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4698 | Scheduled task creation |
| 4720 | User account creation |

### Communication

Windows records security events in its Security event log.

Splunk Universal Forwarder collects the configured Security log and forwards it to Splunk  .

### Verification

- Inspect enabled audit policies.
- Generate harmless test activity.
- Confirm expected events appear in the Security event log.
- Confirm the events reach Splunk.
- Validate event IDs, timestamps, users, and other relevant fields.

### Limitations

- Event availability depends on enabled audit policies.
- Not every activity automatically generates every example event.
- Audit policy and operating-system configuration must be checked before relying on an event for detection.

---

## 7. PowerShell Logging

### What it is

PowerShell Logging provides additional visibility into PowerShell activity.

### Why the project needs it

It supports investigation of suspicious PowerShell execution, including controlled encoded PowerShell scenarios.

### Installation location

Windows 11 ARM64, using the Windows PowerShell logging facilities.

### What it monitors

The project plans to evaluate:

- Script Block Logging.
- Module Logging.

### Data generated

Configured PowerShell logging events can provide information about script activity and module operations.

These events supplement process creation telemetry from Sysmon and Windows Security Auditing.

### Communication

PowerShell writes configured logging events to Windows event logs.

Splunk Universal Forwarder collects the configured event sources and forwards them to Splunk  .

### Verification

- Confirm the intended logging policies are enabled.
- Execute harmless PowerShell test commands.
- Inspect the relevant Windows event logs.
- Confirm the events reach Splunk.
- Check whether the available event content supports the planned investigation.

### Limitations

- Visibility depends on the logging policies enabled.
- Process creation telemetry and PowerShell script logging provide different information.
- Missing logging configuration can leave gaps in investigation evidence.

---

## 8. Linux auditd

### What it is

auditd provides Linux security auditing and records activity covered by configured audit rules.

### Why the project needs it

It provides visibility into selected security-relevant activity on the Ubuntu endpoint.

### Installation location

Ubuntu Linux ARM64.

### What it monitors

Initial monitoring areas:

- Process execution through execve-related audit rules.
- Cron configuration changes.
- SSH configuration changes.
- Authorized-key modifications.
- Selected privileged file modifications.

Audit rules will be designed, configured, and tested individually.

### Data generated

auditd produces audit records describing activity covered by the active rules.

For example, a properly configured rule can record relevant activity when a user modifies a scheduled task configuration.

### Communication

auditd writes audit records to the configured audit log.

Splunk Universal Forwarder collects the configured audit log source and forwards the records to Splunk  .

### Verification

- Confirm auditd is installed and running.
- Inspect active audit rules.
- Perform harmless test activity covered by a rule.
- Confirm the expected audit records are generated.
- Confirm the records reach Splunk.
- Validate timestamps and relevant event information.

### Limitations

- auditd records activity covered by configured rules.
- Missing or incorrect rules create telemetry gaps.
- Audit rules must be tested individually.
- Actual log paths and configuration depend on the installed Ubuntu version.

---

## 9. Velociraptor Server

### What it is

Velociraptor Server is the central management and investigation component of the Velociraptor platform.

### Why the project needs it

It manages endpoint clients and provides centralized endpoint investigation, artifact collection, and hunt management.

### Installation location

MacBook Air M1 running macOS.

### What it manages

- Connected endpoint clients.
- Administrative console access.
- Artifact collections.
- Fleet hunts.
- Investigation results.

### Data handled

- Client information.
- Artifact collection results.
- Endpoint investigation findings.
- Hunt results.
- Other configured investigation data.

### Communication

The server communicates with enrolled Velociraptor clients.

The proposed frontend port is TCP 8002, subject to actual configuration verification.

The administrative GUI is proposed on TCP 8889.

### Verification

- Confirm the server starts successfully.
- Inspect server logs.
- Verify expected listening services.
- Access the administrative GUI.
- Confirm endpoint clients can enroll and communicate.
- Run a basic authorized artifact collection.
- Confirm results are returned and stored.

### Limitations

- Investigation results depend on client connectivity and collection scope.
- Artifacts must be selected appropriately for the investigation.
- Server configuration, certificates, and client configuration must agree.
- Sensitive configuration files and keys must be protected.

---

## 10. Velociraptor Client

### What it is

The Velociraptor Client is the endpoint component that communicates with the Velociraptor Server and performs authorized collections.

### Why the project needs it

It provides endpoint-level information for deeper investigation after suspicious activity is identified.

### Installation location

- Windows 11 ARM64.
- Ubuntu Linux ARM64.

It will not be installed on Kali for this project.

### What it collects

Depending on the selected artifacts and permissions:

- Process information.
- Network information.
- Persistence information.
- Event logs.
- Cron and SSH configuration information.
- Other authorized forensic data.

### Communication

The client communicates with the central Velociraptor Server through its configured frontend address and port.

The proposed frontend port is TCP 8002, subject to verification.

### Verification

- Confirm the client service is running.
- Confirm the client appears in the server console.
- Verify hostname and operating-system information.
- Confirm communication is stable.
- Run basic authorized artifact collections.
- Review returned results.

### Limitations

- Collection results depend on the selected artifact and available endpoint data.
- Some collections require appropriate permissions.
- Client connectivity is required for live server-managed collection.
- Collection scope must be controlled and documented.

---

## 11. VQL and Velociraptor Artifacts

### What they are

**VQL** means Velociraptor Query Language.

VQL is used to query endpoint information and create investigation workflows.

**Artifacts** are reusable definitions for collecting and investigating endpoint information.

### Why the project needs them

They provide repeatable methods for investigating endpoint activity and collecting relevant information.

### Where they are used

Within the Velociraptor platform, through the server and endpoint clients.

### What they investigate

Depending on the artifact:

- Processes.
- Network connections.
- Event logs.
- Persistence.
- Other forensic information.

### Data generated or collected

Artifact execution returns endpoint information and investigation results for analyst review.

### Communication

The analyst schedules or launches a collection through the Velociraptor Server.

The server communicates with the relevant client, which executes the authorized collection and returns results.

### Verification

- Select an appropriate basic artifact.
- Run it against an enrolled endpoint.
- Confirm the collection completes.
- Inspect the returned results.
- Check that the output supports the intended investigation question.
- Record the collection and its scope.

### Limitations

- Results depend on artifact design and execution context.
- An artifact does not automatically prove malicious activity.
- Collection scope and permissions affect available information.
- Findings must be correlated with other evidence.

---

## 12. Kali Linux and Attack Simulation Utilities

### What it is

Kali Linux is the adversary simulation machine in this lab.

### Why the project needs it

It generates controlled activity against the Windows and Ubuntu endpoints so that telemetry, detections, and investigation workflows can be tested.

### Installation location

Kali Linux virtual machine in the UTM lab environment.

### Planned utilities

- Netcat.
- Python.
- SSH testing utilities.
- Hydra, where appropriate for controlled authentication testing.
- Other tools required by individual scenarios.

### What it generates

Depending on the approved scenario:

- Controlled process execution.
- Simulated network connections.
- Authentication test activity.
- Other selected adversary behaviours.

### Communication

Kali communicates with Windows and Ubuntu through the configured private lab network.

### Verification

- Confirm the Kali VM is running.
- Verify its network configuration.
- Confirm only intended lab targets are reachable.
- Verify required utilities are available when a scenario requires them.
- Confirm activity is restricted to authorized lab endpoints.

### Limitations and safety constraints

- Use only isolated lab targets.
- Do not target public IP addresses or third-party systems.
- Use test accounts rather than real credentials.
- Limit authentication attempts.
- Use non-destructive scenarios.
- Keep listeners and simulated callback services private to the lab.
- Define cleanup and rollback before each scenario.

Kali is an adversary simulation node, not a monitored target endpoint.

---

## 13. Technology Interaction Summary

| Source or component | Produces or collects | Destination or consumer |
|---|---|---|
| Sysmon | Windows system telemetry | Windows event log and Universal Forwarder |
| Windows Security Auditing | Security events | Windows Security log and Universal Forwarder |
| PowerShell Logging | PowerShell events | Windows event log and Universal Forwarder |
| Linux auditd | Linux audit records | Audit log and Universal Forwarder |
| Splunk Universal Forwarder | Configured endpoint logs | Splunk   |
| Splunk   | Searches, detections, alerts, dashboards | Analyst |
| Velociraptor Client | Endpoint collection results | Velociraptor Server |
| Velociraptor Server | Managed collections and hunt results | Analyst |
| VQL and Artifacts | Endpoint investigation output | Velociraptor investigation |
| Kali utilities | Controlled test activity | Windows and Ubuntu endpoints |

## 14. Verification and Readiness Requirements

Before attack scenarios begin, the project must verify:

- [ ] Splunk   is operational.
- [ ] Windows Universal Forwarder communicates with Splunk.
- [ ] Ubuntu Universal Forwarder communicates with Splunk.
- [ ] Windows Sysmon generates expected configured events.
- [ ] Windows Security Auditing generates expected configured events.
- [ ] PowerShell Logging produces expected configured events.
- [ ] Ubuntu auditd generates records covered by the configured rules.
- [ ] Windows telemetry is searchable in Splunk.
- [ ] Ubuntu telemetry is searchable in Splunk.
- [ ] Velociraptor Server is operational.
- [ ] Windows Velociraptor Client is enrolled and communicating.
- [ ] Ubuntu Velociraptor Client is enrolled and communicating.
- [ ] Basic artifact collections work on both endpoints.
- [ ] Kali can reach only the intended lab targets for the required tests.
- [ ] Time and timestamps are sufficiently consistent for correlation.

## 15. Document Status

**Status:** Draft — pending review.

Technology versions, ARM64 compatibility, installation methods, configuration details, and exact commands will be verified during Phase 1 before deployment.

**Next deliverable:** `architecture/network-topology.md`.
