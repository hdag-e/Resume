# Incident #1 — Controlled Test Record & Sealed Evidence Package (`SCRUM-49`)

---

## 1. Metadata & Package Control
| Field | Details |
| :--- | :--- |
| **Incident ID** | Incident #1 (`SCRUM-49` / Appendix F Test `T-08`) |
| **Scenario Title** | External Perimeter Probing & Router Control-Plane Compromise |
| **Stager / Lead** | Analyst Alpha |
| **Assigned Blind Reconstructor** | Analyst Beta (Eric — `SCRUM-50`) |
| **Peer Reviewer (`SCRUM-178`)** | [] |
| **Lab Environment File** | `CYBEROPS07_Configuration_v3_5.pkt` |
| **Central Telemetry Collector** | `SRV-SYSLOG` (`10.20.50.10`) |

---

## 2. Pre-Execution Clock & NTP Verification (`SCRUM-176`)

Per project scope requirements, timestamp consistency across all involved devices was verified prior to executing any incident actions.

* **Central NTP / Syslog Reference (`10.20.50.10`):** Stratum 1 / Reference Clock
* **`RTR-INT-01`:**
  * **Status:** Synchronized, Stratum 2, Reference `10.20.50.10`
  * **Clock Offset:** `1.00 msec` (Verified prior to staging)
* **`RTR-EDGE-01`:**
  * **Status:** Synchronized, Stratum 2, Reference `10.20.50.10`
  * **Clock Offset:** `1.00 msec` (Verified prior to staging)

---

## 3. Incident Scenario Narrative & Attack Path (`SCRUM-175`)

### 3.1 Executive Narrative (Private Ground Truth)
An external threat actor operating from the untrusted external segment (`203.0.113.0/24`) conducts ICMP reconnaissance sweeps against internal network subnets through `RTR-EDGE-01`. After probing the perimeter, the actor targets the management plane of internal router `RTR-INT-01` (`10.20.99.2`), executing a VTY brute-force password guessing sequence (`EVT-03`) before successfully authenticating. Once inside privileged EXEC mode, the actor enters global configuration mode (`EVT-02`) to make unauthorized configuration changes, bounces a core interface (`EVT-01`) to disrupt link availability, and forces a control-plane reboot via `reload` (`EVT-05`).

### 3.2 Grading Hook (`T-08` — Earliest Evidence of Activity)
* **Earliest Observable Log Evidence:** The initial `%SEC_LOGIN-5-LOGIN_FAILED` authentication failure entries (`EVT-03`) on `RTR-INT-01` (`10.20.99.2`) forwarded to `SRV-SYSLOG` (`10.20.50.10`).
* **Pre-Log Activity (Simulator Gap):** Stage 1 perimeter ICMP probing (`EVT-04`) occurred immediately prior to Stage 2, but produces no syslog output due to Cisco Packet Tracer's native ACL `log` keyword limitation (see Section 5).

---

## 4. Staged Action Sequence & Observed Evidence (`SCRUM-176`)

