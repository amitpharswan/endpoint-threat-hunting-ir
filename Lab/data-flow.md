# Data Flow

**Project:** Enterprise Endpoint Threat Hunting & Incident Response Platform  
**Document:** `architecture/data-flow.md`  
**Status:** Architecture draft — pending review and environment validation

## 1. Purpose

This document describes how endpoint activity becomes security telemetry, how that telemetry is collected and analyzed, and how suspicious activity progresses through endpoint investigation, evidence preservation, response, and incident reporting.

The project uses two complementary platforms:

- **Splunk Enterprise:** Centralized log collection, indexing, searching, correlation, detection, alerting, and dashboards.
- **Velociraptor:** Endpoint visibility, live investigation, artifact collection, hunting, and forensic evidence gathering.

The intended workflow covers both Windows 11 ARM64 and Ubuntu Linux ARM64 endpoints.

## 2. High-Level Data Flow

```text
                  ENDPOINT ACTIVITY
                         |
             +-----------+-----------+
             |                       |
       WINDOWS ENDPOINT         UBUNTU ENDPOINT
             |                       |
           Sysmon              Linux auditd
             |                       |
       Windows Event Logs      Linux System Logs
       Security Auditing       Authentication Logs
       PowerShell Logging      Audit Events
             |                       |
             v                       v
       Splunk Universal        Splunk Universal
          Forwarder               Forwarder
             |                       |
             +-----------+-----------+
                         |
                         v
                 MAC — SPLUNK
                 ENTERPRISE
                         |
                 Receive and Index
                         |
                   SPL Searches
                         |
                 Detection / Alert
                         |
                         v
                 ANALYST REVIEW
                         |
                         v
               VELOCIRAPTOR SERVER
                         |
                Endpoint Investigation
                         |
              +----------+----------+
              |                     |
        WINDOWS CLIENT         UBUNTU CLIENT
              |                     |
              +----------+----------+
                         |
                 Investigation Results
                         |
                 Correlation and Timeline
                         |
                 Evidence Preservation
                         |
                  Response / Recovery
                         |
                  Incident Report
```

This diagram represents the intended project workflow. Actual event sources, configured collection, and communication must be verified during deployment and telemetry validation.

## 3. Windows Telemetry Flow

### 3.1 Endpoint activity

A user or process performs an action on the Windows endpoint.

Examples include:
- Starting a process.
- Executing a PowerShell command.
- Creating a scheduled task.
- Modifying a registry Run Key.
- Establishing a network connection.

### 3.2 Telemetry generation

Windows monitoring components record configured activity.

**Sysmon**

Sysmon can record process creation and other configured system activity. Depending on the configuration, relevant fields may include:
- Process name.
- Command line.
- Parent process.
- Process identifiers.
- Process GUID.
- Process-associated network activity, when network connection monitoring is enabled.

**Windows Security Auditing**

Configured audit policies can generate Security events for relevant activities, such as process creation, authentication, account changes, or scheduled-task activity, depending on the enabled policies.

**PowerShell Logging**

Configured PowerShell logging provides additional visibility into PowerShell activity.

These sources provide complementary telemetry. Their actual coverage depends on the installed components and their configurations.

### 3.3 Log collection and forwarding

The Splunk Universal Forwarder runs on Windows.

Its role is to:
1. Collect configured Windows event sources.
2. Forward collected events to Splunk Enterprise.
3. Make the configured endpoint telemetry available for centralized analysis.

The forwarder is a collection and forwarding component. It does not independently perform the complete SIEM investigation process.

### 3.4 Splunk processing and detection

Splunk Enterprise on the Mac:
1. Receives forwarded events.
2. Indexes the collected data.
3. Makes the events searchable.
4. Supports searches using Splunk Search Processing Language (SPL).
5. Correlates relevant activity.
6. Runs detection searches.
7. Generates configured alerts and displays dashboard information.

Splunk does not automatically determine whether an event is malicious. Detection logic must be developed and validated against observed activity.

**Windows flow summary:**

Windows process or user activity → Sysmon / Windows Security Auditing / PowerShell Logging → Splunk Universal Forwarder → Splunk Enterprise → Index → SPL search → Detection or alert.

## 4. Ubuntu Linux Telemetry Flow

### 4.1 Endpoint activity

A user or process performs an action on the Ubuntu endpoint.

