# Incident #3 — Controlled Test Record & Sealed Evidence Package (`SCRUM-53`)

## 1. Metadata & Package Control
| Field | Details |
| :--- | :--- |
| **Incident ID** | Incident #3 (`SCRUM-53` / Subtasks `SCRUM-191`–`SCRUM-194` / Appendix F Test `T-10`) |
| **Scenario Title** | Physical Rogue Endpoint Insertion, Switch Security Bypass & Server VLAN Outage |
| **Stager / Lead** | Analyst Alpha |
| **Assigned Blind Reconstructor** | Analyst Beta (Non-Stager — `T-10` Validation) |
| **Lab Environment File** | `CYBEROPS07_Incident3_Exec.pkt` (Isolated Execution Copy of `CYBEROPS07_Configuration_v3_5.pkt`) |
| **Central NTP & Syslog Server** | `SRV-NTP` / `SRV-SYSLOG` (`10.20.50.10`) |
| **Package Status** | **SEALED POST-EXECUTION (`SCRUM-194` Complete)** |

---

## 2. Pre-Execution Clock & NTP Verification (`SCRUM-191`)

Timestamp consistency across involved switching infrastructure was verified prior to executing incident actions:

- **Central NTP Reference (`10.20.50.10`):** Stratum 1 / Reference Clock
- **`SW-ACC-01` (`10.20.99.11`):** Synchronized, Stratum 2, Reference `10.20.50.10`, Clock Offset `0.00 msec` (`13:36:31.305 UTC Tue Oct 6 2026`)
- **`SW-DIST-01` (`10.20.50.1` / `10.20.99.1`):** Synchronized, Stratum 2, Reference `10.20.50.10`

---

## 3. Incident Scenario Narrative & Attack Path (`SCRUM-191`)

### 3.1 Executive Narrative (Private Ground Truth)
An unauthorized actor gains physical access to a wall jack connected to `SW-ACC-01` (`FastEthernet0/6`, Guest/Untrusted segment `192.168.50.0/24`) and connects an unmanaged rogue laptop (`LAPTOP-ROGUE-01`, MAC `0001.6350.4CCD`). `SW-ACC-01` (`10.20.99.11`) immediately detects the unauthorized MAC address via Port Security (`%PORT_SECURITY-2-PSECURE_VIOLATION`) and places `FastEthernet0/6` into `err-disable` state (`EVT-01`) at `01:36:38.216 PM` (`13:36:38 UTC`). Blocked at `Fa0/6`, the actor accesses an internal workstation (`PC-USER-01`, `10.20.10.11`), brute-forces SSH login credentials (`admin`) against `SW-ACC-01` (`10.20.99.11`) (`EVT-03`), enters global configuration mode (`EVT-02`) at `01:40:50.771 PM`, strips Port Security enforcement (`no switchport port-security`) from `FastEthernet0/6`, and bounces the interface (`shutdown` / `no shutdown`), bringing `FastEthernet0/6` back `up` (`EVT-01`) at `01:42:16.953 PM`. Next, the actor reconnects `LAPTOP-ROGUE-01` (`192.168.50.10`) onto the unguarded `Fa0/6` port, brute-forces SSH credentials (`admin`) against `SW-DIST-01` (`10.20.99.1` / logging source `10.20.50.1`) (`EVT-03`) between `01:42:47.538 PM` and `01:42:49.859 PM`, enters global configuration mode (`EVT-02`), and shuts down `interface Vlan30` (`10.20.30.1`) (`EVT-01`) at `01:46:18.308 PM`, severing enterprise connectivity to the protected Application Server segment (`SRV-APP-01` at `10.20.30.11`). Finally, the actor tests reachability to `10.20.30.11` from `LAPTOP-ROGUE-01` (`192.168.50.10`), receiving `Destination host unreachable` from `192.168.50.1` while generating zero remote syslog telemetry (`T-10` undetermined-evidence boundary).

### 3.2 Grading Hook (`T-10` — Earliest Evidence & Undetermined-Evidence Boundary)
- **Earliest Observable Log Evidence:** `%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 0001.6350.4CCD on port FastEthernet0/6` and `%PM-4-ERR_DISABLE` on `SW-ACC-01` (`10.20.99.11`) at `01:36:38.216 PM` (`13:36:38 UTC`).
- **Explicitly Undetermined from Available Logs (`T-10` Requirement):**
  1. **Pivot Workstation Identity on `SW-ACC-01`:** Because Packet Tracer's Catalyst 2960 IOS does not emit `%SEC_LOGIN` syslog messages over UDP 514, the logs show `Fa0/6` disabled at `01:36:38.216 PM` and re-enabled at `01:42:16.953 PM` after `%SYS-5-CONFIG_I` (`01:40:50.771 PM`), but do not reveal which workstation (`PC-USER-01`, `10.20.10.11`) authenticated to `SW-ACC-01` to remove Port Security.
  2. **Exact CLI Commands Typed:** `%SYS-5-CONFIG_I` on both `10.20.99.11` (`01:40:50.771 PM`) and `10.20.50.1` (`01:46:20.395 PM`) confirms configuration mode was exited, but Packet Tracer lacks TACACS+ command accounting to record the exact CLI commands typed (`no switchport port-security` and `interface vlan 30` ->$ `shutdown`).
  3. **Post-Outage Endpoint Activity:** Subsequent ICMP probes from `LAPTOP-ROGUE-01` (`192.168.50.10`) toward `10.20.30.11` produce no syslog entries.