| Stage | MITRE ATT&CK | Catalogue ID | Target Device (IP) | Deliberate Generating Action | Expected / Observed Syslog Message | Exact Timestamp (UTC) | Evidence Reference |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Recon** | `T1046` (Network Service Discovery) | **`EVT-04`** | `RTR-EDGE-01` | Sent ICMP ping sweeps from external host (`203.0.113.x`) toward internal protected/management subnets (`10.20.30.0/24`, `10.20.99.0/24`). | *None emitted to syslog (Packet Tracer ACL `log` limitation — documented analytically).* | `[HH:MM:SS UTC]` | `01_RTR-EDGE-01_ACL_Probe.png` |
| **2. Initial Access** | `T1110` (Brute Force) | **`EVT-03`** | `RTR-INT-01` (`10.20.99.2`) | Executed 3–4 failed VTY login attempts using username `admin` and invalid passwords, followed by 1 valid login. | `%SEC_LOGIN-5-LOGIN_FAILED: Login failed [user: admin]` followed by `%SEC_LOGIN-5-LOGIN_SUCCESS` | `[HH:MM:SS UTC]` | `02_RTR-INT-01_Auth_Logs.png` |
| **3. Tampering** | `T1565` (Data/Config Manipulation) | **`EVT-02`** | `RTR-INT-01` (`10.20.99.2`) | Entered `configure terminal`, modified interface/route parameters, and exited (`end`). | `%SYS-5-CONFIG_I: Configured from console by ...` | `[HH:MM:SS UTC]` | `03_RTR-INT-01_Config_Change.png` |
| **4. Impact (Part A)** | `T1498` (Network DoS) | **`EVT-01`** | `RTR-INT-01` (`10.20.50.1`) | Entered `interface GigabitEthernet0/0/1` (or `Fa0/1`), executed `shutdown`, waited ~10s, then `no shutdown`. | `%LINK-3-UPDOWN` and `%LINEPROTO-5-UPDOWN` (down $\rightarrow$ up) | `[HH:MM:SS UTC]` | `04_RTR-INT-01_Int_Flap.png` |
| **5. Impact (Part B)** | `T1529` (System Shutdown/Reboot) | **`EVT-05`** | `RTR-INT-01` (`10.20.50.1`) | Executed `reload` from privileged EXEC mode to force a device restart. | `%SYS-3-RESTART: System restarted --` and `%PMAN-5-EXITACTION: platform reset cause = Reload command` | `[HH:MM:SS UTC]` | `05_RTR-INT-01_Reload.png` |

### 4.1 Raw Exported Syslog Extract (`SRV-SYSLOG` — `10.20.50.10`)
```text
[Paste the exact raw syslog lines exported from SRV-SYSLOG here after execution]
```

---

## 5. Packet Tracer Simulator Limitations Encountered (`SCRUM-176`)

1. **ACL Packet Drop Logging (`EVT-04`):** Cisco Packet Tracer IOS firmware does not support runtime syslog generation for the `log` keyword on extended ACL entries (`deny icmp any any log`). Consequently, Stage 1 perimeter probing is blocked at the interface level but leaves no direct syslog entry at `SRV-SYSLOG` (`10.20.50.10`).
2. **Command-Level Audit Accounting (`EVT-02`):** Standard Cisco IOS logging emits `%SYS-5-CONFIG_I` when configuration mode is entered/exited, but Packet Tracer lacks TACACS+ command accounting or `archive` configuration diff logging to automatically record exact CLI lines modified during Stage 3.
3. **Source IP / Workstation Attribution (`EVT-03`):** Local authentication failure logs record the target username (`admin`) and line/localport, but do not natively append the initiating external workstation's hardware fingerprint or automated vs. manual session metadata.

---

## 6. Sealing & Blind Reconstruction Sufficiency Verification (`SCRUM-177`)

* **Evidence Sealing Declaration:**  
  All event generation for Incident #1 (`SCRUM-49`) concluded at `[HH:MM:SS UTC]`. The exported syslog text file (`Incident_01_Syslog_Export.txt`) and supporting screenshots have been frozen and sealed. No additional actions or log modifications were performed after sealing.
* **Sufficiency Check (Completed by Analyst Alpha):**  
  * `[x]` All generated events are mapped exclusively to the Sprint 3 Event Catalogue (`EVT-01` through `EVT-05`).
  * `[x]` Raw syslog entries contain consistent, NTP-synchronized timestamps across sources.
  * `[x]` Another analyst (Analyst Beta) can reconstruct the full attack timeline and identify the earliest observable evidence (`EVT-03` auth failures) using only the sealed raw logs and the team's Sprint 3 documentation—without verbal explanation.

---

## 7. Peer Review & Repository Integration (`SCRUM-178`)

* **Peer Reviewer Name:** `[Name]`
* **Review Date:** `[YYYY-MM-DD]`
* **Verification Checklist:**
  * `[ ]` Sealed raw syslog file (`Incident_01_Syslog_Export.txt`) opens cleanly and is readable.
  * `[ ]` Timestamps and device source IPs align with the `SCRUM-48` Manual Correlation Workbook format.
  * `[ ]` Blind reconstruction isolation preserved (Analyst Beta has not viewed this Controlled Test Record).
  * `[ ]` Package committed to GitHub repository (`Evidence/Sprint_4/Incident_01/`).
* **Reviewer Notes / Sign-Off:** `[Reviewer comments & initials]`
