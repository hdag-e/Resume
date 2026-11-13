# Incident #1 — Controlled Test Record & Sealed Evidence Package (`SCRUM-49`)

## 1. Metadata & Package Control
| Field | Details |
| :--- | :--- |
| **Incident ID** | Incident #1 (`SCRUM-49` / Appendix F Test `T-08`) |
| **Scenario Title** | Unauthorized Cross-Zone Probing & Router Control-Plane Compromise |
| **Stager / Lead** | Analyst Alpha |
| **Assigned Blind Reconstructor** | Analyst Beta (Eric — `SCRUM-50`) |
| **Lab Environment File** | `CYBEROPS07_Configuration_v3_5.pkt` (Isolated Execution Copy) |
| **Central NTP & Syslog Server** | `SRV-NTP` / `SRV-SYSLOG` (`10.20.50.10`) |

---

## 2. Pre-Execution Clock & NTP Verification (`SCRUM-176`)

Timestamp consistency across involved devices was verified prior to executing incident actions:

- **Central NTP Reference (`10.20.50.10`):** Stratum 1 / Reference Clock
- **`RTR-INT-01` (`10.20.99.2` / `10.20.250.2`):** Synchronized, Stratum 2, Reference `10.20.50.10`, Clock Offset `1.00 msec`
- **`RTR-EDGE-01` (`10.20.250.1` / `10.20.250.5`):** Synchronized, Stratum 2, Reference `10.20.50.10` (verified post-initial NTP poll)

---

## 3. Incident Scenario Narrative & Attack Path (`SCRUM-175`)

### 3.1 Executive Narrative (Private Ground Truth)
An unauthorized actor operating from an internal User segment workstation (`PC-USER-01`, `10.20.10.11`) conducts unauthorized ICMP probing (`EVT-04`) across trust boundaries toward the restricted Management (`10.20.99.2`) and Transit (`10.20.250.1` / `10.20.250.2`) interfaces. Next, the actor targets the SSH management plane of `RTR-INT-01` (`10.20.99.2`), executing a 5-attempt brute-force login sequence against `secadmin` (`EVT-03`) before successfully authenticating. Once inside privileged EXEC mode, the actor enters global configuration mode (`EVT-02`) to modify router parameters, bounces transit interface `GigabitEthernet0/0/0` (`10.20.250.2`) (`EVT-01`) to disrupt upstream connectivity to `RTR-EDGE-01` (`10.20.250.1`), and forces a control-plane reboot via `reload` (`EVT-05`).

### 3.2 Grading Hook (`T-08` — Earliest Evidence of Activity)
- **Earliest Observable Log Evidence:** The initial `%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: secadmin] [localport: 22]` authentication failure entries (`EVT-03`) at `12:24:47 UTC` on `RTR-INT-01` (`10.20.99.2`) forwarded to `SRV-SYSLOG` (`10.20.50.10`).
- **Pre-Log Activity (Simulator Gap):** Stage 1 cross-zone ICMP probing (`EVT-04`) occurred at `11:52:41.904 UTC` prior to Stage 2, but generates no syslog output due to Cisco Packet Tracer's ACL `log` keyword limitation (Section 5).

---

## 4. Staged Action Sequence & Observed Evidence (`SCRUM-176`)

| Stage | MITRE ATT&CK | Catalogue ID | Target Device (Interface / IP) | Deliberate Generating Action | Observed Syslog Message | Exact Timestamp (UTC) | Evidence Reference |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Recon** | `T1046` (Network Service Discovery) | **`EVT-04`** | `RTR-EDGE-01` (`10.20.250.1`) / `RTR-INT-01` (`10.20.99.2`) | From `PC-USER-01` (`10.20.10.11`), execute ICMP pings toward restricted management/transit IPs (`10.20.99.2`, `10.20.250.1`). | *None emitted to syslog (Packet Tracer ACL `log` limitation — documented analytically).* | `11:52:41.904 UTC` | `01_CrossZone_ICMP.png` |
| **2. Initial Access** | `T1110` (Brute Force) | **`EVT-03`** | `RTR-INT-01` (`Gi0/0/1` — `10.20.99.2`) | Execute 5 failed SSH login attempts from `PC-USER-01` (`10.20.10.11`) using username `secadmin:cisco123`, followed by 1 valid login. | `%SEC_LOGIN-5-LOGIN_FAILED` (x5) followed by `%SEC_LOGIN-5-LOGIN_SUCCESS` | `12:24:47–12:24:52 UTC` | `02_RTR-INT-01_Auth_Logs.png` |
| **3. Tampering** | `T1565` (Data/Config Manipulation) | **`EVT-02`** | `RTR-INT-01` (`10.20.99.2`) | Enter `configure terminal`, modify interface/route parameters, and exit (`end`). | `%SYS-5-CONFIG_I: Configured from console by console` | `12:25:40.399 UTC` | `03_RTR-INT-01_Config_Change.png` |
| **4. Impact (Part A)** | `T1498` (Network DoS) | **`EVT-01`** | `RTR-INT-01` (`Gi0/0/0` — `10.20.250.2`) | Enter `interface GigabitEthernet0/0/0`, execute `shutdown`, wait ~10s, then execute `no shutdown`. | `%LINK-5-CHANGED` and `%LINEPROTO-5-UPDOWN` (administratively down $\rightarrow$ up) | `12:28:56.322 UTC` | `04_RTR-INT-01_Int_Flap.png` |
| **5. Impact (Part B)** | `T1529` (System Shutdown/Reboot) | **`EVT-05`** | `RTR-INT-01` (`10.20.99.2`) | Execute `reload` from privileged EXEC mode (without saving startup-config) to force a device restart. | `%LINK-3-UPDOWN` / `%LINEPROTO-5-UPDOWN` / `%LINK-5-CHANGED` (`FastEthernet0/1` down $\rightarrow$ up) | `12:36:43.673 UTC` | `05_RTR-INT-01_Reload.png` |