---

## 4. Staged Action Sequence & Observed Evidence (`SCRUM-192` / `SCRUM-193`)

| Stage | MITRE ATT&CK | Catalogue ID | Target Device (Interface / IP) | Deliberate Generating Action | Observed Syslog Message | Exact Timestamp | Evidence Reference |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Rogue Endpoint Connection** | `T1200` (Hardware Additions) | **`EVT-01`** | `SW-ACC-01` (`Fa0/6` — `10.20.99.11`) | Connect `LAPTOP-ROGUE-01` (MAC `0001.6350.4CCD`) to Port-Security-protected `Fa0/6` and transmit a frame. | `%PORT_SECURITY-2-PSECURE_VIOLATION` (`0001.6350.4CCD`), `%PM-4-ERR_DISABLE`, `%LINK-5-CHANGED` / `%LINEPROTO-5-UPDOWN` (`down`) | `01:36:38.216 PM` (`13:36:38 UTC`) | `Incident_03_Syslog_Export.txt` (`01:36:38.216 PM` capture) |
| **2. Access Switch Pivot & Bypass** | `T1110` / `T1562.001` (Impair Defenses) | **`EVT-03` + `EVT-02` + `EVT-01`** | `SW-ACC-01` (`Fa0/6` — `10.20.99.11`) | From `PC-USER-01` (`10.20.10.11`), brute-force SSH (`admin`) into `10.20.99.11`, enter `conf t`, execute `no switchport port-security` on `Fa0/6`, and bounce `Fa0/6` (`shutdown` / `no shutdown`). | `%SYS-5-CONFIG_I` (`01:40:50.771 PM`) followed by `%LINK-5-CHANGED` / `%LINEPROTO-5-UPDOWN` (`Fa0/6` `up` at `01:42:16.953 PM`) | `01:40:50.771 PM` – `01:42:16.953 PM` | `02_SW-ACC-01_SSH_PortSec_Bypass.png` (`image.png`) |
| **3. Distribution Pivot & VLAN 30 Outage** | `T1110` / `T1489` (Service Stop) | **`EVT-03` + `EVT-02` + `EVT-01`** | `SW-DIST-01` (`Vlan30` — logging IP `10.20.50.1`) | From `LAPTOP-ROGUE-01` (`192.168.50.10`), brute-force SSH (`admin`) into `SW-DIST-01` (`10.20.99.1`), enter `conf t`, and execute `shutdown` on `interface Vlan30` (`10.20.30.1`). | `%SEC_LOGIN-5-LOGIN_FAILED` (x2) & `%SEC_LOGIN-5-LOGIN_SUCCESS` (`01:42:47`–`01:42:49 PM`), `%LINK-5-CHANGED` / `%LINEPROTO-5-UPDOWN` (`Vlan30` `down` at `01:46:18.308 PM`), `%SYS-5-CONFIG_I` (`01:46:20.395 PM`) | `01:42:47.538 PM` – `01:46:20.395 PM` | `03_SW-DIST-01_SSH_Vlan30_Shutdown.png` (`image1.png`) |
| **4. Post-Outage Verification Probe** | `T1046` (Network Discovery) | **`EVT-04` / Undetermined (`T-10`)** | `LAPTOP-ROGUE-01` (`192.168.50.10`) -> `SRV-APP-01` (`10.20.30.11`) | From `LAPTOP-ROGUE-01` (`192.168.50.10`), run `ping 10.20.30.11` to verify `VLAN 30` service outage (`Destination host unreachable` from `192.168.50.1`). | *None emitted to syslog (Packet Tracer ICMP/routing drop logging limitation — documented in Section 5).* | `01:46:35 PM` *(approx. post-exit)* | `04_Laptop-Rogue-01_PostBypass_Ping.png` (`image2.png`) |

