# Security Event Records — Sprint 3 Set (SCRUM-168: SW-DIST-01)

- Related Jira Issues: SCRUM-168 (EVT-06, EVT-07, EVT-08, EVT-09, EVT-10)
- Contributor: Hasan Dagdelen / Distribution Switch Lead
- Date: 10/27/2026
- Target Environment: Packet Tracer Centralized Logging (10.20.50.10)

---

## Event 1: EVT-06

- **Event ID:** EVT-06
- **Event Name:** Distribution Switch Interface State Change
- **Jira Issue:** SCRUM-168
- **Contributor:** Hasan Dagdelen
- **Date and Time:** Oct 06 05:31:01.3131 / Oct 06 05:31:04.3131
- **Source Device:** SW-DIST-01 (10.20.50.1)
- **Packet Tracer file/version:** CYBEROPS07_Configuration_v1_1 / Cisco Packet Tracer 8.2.1
- **Evidence status:** Observed

### Generating Action
Accessed SW-DIST-01 CLI, navigated to interface FastEthernet0/1, executed `shutdown`, and subsequently executed `no shutdown`.

### Log Appearance
```text
*Oct 06, 05:31:01.3131: %LINK-5-CHANGED: Interface FastEthernet0/1, changed state to administratively down
*Oct 06, 05:31:01.3131: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
*Oct 06, 05:31:04.3131: %LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up
*Oct 06, 05:31:04.3131: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
```

## Explanation
Legitimate: Scheduled interface maintenance, re-cabling, or administrative reset of a distribution port.

Malicious: An unauthorized user or attacker taking down a core trunk or access line to cause Denial of Service (DoS) across attached subnets.

## Distinguishing Evidence
Correlation with approved network maintenance windows and concurrent `%SYS-5-CONFIG_I` terminal session logs.

## Classification
Syslog severity, if available: Notice (Severity 5)

Assigned analytical severity and rationale: Low to Medium (Routine if part of maintenance; Medium if executed out-of-hours on a core distribution port).

Analytical question: Was this interface state change pre-approved by an active network change ticket?

Response category: Correlate

Security Property affected: Availability

## Limitations
The link-state event logs physical line protocol state transitions but does not detail the specific CLI user account or session line that initiated the command without external AAA/TACACS+ accounting.

---

## Event 2: EVT-07

- **Event ID:** EVT-07
- **Event Name:** Distribution Switch Configuration Mode Entry & Unauthorized VLAN Creation
- **Jira Issue:** SCRUM-168
- **Contributor:** Hasan Dagdelen
- **Date and Time:** Oct 06 05:21:50.2121
- **Source Device:** SW-DIST-01 (10.20.50.1)
- **Packet Tracer file/version:** CYBEROPS07_Configuration_v1_1 / Cisco Packet Tracer 8.2.1
- **Evidence status:** Observed

### Generating Action
Accessed SW-DIST-01 CLI, entered global configuration mode (`configure terminal`), created an unapproved VLAN (`vlan 999 name SUSPICIOUS_MGMT_TEMP`), and exited using `end`.

### Log Appearance
```text
*Oct 06, 05:21:50.2121: %SYS-5-CONFIG_I: Configured from console by console
```

## Explanation
Legitimate: Network engineer configuring new network segments or modifying Layer 2/3 parameters during scheduled maintenance.

Malicious: An attacker attempting to establish persistent out-of-band management channels, bypass VLAN isolation, or perform VLAN hopping.

## Distinguishing Evidence
Running-configuration diff comparison showing the addition of `VLAN 999`, paired with change-control ticket verification.

## Classification
Syslog severity, if available: Notice (Severity 5)

Assigned analytical severity and rationale: Medium. Unapproved VLAN creation on a distribution switch alters Layer 2 network segmentation boundaries.

Analytical question: Do the newly added configuration parameters (VLAN 999) match an approved architecture ticket?

Response category: Correlate

Security Property affected: Integrity

## Limitations
Packet Tracer logs the entry and exit from global configuration mode (`%SYS-5-CONFIG_I`), but does not stream individual CLI command strings (`vlan 999`) to Syslog.

---

## Event 3: EVT-08

- **Event ID:** EVT-08
- **Event Name:** Privilege Exec Mode Authentication Failure (`enable` secret)
- **Jira Issue:** SCRUM-168
- **Contributor:** Hasan Dagdelen
- **Date and Time:** Oct 06 05:25:00.000 (Approx)
- **Source Device:** SW-DIST-01 (10.20.50.1)
- **Packet Tracer file/version:** CYBEROPS07_Configuration_v1_1 / Cisco Packet Tracer 8.2.1
- **Evidence status:** Observed (Console) / PT Forwarding Limitation Documented

### Generating Action
Entered user EXEC mode on SW-DIST-01, initiated the `enable` command, and deliberately entered an incorrect secret password three consecutive times.

### Log Appearance
```text
% Bad secrets
```

## Explanation
Legitimate: Administrator password mistype or outdated credentials entered during routine management.