Examples include:
- Executing a command.
- Attempting SSH authentication.
- Modifying a scheduled task configuration.
- Changing SSH authorized keys.
- Performing activity covered by configured audit rules.

### 4.2 Telemetry generation

Linux monitoring components record configured activity.

**auditd**

Linux auditd provides security auditing. A properly configured audit rule can record relevant activity, such as modification of a scheduled-task configuration.

Other configured sources may include:
- Authentication logs.
- System logs.
- Audit logs.

The actual log locations and collection paths must be verified against the installed Ubuntu version and configuration.

### 4.3 Log collection and forwarding

The Splunk Universal Forwarder runs on Ubuntu.

Its role is to:
1. Collect configured Linux log sources.
2. Forward the collected data to Splunk Enterprise.
3. Make the endpoint telemetry available for centralized searching.

### 4.4 Splunk processing and detection

Splunk Enterprise:
1. Receives Linux events.
2. Indexes the collected data.
3. Makes events searchable.
4. Supports investigation through SPL.
5. Correlates relevant activity.
6. Runs detection searches.
7. Generates configured alerts and dashboard information.

Detection searches must be validated using observed endpoint activity.

**Ubuntu flow summary:**

Linux activity → auditd / configured authentication and system logs → Splunk Universal Forwarder → Splunk Enterprise → Index → SPL search → Detection or alert.

## 5. Velociraptor Investigation Flow

Splunk provides centralized visibility into collected telemetry. When suspicious activity is identified, Velociraptor provides additional endpoint investigation and collection capabilities.

### 5.1 Velociraptor Server

The Velociraptor Server runs on the Mac.

Its responsibilities include:
- Managing connected clients.
- Providing the administrative console.
- Scheduling artifact collections.
- Managing hunts.
- Storing investigation results.

### 5.2 Velociraptor Client

The Velociraptor Client runs on each monitored endpoint:
- Windows 11 ARM64.
- Ubuntu Linux ARM64.

The client communicates with the server, executes authorized VQL-based artifact collections, and returns collected endpoint information.

### 5.3 VQL and artifacts

VQL means Velociraptor Query Language. It is used to query endpoint information and create investigation workflows.

Artifacts are reusable definitions for collecting and investigating endpoint information. They can support collection of:
- Processes.
- Network connections.
- Event logs.
- Persistence information.
- Other forensic data.

### 5.4 Investigation sequence

When Splunk identifies suspicious activity:

1. The analyst reviews the relevant Splunk event or alert.
2. The analyst identifies the endpoint and activity requiring investigation.
3. The analyst opens Velociraptor.
4. The analyst selects appropriate endpoint artifacts or investigation workflows.
5. Velociraptor collects additional endpoint information.
6. The analyst reviews process relationships, network activity, persistence, and other relevant findings.
7. Relevant findings are correlated with the original Splunk telemetry.
8. Evidence is preserved before cleanup or other response actions.

**Investigation flow summary:**

Splunk detection → Analyst review → Velociraptor collection → Endpoint findings → Correlation → Evidence preservation → Incident response.

## 6. How Splunk and Velociraptor Work Together

Example: suspicious PowerShell execution on Windows.

| Step | Activity | Component |
|---|---|---|
| 1 | PowerShell executes on Windows | Windows endpoint |
| 2 | Process creation event is recorded | Sysmon |
| 3 | Event is forwarded | Splunk Universal Forwarder |
| 4 | Event is indexed and searchable | Splunk Enterprise |
| 5 | Detection search identifies suspicious command-line activity | SPL |
| 6 | Analyst reviews the alert | Analyst / Splunk |
| 7 | Additional endpoint information is collected | Velociraptor |
| 8 | Process relationships, network activity, and persistence are investigated | Velociraptor |
| 9 | Relevant evidence is preserved | Investigation workflow |
| 10 | Appropriate response is performed | Incident response workflow |

### Platform distinction

- **Splunk** helps identify suspicious activity from collected telemetry.
- **Velociraptor** helps investigate endpoints and collect additional evidence.
- **Together**, they support a broader detection and incident investigation workflow.

Neither platform replaces the need to validate findings and understand the underlying endpoint activity.

## 7. Correlation and Investigation

After collecting findings from Splunk and Velociraptor, the analyst must correlate the information to build an accurate incident picture.

