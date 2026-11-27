# Endpoint Threat Hunting & Incident Response Platform

## 1. Project Overview

The Endpoint Threat Hunting & Incident Response Platform is a hands-on cybersecurity lab designed to demonstrate a multi-endpoint security monitoring, threat hunting, investigation, and incident response workflow.

The lab combines:

- **Splunk  ** for centralized security monitoring, log analysis, detection searches, alerts, and dashboards.
- **Velociraptor** for endpoint visibility, live investigation, artifact collection, and hunting.
- **Sysmon and Windows Security Auditing** for Windows endpoint telemetry.
- **Linux auditd** for Linux security auditing.
- **Kali Linux** for controlled adversary simulation.

The environment will use a MacBook Air with Apple Silicon M1, Windows 11 ARM64, Ubuntu Linux ARM64, and Kali Linux.

## 2. Project Objectives

The project aims to:

1. Monitor Windows and Ubuntu endpoints from a central security platform.
2. Collect and centralize endpoint security telemetry.
3. Develop and validate SPL-based detection searches.
4. Simulate controlled adversary activity from Kali Linux.
5. Investigate suspicious endpoint activity using Velociraptor.
6. Correlate Splunk findings with Velociraptor investigation results.
7. Collect and preserve forensic evidence.
8. Perform appropriate containment, cleanup, and recovery.
9. Produce an individual incident report for each completed attack scenario.
10. Document the architecture, deployment, detections, investigations, and results.

This is a practical two-endpoint lab. It is not a production EDR deployment or a replacement for commercial EDR platforms.

## 3. Overall Architecture

The platform consists of four machines.

| Machine   | Operating system       | Primary role              |
| --------- | ---------------------- | ------------------------- |
| Machine 1 | MacBook Air M1 / macOS | Central security server   |
| Machine 2 | Windows 11 ARM64       | Target endpoint 1         |
| Machine 3 | Ubuntu Linux ARM64     | Target endpoint 2         |
| Machine 4 | Kali Linux             | Adversary simulation node |

### 3.1 Machine 1 — MacBook Air: Central Security Server

**Operating system:** macOS
**Architecture:** Apple Silicon M1 / ARM64

Planned components:

- Splunk  
- Velociraptor Server
- Web browser
- Project evidence directory

Responsibilities:

- Receive forwarded security logs from Windows and Ubuntu.
- Search, correlate, and analyse endpoint events.
- Develop and validate detection searches.
- Generate detection alerts and dashboards.
- Manage Velociraptor clients.
- Launch endpoint artifact collections and hunts.
- Preserve investigation evidence.
- Maintain project documentation.

The Mac is the central monitoring and investigation system.

### 3.2 Machine 2 — Windows 11 ARM64: Target Endpoint 1

Planned components:

**Splunk Universal Forwarder**

- Collect configured Windows event logs.
- Forward Security, System, PowerShell, and Sysmon events to Splunk  .

**Sysmon**

- Record detailed Windows system activity.
- Provide process execution telemetry.
- Record configured network connections.
- Record selected registry and file activity.
- Support process lineage and behavioural investigations.

Important planned Sysmon event examples:

| Event ID | Activity            |
| -------- | ------------------- |
| 1        | Process creation    |
| 3        | Network connection  |
| 5        | Process termination |
| 10       | Process access      |
| 11       | File creation       |
| 12–14    | Registry activity   |
| 22       | DNS query           |

Actual event availability will depend on the Sysmon configuration.

**Windows Security Auditing**

- Record security-related operating-system activity.
- Support process creation, authentication, and account activity investigations.

Example event IDs include 4688, 4624, 4625, 4698, and 4720. Availability depends on enabled audit policies and operating-system configuration.

**PowerShell Logging**

- Provide visibility into configured PowerShell script block and module logging events.

**Velociraptor Client**

- Communicate with the central Velociraptor Server.
- Execute authorized endpoint collections.
- Collect process, network, and persistence information.
- Support endpoint investigation and hunting.

### 3.3 Machine 3 — Ubuntu Linux ARM64: Target Endpoint 2

Planned components:

**Splunk Universal Forwarder**

- Collect configured Linux authentication logs.
- Forward syslog and audit logs to Splunk  .

