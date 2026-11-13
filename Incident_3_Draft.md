# Incident #3 — Scenario Design & Pre-Execution Test Record (`SCRUM-53` / `SCRUM-191`)

## 1. Metadata & Package Control
| Field | Details |
| :--- | :--- |
| **Incident ID** | Incident #3 (`SCRUM-53` / Subtask `SCRUM-191`) |
| **Scenario Title** | Rogue Endpoint Connection & Cross-VLAN Lateral Movement |
| **Scenario Lead / Stager** | Analyst Alpha |
| **Lab Environment File** | `CYBEROPS07_Configuration_v3_5.pkt` (Isolated Execution Copy) |
| **Central NTP & Syslog Server** | `SRV-NTP` / `SRV-SYSLOG` (`10.20.50.10`) |
| **Design Status** | **VERIFIED PRE-EXECUTION (`SCRUM-191` Complete — Ready for Staging)** |

---

## 2. Verified Topology & Addressing Reference

- **`SW-ACC-01` (Access Switch — NTP & Syslog Active to `10.20.50.10`):**
  - Management SVI (`Vlan99`): `10.20.99.11`
  - `FastEthernet0/2`: Connected to legitimate workstation `PC-USER-01` (`10.20.10.11`, `VLAN 10`)
  - `FastEthernet0/6`: Port Security enabled with static/sticky MAC restriction (`VLAN 10`); currently disabled (`shutdown`) due to unauthorized MAC enforcement
- **`SW-DIST-01` (Distribution / Inter-VLAN Router):**
  - `Vlan10` SVI (User Gateway): `10.20.10.1/24`[cite: 1]
  - `Vlan30` SVI (Application Server Gateway): `10.20.30.1/24`[cite: 1]
  - `Vlan50` SVI (Management/Telemetry Server Gateway): `10.20.50.1/24`[cite: 1]
  - `Vlan99` SVI (Network Infrastructure Management Gateway): `10.20.99.1/24`[cite: 1]
- **`VLAN 30` Application Servers:**
  - `SRV-APP-01`: `10.20.30.11` (Gateway `10.20.30.1`)[cite: 1]
  - `SRV-APP-02`: `10.20.30.12` (Gateway `10.20.30.1`)[cite: 1]
- **`RTR-INT-01` (Internal Core Router):**
  - Management Interface (`Gi0/0/1`): `10.20.99.2`

---

## 3. Incident Scenario Narrative & Attack Path (`SCRUM-191`)

### 3.1 Executive Narrative (Pre-Execution Design)
An unauthorized actor gains physical access to a wall jack in the User access layer (`VLAN 10`, `10.20.10.0/24`) connected to `SW-ACC-01` (`FastEthernet0/6`) and connects an unmanaged rogue laptop (`Laptop-Rogue-01`) with an unapproved MAC address. Because `Fa0/6` enforces Port Security MAC validation, `SW-ACC-01` immediately blocks the frame and shuts down `Fa0/6` (`EVT-01`). Unable to pass traffic on `Fa0/6`, the actor pivots onto an active `VLAN 10` access port (`SW-ACC-01` `Fa0/2`, `PC-USER-01`, `10.20.10.11`) and conducts cross-VLAN lateral discovery (`EVT-04`) across `SW-DIST-01` (`10.20.10.1` $\rightarrow$ `10.20.30.1`)[cite: 1] into the Application Server segment (`VLAN 30`, `10.20.30.0/24`, targeting `SRV-APP-01` at `10.20.30.11` and `SRV-APP-02` at `10.20.30.12`). Next, the actor attempts lateral credential reuse (`EVT-03`) against infrastructure management (`RTR-INT-01` at `10.20.99.2`) and modifies switch port configuration on `SW-ACC-01` (`10.20.99.11`) (`EVT-02`) to remove Port Security MAC enforcement (`no switchport port-security` on `Fa0/6`) so `Laptop-Rogue-01` can obtain persistent network access.

### 3.2 Distinction from Incidents #1 and #2
- **Layer-2 Access Origin:** Unlike Incident #1 (which targeted `RTR-INT-01`'s management plane directly over SSH), Incident #3 originates at the **Access Switch layer (`SW-ACC-01`)** via physical Port Security enforcement on `Fa0/6`.
- **East-West Lateral Pivot:** Rather than targeting WAN/Transit routing (`10.20.250.0/30`), lateral movement traverses `SW-DIST-01` SVIs from **User (`VLAN 10`, `10.20.10.0/24`) $\rightarrow$ Application Servers (`VLAN 30`, `10.20.30.0/24`)**.