Malicious: Brute-force or dictionary attack attempting to escalate privileges from user EXEC to privileged EXEC mode on the core distribution switch.

## Distinguishing Evidence
Frequency and rate of failure occurrences, line connection source (console vs. VTY line), and active SSH session history.

## Classification
Syslog severity, if available: Level 4 (Warning)

Assigned analytical severity and rationale: Medium to High. Indicates unauthorized privilege escalation attempts on control-plane infrastructure.

Analytical question: Did this authentication failure originate from an authorized console/management IP, and was it followed by account lockout?

Response category: Immediate action

Security Property affected: Confidentiality / Integrity

## Limitations
Packet Tracer IOS images print `% Bad secrets` locally to the console terminal but fail to forward `%SYS-4-BADPASS` or enable-failure syslog traps over UDP 514 to remote Syslog servers.

---

## Event 4: EVT-09

- **Event ID:** EVT-09
- **Event Name:** Port Security Violation and Error-Disable Action
- **Jira Issue:** SCRUM-168
- **Contributor:** Hasan Dagdelen
- **Date and Time:** Oct 06 05:55:53.5555
- **Source Device:** SW-DIST-01 (10.20.50.1)
- **Packet Tracer file/version:** CYBEROPS07_Configuration_v1_1 / Cisco Packet Tracer 8.2.1
- **Evidence status:** Observed

### Generating Action
Configured port security on SW-DIST-01 interface FastEthernet0/10 with static MAC `1111.2222.3333` and violation mode `shutdown`. Connected host PC0 (MAC `0030.A314.C145`) and initiated traffic (`ping 10.20.50.1`).

### Log Appearance
```text
*Oct 06, 05:55:53.5555: %LINK-5-CHANGED: Interface FastEthernet0/10, changed state to administratively down
*Oct 06, 05:55:53.5555: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/10, changed state to down
*Oct 06, 05:55:53.5555: %PM-4-ERR_DISABLE: psecure-violation error detected on Fa0/10, putting Fa0/10 in err-disable state.
*Oct 06, 05:55:53.5555: %PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 0030.A314.C145 on port FastEthernet0/10.
```

## Explanation
Legitimate: User swapping out an authorized workstation or attaching an unapproved personal device (BYOD) to a secured port.

Malicious: Rogue device insertion or MAC spoofing attempt targeting the distribution layer to intercept subnet traffic.

## Distinguishing Evidence
Output of `show port-security interface FastEthernet0/10` confirming violation count `1` and last source MAC `0030.A314.C145`.

## Classification
Syslog severity, if available: Critical (Severity 2) / Warning (Severity 4)

Assigned analytical severity and rationale: High. Represents an unauthorized physical connection attempt, resulting in automatic port suppression.

Analytical question: Is MAC address `0030.A314.C145` listed in the corporate asset management inventory?

Response category: Immediate action

Security Property affected: Integrity / Availability

## Limitations
Identifies the violating MAC address and switch port, but does not identify host operating system details or physical user identity.

---

## Event 5: EVT-10

- **Event ID:** EVT-10
- **Event Name:** Spanning Tree BPDU Guard Violation and Port Error-Disable
- **Jira Issue:** SCRUM-168
- **Contributor:** Hasan Dagdelen
- **Date and Time:** Oct 06 06:04:19.044
- **Source Device:** SW-DIST-01 (10.20.50.1)
- **Packet Tracer file/version:** CYBEROPS07_Configuration_v1_1 / Cisco Packet Tracer 8.2.1
- **Evidence status:** Observed

### Generating Action
Enabled `spanning-tree portfast` and `spanning-tree bpduguard enable` on interface FastEthernet0/11 on SW-DIST-01, then connected a device transmitting Spanning Tree Bridge Protocol Data Units (BPDUs).

### Log Appearance
```text
*Oct 06, 06:04:19.044: %SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port FastEthernet0/11 with BPDU Guard enabled. Disabling port.
*Oct 06, 06:04:19.044: %PM-4-ERR_DISABLE: bpduguard error detected on 0/11, putting 0/11 in err-disable state.
```

## Explanation
Legitimate: Accidental connection of an unauthorized switch, hub, or wireless router by an employee attempting to expand port availability.

Malicious: Rogue switch insertion attempt to manipulate Spanning Tree topology, perform STP root bridge hijacking, or conduct Layer 2 Man-in-the-Middle (MitM) attacks.

## Distinguishing Evidence
`show interfaces FastEthernet0/11 status` output showing `err-disabled` state with `bpduguard` cause code.

## Classification
Syslog severity, if available: Critical (Severity 2) / Warning (Severity 4)

Assigned analytical severity and rationale: High. Unauthorized STP BPDUs threaten Layer 2 topology stability and routing path integrity.

Analytical question: Was an unauthorized switch or bridge device physically attached to access interface FastEthernet0/11?

Response category: Immediate action

Security Property affected: Availability / Integrity

## Limitations
BPDU Guard immediately disables the port upon frame reception, preventing further topology calculation attacks but omitting higher-layer packet payloads.