Initial log sources to evaluate:

- `/var/log/auth.log`
- `/var/log/syslog`
- auditd logs

Actual log paths will be verified against the installed Ubuntu version.

**Linux auditd**

- Record configured security-relevant system activity.
- Monitor selected process execution.
- Monitor selected privileged file modifications.
- Record relevant configuration and persistence changes.

Initial monitoring areas:

- Process execution through execve-related audit rules.
- Cron configuration changes.
- SSH configuration and authorized-key changes.
- Selected privileged file modifications.

**Velociraptor Client**

- Communicate with the central Velociraptor Server.
- Collect Linux process and network information.
- Investigate cron jobs and SSH configuration.
- Perform authorized forensic collections.

### 3.4 Machine 4 — Kali Linux: Adversary Simulation Node

Planned tools include:

- Netcat
- Python
- SSH testing utilities
- Hydra, where appropriate for controlled authentication testing
- Other scenario-specific testing utilities

Responsibilities:

- Generate controlled attack activity.
- Simulate attacker network connections.
- Perform authorized authentication testing.
- Initiate selected attack scenarios against Windows and Ubuntu.
- Provide an attacker-side perspective for investigations.

**Project design decision:** Splunk Universal Forwarder and Velociraptor Client will not be installed on Kali. Kali remains the adversary simulation machine, not a monitored target endpoint.

## 4. Central Security Platform

### 4.1 Splunk   — SIEM

Splunk   is the central SIEM platform.

Its responsibilities are to:

1. Receive endpoint logs.
2. Index collected events.
3. Make events searchable.
4. Correlate activity from different endpoints.
5. Run SPL detection searches.
6. Generate alerts.
7. Display security dashboards.
8. Support incident investigation.

Splunk does not automatically determine whether an event is malicious. Detection logic must be developed and validated.

### 4.2 Splunk Universal Forwarder

The Universal Forwarder is the endpoint log collection and forwarding component.

It collects configured data sources from Windows and Ubuntu and sends the data to Splunk  .

It does not independently perform the complete SIEM investigation process.

### 4.3 Velociraptor — Endpoint Investigation

Velociraptor provides endpoint visibility, collection, and investigation capabilities.

**Velociraptor Server**

- Manages connected clients.
- Provides the administrative console.
- Schedules artifact collections.
- Manages hunts.
- Stores investigation results.

**Velociraptor Client**

- Runs on the endpoint.
- Communicates with the server.
- Executes authorized VQL-based artifact collections.
- Returns collected endpoint information.

**VQL — Velociraptor Query Language**

VQL is used to query endpoint information and create investigation workflows.

**Artifacts**

Artifacts are reusable definitions for collecting and investigating endpoint information, including processes, network connections, event logs, persistence, and other forensic data.

## 5. How the Platforms Work Together

Splunk and Velociraptor have different but complementary responsibilities.

- Splunk identifies suspicious activity using collected telemetry and validated detection logic.
- Velociraptor performs deeper endpoint investigation and collects additional evidence.

Example: suspicious PowerShell execution on Windows.

1. PowerShell executes on the Windows endpoint.
2. Sysmon records the process creation event, when covered by the configuration.
3. Splunk Universal Forwarder sends the event to Splunk.
4. Splunk detection searches examine suspicious command-line activity.
5. The analyst reviews the result.
6. The analyst uses Velociraptor to collect additional endpoint information.
7. Process relationships, network activity, and persistence are investigated.
8. Relevant evidence is preserved.
9. The analyst performs the appropriate response.
10. The incident is documented.

## 6. Network Architecture

The lab will use the existing UTM virtual network.

The four machines must communicate through the configured private lab network.

The final network documentation will identify:

- Mac host IP.
- Windows VM IP.
- Ubuntu VM IP.
- Kali VM IP.
- Default gateways and network interfaces.
- Allowed communication paths.
- Required TCP ports.

**No IP addresses are assumed in this architecture document.** They must be verified from the actual environment.

### 6.1 Planned Communication Paths