### 4.1 Raw Exported Syslog Extract (`SRV-SYSLOG` — `10.20.50.10`)
```
10.06.2026 01:46:20.395 PM		10.20.50.1	%SYS-5-CONFIG_I: Configured from console by console
10.06.2026 01:46:18.308 PM		10.20.50.1	%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan30, changed state to down
10.06.2026 01:46:18.308 PM		10.20.50.1	%LINK-5-CHANGED: Interface Vlan30, changed state to administratively down
10.06.2026 01:42:49.859 PM		10.20.50.1	%SEC_LOGIN-5-LOGIN_SUCCESS: Login Success [user: admin] [localport: 0] at 13:42:49 UTC Tue Oct 6 2026
10.06.2026 01:42:48.254 PM		10.20.50.1	%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: admin] [localport: 22] [Reason: Login Authentication Failed] at 13:42:48 UTC Tue Oct 6 2026
10.06.2026 01:42:47.538 PM		10.20.50.1	%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: admin] [Source: 192.168.50.10] [localport: 22] [Reason: Login Authentication Failed] at 13:42:47 UTC Tue Oct 6 2026
10.06.2026 01:42:16.953 PM		10.20.99.11	%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/6, changed state to up
10.06.2026 01:42:16.953 PM		10.20.99.11	%LINK-5-CHANGED: Interface FastEthernet0/6, changed state to up
10.06.2026 01:40:50.771 PM		10.20.99.11	%SYS-5-CONFIG_I: Configured from console by console
10.06.2026 01:36:38.216 PM		10.20.99.11	%LINK-5-CHANGED: Interface FastEthernet0/6, changed state to up
10.06.2026 01:36:38.216 PM		10.20.99.11	%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/6, changed state to down
10.06.2026 01:36:38.216 PM		10.20.99.11	%LINK-5-CHANGED: Interface FastEthernet0/6, changed state to administratively down
10.06.2026 01:36:38.216 PM		10.20.99.11	%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/6, changed state to down
10.06.2026 01:36:38.216 PM		10.20.99.11	%PM-4-ERR_DISABLE: psecure-violation error detected on Fa0/6, putting Fa0/6 in err-disable state.
10.06.2026 01:36:38.216 PM		10.20.99.11	%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 0001.6350.4CCD on port FastEthernet0/6.
```
---

## 5. Packet Tracer Simulator Limitations Encountered (SCRUM-193)
1. **Layer-2 Switch SSH Auth Syslog Suppression (SW-ACC-01):** Cisco Packet Tracer's Catalyst 2960 IOS image (SW-ACC-01, 10.20.99.11) does not emit %SEC_LOGIN-5-LOGIN_FAILED or %SEC_LOGIN-5-LOGIN_SUCCESS messages to remote syslog during VTY/SSH authentication (unlike the Catalyst 3560 multilayer switch image on SW-DIST-01), leaving the Stage 2 SSH login source (10.20.10.11) unrecorded in SRV-SYSLOG.
2. **Command-Level Audit Accounting (EVT-02):** Both SW-ACC-01 (10.20.99.11) and SW-DIST-01 (10.20.50.1) emit %SYS-5-CONFIG_I: Configured from console by console upon exiting configuration mode (even when configured over SSH VTY lines), and Packet Tracer lacks TACACS+ command accounting or archive diff logging to record exact CLI commands executed.
3. **Egress Syslog Source Interface (SW-DIST-01):** Because SW-DIST-01 reaches SRV-SYSLOG (10.20.50.10) directly via its Vlan50 SVI without logging source-interface Vlan99, SW-DIST-01 syslog messages carry source IP 10.20.50.1 rather than its Vlan99 management IP (10.20.99.1).
4. **Post-Compromise ICMP / Unreachable Telemetry Gap (EVT-04):** Packet Tracer does not emit syslog messages for ICMP probes or Destination host unreachable responses generated when LAPTOP-ROGUE-01 (192.168.50.10) attempted to reach 10.20.30.11 during Stage 4.

---

6. Sealing & Blind Reconstruction Sufficiency Verification (SCRUM-194)
- **Evidence Sealing Declaration:**
Event generation for Incident #3 (SCRUM-53) concluded at 01:46:20.395 PM (13:46:20 UTC, Tue Oct 6 2026). The raw syslog export (Incident_03_Syslog_Export.txt) and supporting screenshots (01–04) are frozen and sealed with no post-capture modifications.
- **Sufficiency Check:**
[x] Distinct from Incidents #1 and #2 (targets Layer-2 Port Security on SW-ACC-01 and Layer-3 SVI availability on SW-DIST-01).
[x] All generated events map to the Sprint 3 Event Catalogue (EVT-01 through EVT-04).
[x] Raw syslog entries contain NTP-synchronized timestamps across 10.20.99.11 (SW-ACC-01) and 10.20.50.1 (SW-DIST-01).
[x] Sufficient for blind reconstruction (T-10), including clear evidence boundaries for the required undetermined-evidence statement.