---

## 4. Planned Action Sequence & Expected Telemetry (`SCRUM-191`)

> **Note:** Timestamps and observed syslog columns below are intentionally left blank prior to lab execution to satisfy `SCRUM-191` pre-execution documentation requirements.

| Stage | Attack Phase (MITRE ATT&CK) | Catalogue ID | Target Device (Interface / IP) | Planned Generating Action | Expected Telemetry / Syslog Signature | Planned Timestamp (UTC) | Observed Timestamp (UTC) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Rogue Endpoint Connection** | Physical Access / Hardware Additions (`T1200`) | **`EVT-01`** *(Port/Link Disruption)* | `SW-ACC-01` (`Fa0/6` — `VLAN 10`) | After resetting `Fa0/6` to enabled state in the isolated execution copy, connect `Laptop-Rogue-01` (unapproved MAC) to `SW-ACC-01` `Fa0/6` and send a frame (`ping 10.20.10.1`). | Port Security shutdown state transitions on `Fa0/6` (`%LINK-5-CHANGED` / `%LINEPROTO-5-UPDOWN` to `down` / `err-disabled`) forwarded to `SRV-SYSLOG` | *TBD (Pre-Execution)* | *Pending Execution* |
| **2. Cross-VLAN Lateral Recon** | Lateral Movement / Internal Discovery (`T1046`) | **`EVT-04`** *(Cross-Zone Probing)* | `SW-DIST-01` (`10.20.10.1`) $\rightarrow$ `SRV-APP-01` (`10.20.30.11`) / `SRV-APP-02` (`10.20.30.12`) | Pivoting from active `VLAN 10` workstation `PC-USER-01` (`SW-ACC-01` `Fa0/2`, `10.20.10.11`), sweep `VLAN 30` server targets (`10.20.30.11`, `10.20.30.12`) via ICMP probes across `SW-DIST-01` inter-VLAN routing. | ARP/CAM table population on `SW-DIST-01` (`VLAN 30`); *ICMP syslog gap documented per Packet Tracer ACL limitation*. | *TBD (Pre-Execution)* | *Pending Execution* |
| **3. Lateral Auth / Pivot Attempt** | Lateral Tool Transfer / Remote Services (`T1021`) | **`EVT-03`** *(Auth Failure / Access)* | `RTR-INT-01` (`Gi0/0/1` — `10.20.99.2`) | From `PC-USER-01` (`10.20.10.11`), execute 3–4 failed SSH login attempts toward `10.20.99.2` across `SW-DIST-01` (`Vlan99` `10.20.99.1`) to test lateral credential reuse. | `%SEC_LOGIN-5-LOGIN_FAILED` (`[localport: 22]`) forwarded to `SRV-SYSLOG` (`10.20.50.10`) | *TBD (Pre-Execution)* | *Pending Execution* |
| **4. Access Persistence / Tampering** | Impair Defenses / Config Change (`T1562`) | **`EVT-02`** *(Config Modification)* | `SW-ACC-01` (`Vlan99` — `10.20.99.11`) | Enter `configure terminal` on `SW-ACC-01`, remove Port Security MAC enforcement on `Fa0/6` (`no switchport port-security`), and bounce `Fa0/6` (`shutdown` / `no shutdown`) so `Laptop-Rogue-01` gains active `VLAN 10` connectivity. | `%SYS-5-CONFIG_I: Configured from console` and `%LINK-5-CHANGED` / `%LINEPROTO-5-UPDOWN` (`Fa0/6` `up`) | *TBD (Pre-Execution)* | *Pending Execution* |

---

## 5. Pre-Execution Verification Checklist (`SCRUM-191`)
- `[x]` Scenario narrative and 4-stage attack path drafted prior to lab execution.
- `[x]` Distinct from Incident #1 (focuses on `SW-ACC-01` Layer-2 Port Security on `Fa0/6` and `VLAN 10` $\rightarrow$ `VLAN 30` lateral movement).
- `[x]` Maps exclusively to established Sprint 3 Event Catalogue IDs (`EVT-01`, `EVT-04`, `EVT-03`, `EVT-02`).
- `[x]` All target interfaces, SVIs, server IPs (`10.20.30.11`, `10.20.30.12`), and device hostnames verified against `CYBEROPS07` baseline topology.
- `[x]` `SW-ACC-01` (`10.20.99.11`) verified active for NTP synchronization and syslog forwarding to `10.20.50.10`.
- `[ ]` Isolated execution copy of `CYBEROPS07_Configuration_v3_5.pkt` created prior to running Stage 1 (master baseline preserved).