| Source                    | Destination             | Purpose                      |
| ------------------------- | ----------------------- | ---------------------------- |
| Windows                   | Mac / Splunk   | Forward endpoint logs        |
| Ubuntu                    | Mac / Splunk   | Forward endpoint logs        |
| Mac / Velociraptor Server | Windows                 | Endpoint investigation       |
| Mac / Velociraptor Server | Ubuntu                  | Endpoint investigation       |
| Kali                      | Windows                 | Controlled attack simulation |
| Kali                      | Ubuntu                  | Controlled attack simulation |

### 6.2 Proposed Service Ports

These are proposed allocations and must be checked against existing configurations before use.

| Service                  | Proposed port | Purpose                        |
| ------------------------ | ------------: | ------------------------------ |
| Splunk Web               |      TCP 8000 | Browser access                 |
| Splunk Management        |      TCP 8089 | Splunk management              |
| Splunk Receiver          |      TCP 9997 | Forwarder-to-Splunk data       |
| Velociraptor GUI         |      TCP 8889 | Browser access                 |
| Velociraptor Frontend    |      TCP 8002 | Client-to-server communication |
| Velociraptor API         |      TCP 8001 | Internal API, where configured |
| Simulated Windows C2     |      TCP 4444 | Controlled lab connection      |
| Simulated Linux callback |      TCP 5555 | Controlled lab connection      |
| Test HTTP service        |      TCP 8080 | Controlled lab HTTP activity   |

Security constraints:

- Preserve existing Splunk ports.
- Keep the Velociraptor frontend separate from existing Splunk services.
- Restrict the Velociraptor API to the host unless external access is specifically required.
- Do not expose the Velociraptor GUI to public networks.
- Ensure client and server configurations agree on the frontend address and port.
- Open C2 and test service ports only for the relevant controlled scenario.

## 7. End-to-End Security Operations Workflow

The complete project workflow is:

1. Prepare and verify the lab environment.
2. Generate controlled activity from Kali.
3. Verify the activity on the target endpoint.
4. Collect endpoint telemetry through the configured monitoring components.
5. Forward telemetry to Splunk.
6. Search and correlate events using SPL.
7. Develop and validate detection logic.
8. Investigate the endpoint using Velociraptor.
9. Correlate SIEM and endpoint investigation findings.
10. Preserve relevant forensic evidence.
11. Perform containment and response.
12. Verify cleanup and recovery.
13. Prepare an individual incident report.
14. Review the scenario before proceeding to the next attack.

Each attack scenario must be completed and documented before the next scenario begins.

## 8. Security and Operational Constraints

The lab must follow these constraints:

- Use an isolated lab environment.
- Do not target public IP addresses or third-party systems.
- Use test accounts rather than real credentials.
- Keep authentication testing controlled and limited.
- Use non-destructive attack scenarios.
- Do not expose management services or simulated callback listeners publicly.
- Define cleanup and rollback before executing each scenario.
- Preserve evidence before removing simulated attack artifacts.
- Verify recovery before beginning another scenario.
- Do not claim a detection, investigation, or configuration is successful without testing.

## 9. Scope and Limitations

This project is a practical two-endpoint security monitoring and incident response lab.

It demonstrates selected   security operations workflows, including telemetry collection, detection engineering, endpoint investigation, forensic evidence preservation, and incident response.

It is not intended to be a production EDR deployment or a replacement for commercial EDR platforms.

The availability of telemetry and investigation results depends on actual endpoint configurations, enabled audit policies, Sysmon configuration, configured log sources, and successful client-server communication.

## 10. Architecture Completion Criteria

The architecture documentation will be considered complete when:

- [x] All four machine roles are documented.
- [x] Splunk   and Universal Forwarder responsibilities are clear.
- [x] Windows telemetry components are documented.
- [x] Ubuntu audit and monitoring components are documented.
- [x] Velociraptor Server and Client responsibilities are clear.
- [x] Kali's adversary simulation role is documented.
- [x] Communication paths are documented.
- [x] Proposed ports and security constraints are documented.
- [x] Splunk detection and Velociraptor investigation responsibilities are distinguished.
- [x] The architecture has been reviewed for technical consistency.

**Document status:** Architecture draft — pending review and environment verification.

**Next Phase 0.4 deliverable:** `technology-stack.md`.
