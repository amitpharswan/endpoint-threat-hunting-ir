# Network Topology

**Project:**   Endpoint Threat Hunting & Incident Response Platform  
**Document:** `architecture/network-topology.md`  
**Status:** Architecture draft — pending environment and network verification

## 1. Purpose

This document defines the planned network architecture for the four-machine security lab. It identifies the intended communication paths, proposed service ports, and network restrictions required for centralized monitoring, endpoint investigation, and controlled attack simulation.

The lab will use the existing UTM virtual network. All machine-to-machine communication must use the configured private lab network.

Actual IP addresses, interfaces, gateways, and port availability must be verified from the running environment. No IP address or existing network configuration is assumed.

## 2. Lab Machines and Network Roles

| Machine | Network role | Intended communication |
|---|---|---|
| MacBook Air M1 — macOS | Central management and monitoring host | Receives forwarded logs, hosts Splunk and Velociraptor Server, and manages endpoint investigations |
| Windows 11 ARM64 VM | Monitored endpoint | Sends telemetry to Splunk and communicates with Velociraptor Server; receives controlled attack simulations from Kali |
| Ubuntu Linux ARM64 VM | Monitored endpoint | Sends telemetry to Splunk and communicates with Velociraptor Server; receives controlled attack simulations from Kali |
| Kali Linux VM | Adversary simulation machine | Initiates controlled attack scenarios against Windows and Ubuntu |

## 3. Planned Network Topology

The following is a logical representation. It does not establish actual IP addresses, interface names, or verified routes.

```text
                    PRIVATE UTM LAB NETWORK
                              |
          +-------------------+-------------------+
          |                   |                   |
          |                   |                   |
   +------+-------+    +------+-------+    +------+-------+
   | MacBook Air  |    | Windows 11   |    | Ubuntu Linux |
   | macOS        |    | ARM64 VM     |    | ARM64 VM     |
   |              |    |              |    |              |
   | Splunk       |    | Splunk UF    |    | Splunk UF    |
   |     |    | Sysmon       |    | auditd       |
   |              |    | VR Client    |    | VR Client    |
   | VR Server    |    |              |    |              |
   +--------------+    +--------------+    +--------------+
          |                   ^                   ^
          |                   |                   |
          +-------------------+-------------------+
                              |
                       +------+-------+
                       | Kali Linux   |
                       | VM           |
                       |              |
                       | Attack       |
                       | simulation   |
                       +--------------+
```

**Diagram interpretation:**
- Windows and Ubuntu forward configured telemetry to Splunk   on the Mac.
- The Mac's Velociraptor Server communicates with the Velociraptor Clients on both endpoints.
- Kali initiates controlled attack simulations against Windows and Ubuntu.
- All communication is intended to remain within the configured lab network.

## 4. Planned Service Ports

The following ports are proposed allocations from the project blueprint. They are not confirmation that services are currently listening on these ports. Existing configurations must be inspected before making changes.

| Service | Proposed port | Direction / purpose |
|---|---:|---|
| Splunk Web | TCP 8000 | Browser access to Splunk |
| Splunk Management | TCP 8089 | Splunk management |
| Splunk Receiver | TCP 9997 | Forwarder-to-Splunk data |
| Velociraptor GUI | TCP 8889 | Browser access to Velociraptor |
| Velociraptor Frontend | TCP 8002 | Client-to-server communication |
| Velociraptor API | TCP 8001 | Internal API service, where configured |
| Simulated Windows C2 | TCP 4444 | Controlled lab connection |
| Simulated Linux callback | TCP 5555 | Controlled lab connection |
| Test HTTP service | TCP 8080 | Controlled lab HTTP activity |

### Port and exposure requirements

1. Preserve existing Splunk ports and configurations.
2. Configure the Velociraptor frontend on a separate port.
3. Keep the Velociraptor API restricted to the host unless external access is specifically required.
4. Do not expose the Velociraptor GUI to public networks.
5. Ensure Velociraptor client and server configurations agree on the frontend address and port.
6. Open C2 and test-service ports only when required by the relevant controlled attack scenario.
7. Verify actual listening services and connection behavior during network validation.