### 4.1 Raw Exported Syslog Extract (`SRV-SYSLOG` — `10.20.50.10`)
```
%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: secadmin] [localport: 22] [Reason: Login Authentication Failed] at 12:24:47 UTC Tue Oct 6 2026
%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: secadmin] [localport: 22] [Reason: Login Authentication Failed] at 12:24:47 UTC Tue Oct 6 2026
%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: secadmin] [localport: 22] [Reason: Login Authentication Failed] at 12:24:48 UTC Tue Oct 6 2026
%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: secadmin] [localport: 22] [Reason: Login Authentication Failed] at 12:24:49 UTC Tue Oct 6 2026
%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: secadmin] [localport: 22] [Reason: Login Authentication Failed] at 12:24:50 UTC Tue Oct 6 2026
%SEC_LOGIN-5-LOGIN_SUCCESS: Login Success [user: secadmin] [localport: 0] at 12:24:52 UTC Tue Oct 6 2026
%SYS-5-CONFIG_I: Configured from console by console
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to down
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
%SYS-5-CONFIG_I: Configured from console by console
%LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
%SEC_LOGIN-5-LOGIN_SUCCESS: Login Success [user: admin] [localport: 0] at 00:02:21 UTC Mon Mar 1 1993
%LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
%SEC_LOGIN-5-LOGIN_SUCCESS: Login Success [user: admin] [localport: 0] at 00:02:57 UTC Mon Mar 1 1993
%LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
```

---

## 5. Packet Tracer Simulator Limitations Encountered (`SCRUM-176`)

1. **ACL Packet Drop Logging (`EVT-04`):** Cisco Packet Tracer IOS firmware does not emit syslog messages for the `log` keyword on extended ACL entries (`deny icmp any any log`), leaving Stage 1 cross-zone reconnaissance probes (`10.20.10.11` $\rightarrow$ `10.20.99.2` / `10.20.250.1`) invisible in `SRV-SYSLOG`.
2. **Command-Level Audit Accounting (`EVT-02`):** Standard Cisco IOS emits `%SYS-5-CONFIG_I` upon entering/exiting configuration mode, but Packet Tracer lacks TACACS+ command accounting or `archive` configuration diff logging to record exact CLI commands executed during Stage 3.
3. **Source Workstation Attribution (`EVT-03`):** Local authentication failure logs record the attempted username (`secadmin`) and line/localport (`[localport: 22]`), but do not append source workstation hardware details or automated vs. manual session attributes.
4. **Baseline Credential Isolation & Console Attribution (`EVT-02`):** Because the Week 3 frozen `username admin` MD5 (`secret 5`) hash could not be authenticated remotely without modifying master baseline credentials, a temporary local account (`secadmin`) was provisioned in an isolated execution copy for Stage 2, while Stage 3 configuration changes logged locally (`Configured from console by console`).
5. **Pre-Network Boot Syslog Suppression & Post-Reload Epoch Clock (`EVT-05`):** Cisco Packet Tracer does not transmit `%SYS-3-RESTART` over remote syslog during `reload` because the restart occurs before IP routing and the UDP `514` logging socket initialize; instead, the reboot is observable on `SRV-SYSLOG` via the `FastEthernet0/1` `%LINK-3-UPDOWN` / `%LINEPROTO-5-UPDOWN` (`down` $\rightarrow$ `up`) state transitions and transient default epoch timestamps (`Mar 1 1993`) emitted prior to post-boot NTP resynchronization.

---

## 6. Sealing & Blind Reconstruction Sufficiency Verification (`SCRUM-177`)

- **Evidence Sealing Declaration:**  
  Event generation for Incident #1 (`SCRUM-49`) concluded at `12:36:43.673 UTC` (Tue Oct 6 2026). The exported syslog file (`Incident_01_Syslog_Export.txt`) and supporting screenshots are frozen and sealed with no post-capture modifications.
- **Sufficiency Check:**  
  * `[x]` All generated events map exclusively to the Sprint 3 Event Catalogue (`EVT-01` through `EVT-05`).
  * `[x]` Raw syslog entries contain consistent, NTP-synchronized timestamps across primary attack phases (with expected post-reload `1993` epoch behavior documented).
  * `[x]` Analyst Beta (`SCRUM-50`) can reconstruct the full timeline and identify the earliest observable evidence (`EVT-03` auth failures at `12:24:47 UTC`) using only the sealed raw logs and Sprint 3 documentation.