Relevant correlation details include:
- Endpoint hostname.
- Event timestamps.
- User account.
- Process name.
- Process ID and other available process identifiers.
- Command line.
- Parent process.
- Network addresses and connections.
- Relevant event identifiers.
- Scheduled-task, registry, cron, or SSH persistence information where applicable.

The analyst should:
1. Establish the activity timeline.
2. Compare Splunk events with Velociraptor findings.
3. Identify relationships between processes and user activity.
4. Examine relevant network activity.
5. Identify persistence mechanisms where applicable.
6. Separate directly observed facts from assumptions or interpretations.

The resulting timeline and findings should support the individual incident report.

## 8. Evidence Preservation

Relevant forensic evidence must be preserved before cleanup or destructive response actions.

Depending on the scenario and endpoint, evidence may include:

**Windows**
- Windows event logs.
- Sysmon events.
- Prefetch.
- Registry hives.
- Scheduled tasks.
- Process and network information.

**Ubuntu Linux**
- Authentication logs.
- Audit logs.
- Cron configuration.
- SSH authorized keys.
- Relevant system and process information.

For each collected item, document:
- Collection time.
- Source endpoint.
- Collection method.
- Filename.
- SHA256 hash.
- Evidence storage location.

Maintain an evidence inventory and chain-of-custody record.

Do not remove simulated artifacts or modify relevant evidence before required collection and preservation are complete.

## 9. Response and Recovery Flow

After investigation and evidence preservation, perform the appropriate response actions.

The planned response workflow includes:
1. Terminate the simulated process where applicable.
2. Disable test persistence mechanisms.
3. Isolate the endpoint using a validated method where required.
4. Remove test artifacts.
5. Preserve relevant evidence.
6. Restore network connectivity where applicable.
7. Verify the endpoint has returned to its expected state.

Recovery verification must confirm:
- The simulated process has stopped.
- The simulated connection has ended.
- Test persistence has been removed.
- Relevant changes have been reversed.
- Security services are functioning.
- Splunk forwarding is working.
- Velociraptor communication is working.
- Normal endpoint behavior has resumed.

The analyst then documents the incident, investigation, evidence, response, and recovery results in an individual Markdown report.

## 10. Complete Attack-to-Incident Workflow

Each approved attack scenario follows the same end-to-end process:

1. Understand the attack and its expected behavior.
2. Prepare the attacker and target environments.
3. Record the baseline.
4. Execute the controlled scenario from Kali.
5. Verify the endpoint effects.
6. Review the resulting telemetry in Splunk.
7. Develop and validate the SPL detection search.
8. Investigate the endpoint using Velociraptor.
9. Correlate findings and build a timeline.
10. Preserve forensic evidence.
11. Perform containment and response.
12. Verify recovery and cleanup.
13. Write the individual incident report.
14. Review the completed scenario before proceeding.

A scenario is not considered complete merely because the attack command executed. Detection, investigation, evidence preservation, response, recovery, and reporting must also be completed or any failure explicitly documented.

## 11. Validation Requirements

The planned data flow must be validated during Phase 2.

| Validation area | Expected result |
|---|---|
| Windows telemetry generation | Configured Windows sources produce expected events |
| Windows forwarding | Configured events reach Splunk Enterprise |
| Ubuntu telemetry generation | Configured Linux sources produce expected events |
| Ubuntu forwarding | Configured events reach Splunk Enterprise |
| Splunk indexing | Expected endpoint events are searchable |
| SPL detection | Searches identify the intended controlled activity |
| Velociraptor Windows client | Client communicates with the server |
| Velociraptor Ubuntu client | Client communicates with the server |
| Artifact collection | Basic authorized collections return endpoint information |
| Correlation | Findings can be compared using relevant endpoint and activity details |
| Evidence preservation | Evidence is recorded with required inventory and hash information |

Record actual results during validation. Do not mark a data-flow stage as operational until it has been tested.

## 12. Document Status

**Current status:** Draft prepared from the project blueprint.

**Pending validation:**
- Actual Windows event-source configuration.
- Actual Ubuntu log-source and audit-rule configuration.
- Splunk forwarding and indexing.
- SPL detection behavior.
- Velociraptor client communication.
- Artifact collection results.
- End-to-end correlation and evidence workflow.

**Completion condition:** The data-flow architecture is considered technically validated when the expected telemetry, investigation, and evidence workflows have been tested and documented in the relevant project phases.