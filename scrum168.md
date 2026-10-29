Security Event Record
Event ID: SYS-EVT-006

Event Name: Unauthorized Access Control List (ACL) Traffic Denial
Jira Issue: SEC-106
Contributor: Hasan Dagdelen
Date and Time: 2026-10-27 15:10:00 UTC
Source Device: RTR-INT-01 (10.20.99.1)
Packet Tracer file/version: Enterprise_Monitored_v3_0.pkt / Cisco Packet Tracer 8.2.1

Evidence status: Observed

Generating Action
Initiated a ping/ICMP or TCP connection attempt from an endpoint in VLAN 10 (USERS subnet 10.20.10.0/24) targeting a restricted resource in VLAN 20 (ADMINS) or VLAN 50 (SYSLOG_VLAN), matching an explicit deny statement configured with logging (deny ip any any log) on RTR-INT-01's interface.

Log Appearance
Plaintext
%SEC-6-IPACCESSLOGP: list 101 denied ip 10.20.10.15 -> 10.20.20.5 (10/0), 1 packet
Explanation
Legitimate Activity: A user misconfiguring an application endpoint, typing an incorrect IP address into a administrative management tool, or an automated discovery tool running from an authorized subnet.

Malicious Activity: An attacker or compromised internal host performing internal network reconnaissance, subnet scanning, or attempting lateral movement into restricted administrative segments (VLAN 20/VLAN 50).

Distinguishing Evidence
Traffic Volume/Frequency: Multiple denied packets across sequential target IP addresses or port numbers indicate active port scanning, whereas a single isolated packet suggests a user typo or misconfiguration.

Source MAC/IP Correlation: Cross-referencing the source IP 10.20.10.15 against DHCP binding tables to identify the specific physical host.

##Classification
Syslog severity, if available: Informational (Severity 6)

Assigned analytical severity and rationale: Medium. Denied cross-VLAN traffic indicates that network boundary controls are working, but repeated denials suggest active internal scanning or policy violation.

Analytical question: Is the source IP associated with an authorized administrative endpoint, and was this traffic part of a single connection attempt or a sequential scan?

Response category: Correlate

Security Property affected: Confidentiality / Integrity

Limitations
The ACL log entry confirms that the packet was intercepted and dropped at the router interface, but it cannot determine whether the payload contained malicious exploit traffic or simple echo requests.

---

Security Event Record
Event ID: SEC-EVT-007

Event Name: Port Security Violation and Interface Shutdown

Jira Issue: SEC-107

Contributor: [Your Name / Group]

Date and Time: 2026-10-27 15:25:30 UTC

Source Device: SW-ACC-01 (10.20.99.11)

Packet Tracer file/version: Enterprise_Monitored_v3.pkt / Cisco Packet Tracer 8.2.1

Evidence status: Observed

Generating Action
Connected an unauthorized rogue device (LAPTOP-ROGUE-01) with an unknown MAC address into access switch port Fa0/4 on SW-ACC-01 where switchport port-security was configured with a maximum MAC limit of 1 and violation mode set to shutdown.

Log Appearance
Plaintext
%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC a1b2.c3d4.e5f6 on port FastEthernet0/4.
%LINK-4-ERROR: err-disable state detected on FastEthernet0/4.
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
Explanation
Legitimate Activity: An authorized employee swapping out their workstation or plugging an unapproved personal device (BYOD) into a wall jack bound to a specific MAC address.

Malicious Activity: An attacker disconnecting an authorized office PC and connecting a rogue device (LAPTOP-ROGUE-01) to gain physical access to VLAN 10.

Distinguishing Evidence
Switch MAC Table (show mac address-table): Confirms that the MAC address a1b2.c3d4.e5f6 does not match the configured static or sticky MAC address on port Fa0/4.

Port Status (show interfaces Fa0/4 status): Shows the port state as err-disabled.

Classification
Syslog severity, if available: Critical (Severity 2) / Warning (Severity 4)

Assigned analytical severity and rationale: High. A port security violation indicates an unauthorized physical network attachment attempt, resulting in immediate port disabling.

Analytical question: What device MAC address triggered the violation on Fa0/4, and is that MAC address registered in company asset inventories?

Response category: Immediate action

Security Property affected: Integrity / Availability

Limitations
This log event identifies the offending MAC address and port, but does not provide details regarding the operating system, hostname, or intent of the unauthorized device plugged into the port.

---

Security Event Record
Event ID: AUTH-EVT-008

Event Name: Privilege Exec Mode Authentication Failure (Enable Secret)

Jira Issue: SEC-108

Contributor: [Your Name / Group]

Date and Time: 2026-10-27 15:40:12 UTC

Source Device: SW-DIST-01 (10.20.99.2)

Packet Tracer file/version: Enterprise_Monitored_v3.pkt / Cisco Packet Tracer 8.2.1

Evidence status: Observed