## 5. Communication Paths

### Path A — Windows log forwarding

**Flow:**

Windows endpoint → Splunk Universal Forwarder → Mac Splunk  

Purpose:
- Forward configured Windows telemetry.
- Deliver Security, System, PowerShell, and Sysmon events as configured.
- Make endpoint events available for Splunk searches and detection development.

### Path B — Ubuntu log forwarding

**Flow:**

Ubuntu endpoint → Splunk Universal Forwarder → Mac Splunk  

Purpose:
- Forward configured Linux telemetry.
- Deliver authentication, system, and audit-related events as configured.
- Make Linux endpoint activity available for Splunk searches and detection development.

### Path C — Windows endpoint investigation

**Flow:**

Mac Velociraptor Server ↔ Windows Velociraptor Client

Purpose:
- Maintain endpoint investigation communication.
- Support authorized artifact collection and endpoint hunting.
- Retrieve investigation results for analysis.

### Path D — Ubuntu endpoint investigation

**Flow:**

Mac Velociraptor Server ↔ Ubuntu Velociraptor Client

Purpose:
- Maintain endpoint investigation communication.
- Support authorized artifact collection and endpoint hunting.
- Retrieve investigation results for analysis.

### Path E — Windows attack simulation

**Flow:**

Kali Linux → Windows endpoint

Purpose:
- Execute approved Windows attack scenarios.
- Generate controlled endpoint activity for telemetry and detection validation.
- Verify the effects of each scenario.

### Path F — Linux attack simulation

**Flow:**

Kali Linux → Ubuntu endpoint

Purpose:
- Execute approved Linux attack scenarios.
- Generate controlled endpoint activity for telemetry and detection validation.
- Verify the effects of each scenario.

Communication direction and connection behavior for all paths must be tested during Phase 2.

## 6. Network Verification Requirements

Before relying on this topology, verify and document the following for all four machines:

- Actual IP address.
- Network interface name.
- Subnet and network configuration.
- Default gateway, where applicable.
- Connectivity through the configured private UTM network.
- Required service ports and listening services.
- Forwarder-to-Splunk receiver connectivity.
- Velociraptor client-to-server connectivity.
- Kali-to-Windows connectivity.
- Kali-to-Ubuntu connectivity.

Do not infer successful communication solely from an IP address or an assumed port allocation. Validate the actual connection behavior.

## 7. Security and Isolation Requirements

- Keep the lab on its configured private virtual network.
- Do not expose management services or simulated C2 listeners to public networks.
- Restrict the Velociraptor API to the host unless a documented requirement justifies otherwise.
- Keep the Velociraptor GUI private.
- Enable scenario-specific C2 and test ports only when needed.
- Use controlled attack activity against the designated lab endpoints only.
- Verify network restrictions before executing attack scenarios.

## 8. Phase 2 Validation

The following checks are required during Phase 2:

| Validation | Expected result |
|---|---|
| Mac to Windows | Required lab communication works |
| Mac to Ubuntu | Required lab communication works |
| Kali to Windows | Controlled scenario communication works |
| Kali to Ubuntu | Controlled scenario communication works |
| Windows to Splunk receiver | Forwarding connection works |
| Ubuntu to Splunk receiver | Forwarding connection works |
| Windows to Velociraptor frontend | Client communication works |
| Ubuntu to Velociraptor frontend | Client communication works |

Record the observed result, relevant address and port, test method, and any failure or corrective action. Do not mark a path as validated until it has been tested.

## 9. Document Status

**Current status:** Draft prepared from the project blueprint.

**Pending verification:**
- Actual machine IP addresses.
- Network interfaces and default gateways.
- Existing UTM network configuration.
- Actual Splunk and Velociraptor service ports.
- Required communication paths and their connection behavior.

**Completion condition:** The network topology is considered verified only after the actual environment has been inspected and the required communication paths have been tested and documented during the appropriate project phase.