Generating Action
Entered the enable command on SW-DIST-01 via console or VTY line and deliberately supplied an incorrect administrative password three consecutive times.

Log Appearance
Plaintext
%SYS-4-BADPASS: Password incorrect
Explanation
Legitimate Activity: An authorized administrator misTyping their privileged password or confusing login credentials across environments.

Malicious Activity: An attacker who has achieved user-level CLI access attempting to brute-force or dictionary-attack the enable password to gain full administrative privilege over SW-DIST-01.

Distinguishing Evidence
Failure Count & Rate: Rapid, sequential BADPASS logs occurring within seconds indicate an automated brute-force attack.

Terminal Source (show users / VTY log): Identifies whether the failed attempt originated from the local console port or a remote VTY/SSH IP session.

Classification
Syslog severity, if available: Warning (Severity 4)

Assigned analytical severity and rationale: Medium to High. Repeated privilege escalation failures suggest potential credential stuffing or active internal privilege escalation attempts.

Analytical question: How many consecutive failed password attempts occurred, and from what IP address or line interface were they initiated?

Response category: Immediate action

Security Property affected: Confidentiality / Integrity

Limitations
The %SYS-4-BADPASS syslog message records that an incorrect password was submitted, but does not record the actual string entered or the user account attempted.

---

Security Event Record
Event ID: NET-EVT-009

Event Name: Unscheduled Interface Administrative State Change (Shutdown)

Jira Issue: SEC-109

Contributor: [Your Name / Group]

Date and Time: 2026-10-27 15:55:04 UTC

Source Device: SW-ACC-02 (10.20.99.12)

Packet Tracer file/version: Enterprise_Monitored_v3.pkt / Cisco Packet Tracer 8.2.1

Evidence status: Observed

Generating Action
Entered global interface configuration mode on SW-ACC-02 for an active port (interface FastEthernet0/2) and executed the shutdown command without prior operational notice.

Log Appearance

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
Explanation
Legitimate Activity: Network administrator disabling an unused or compromised port during isolation procedures, or executing standard port maintenance.

Malicious Activity: An attacker with privilege access disabling key switch interfaces to cause a Denial of Service (DoS) for connected users or admin devices in VLAN 20.

Distinguishing Evidence
Change Window Approval: Verification whether a ticket existed requesting port Fa0/2 to be taken offline.

Concurrent Config Logs: Checking for %SYS-5-CONFIG_I logs from the same timestamp to determine which admin line executed the shutdown.

Classification
Syslog severity, if available: Notice (Severity 5)

Assigned analytical severity and rationale: Medium. Disabling an active interface directly impacts network availability; unannounced port shutdowns require immediate operational check.

Analytical question: Was the administrative shutdown of Fa0/2 performed in response to a security incident or an approved operational request?

Response category: Correlate

Security Property affected: Availability

Limitations
The interface state change log confirms the port was brought down, but does not prove whether the action was taken maliciously or as part of manual incident containment.

---

Security Event Record
Event ID: SYS-EVT-010

Event Name: Centralized Syslog Service Interruption / Logging Target Unreachable

Jira Issue: SEC-110

Contributor: [Your Name / Group]

Date and Time: 2026-10-27 16:10:22 UTC

Source Device: RTR-EDGE-01 (10.20.99.1)

Packet Tracer file/version: Enterprise_Monitored_v3.pkt / Cisco Packet Tracer 8.2.1

Evidence status: Observed

Generating Action
Removed or misconfigured the logging target IP address on RTR-EDGE-01 (no logging host 10.20.50.10) or administratively shut down the uplink interface to SRV-SYSLOG, disrupting telemetry transport.

Log Appearance
Plaintext
%SYS-6-LOGGINGHOST_STARTSTOP: Logging to host 10.20.50.10 stopped
Explanation
Legitimate Activity: Network engineer reconfiguring central logging destinations during a SIEM migration or updating syslog host IP addresses.

Malicious Activity: An adversary attempting defense evasion by disabling remote log forwarding prior to launching further attacks, hiding their tracks from central SOC monitoring.

Distinguishing Evidence
Local Buffer Inspection (show logging): Reviewing the router's local memory buffer to see commands executed immediately before log forwarding stopped.

Network Path Health: Ping and traceroute checks to verify if SRV-SYSLOG (10.20.50.10) is physically unreachable or if remote logging was manually turned off.

Classification
Syslog severity, if available: Informational (Severity 6) / Notice (Severity 5)

Assigned analytical severity and rationale: High. Loss of telemetry from an edge infrastructure device blinds security monitoring and directly violates logging policy prerequisites.

Analytical question: Was the termination of log forwarding caused by a network routing failure or a deliberate manual configuration change?

Response category: Immediate action

Security Property affected: Integrity / Availability

Limitations
When centralized logging is stopped, subsequent actions performed on RTR-EDGE-01 will no longer transmit to SRV-SYSLOG, requiring local console access to retrieve remaining forensic evidence.
