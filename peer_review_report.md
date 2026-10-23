# Peer Review

- **Artifact reviewed:** Centralized Logging & Network Time Synchronization Baseline (`CYBEROPS07_Monitoring_Baseline_Review.pkt`, running configs, Syslog/NTP server evidence)
- **Artifact link:** `https://github.com/CyberOps-Team-7/CYBEROPS-07/blob/main/Topology/CYBEROPS07_Monitoring_Baseline_Review.pkt`
- **Version or commit reviewed:** `c8f12a4` (Sprint 2 Infrastructure Review Commit)
- **Related Jira issue:** `SCRUM-29`, `SCRUM-30`, `SCRUM-31` (Parent Epics: `SCRUM-5`, `SCRUM-6`)
- **Reviewer:** Member 3 (Validation & Coverage Lead)
- **Review date:** October 23, 2026

## Review Scope
Comprehensive audit of the Sprint 2 monitoring and time synchronization foundation across all 7 network infrastructure devices (`RTR-EDGE-01`, `RTR-INT-01`, `RTR-INT-02`, `SW-DIST-01`, `SW-ACC-01`, `SW-ACC-02`, `SW-ACC-03`) and central servers (`SRV-SYSLOG`, `SRV-NTP`):

1. **Syslog Centralization & Forwarding (`SCRUM-29` & `SCRUM-30`):** Verified that `SRV-SYSLOG` (`10.20.50.10`) is enabled and that all routers and switches carry `logging host 10.20.50.10`, proper timestamp formatting (`service timestamps log datetime msec`), and explicit logging trap levels.
2. **Functional Event Logging Test (Test T-01):** Validated live Syslog message receipt at `SRV-SYSLOG` following interface status changes (`shutdown` / `no shutdown`) across routers and access switches.
3. **Network Time Protocol Baseline (`SCRUM-31`):** Audited `SRV-NTP` (`10.20.50.20`) setup, `ntp server 10.20.50.20` CLI configurations, device clock convergence, and baseline timestamp alignment across network devices prior to Sprint 3 clock-skew testing.
4. **Simulator Limitations & Documentation:** Identified and recorded Cisco Packet Tracer quirks regarding status commands and log field formatting.

## Findings and Corrections

| Finding | Required correction | Owner | Status | Correction Evidence |
|---|---|---|---|---|
| **F-01:** `SW-ACC-01`, `SW-ACC-02`, and `SW-ACC-03` are missing Syslog host configuration (`logging host 10.20.50.10`) and log timestamps (`service timestamps log datetime msec`). Switches log locally to console but do not forward logs to `SRV-SYSLOG`. | Configure `logging host 10.20.50.10`, `logging trap debugging`, and `service timestamps log datetime msec` on `SW-ACC-01`, `SW-ACC-02`, and `SW-ACC-03`. | Hussein | Changes required | Pending updated `show run \| include logging` output from access switches. |
| **F-02:** `SW-ACC-01`, `SW-ACC-02`, and `SW-ACC-03` are missing Network Time Protocol configuration (`ntp server 10.20.50.20`). | Configure `ntp server 10.20.50.20` across all three access switches. | Hussein | Changes required | Pending updated `show run \| include ntp` output from access switches. |
| **F-03:** All three routers (`RTR-EDGE-01`, `RTR-INT-01`, `RTR-INT-02`) and distribution switch `SW-DIST-01` have debug timestamps explicitly disabled (`no service timestamps debug datetime msec`). | Run `service timestamps debug datetime msec` and `logging trap debugging` on `RTR-EDGE-01`, `RTR-INT-01`, `RTR-INT-02`, and `SW-DIST-01`. | Hussein | Changes required | Pending updated CLI output showing active debug timestamps. |
| **F-04:** Minor clock variance (1–2 minutes) observed across router clocks (`0:35:12` to `0:36:09`) relative to `SRV-NTP` (`12:37:33 AM`). | Document Packet Tracer NTP polling convergence behavior in the limitations log. Fast-forward simulation time to allow full clock synchronization before capturing final baseline screenshots. | Hussein | Changes required | Documented NTP polling behavior; awaiting synchronized `show clock` outputs. |
| **F-05:** `show ntp status` on routers displays `Clock is unsynchronized` despite accepting the NTP configuration and updating system time. | Document this display quirk as a Packet Tracer simulator limitation in `PT_Limitations.md` rather than reporting a configuration failure. | Hussein | Open | Pending entry in `PT_Limitations.md`. |

## Outcome

Changes required

While the core Syslog server (`SRV-SYSLOG` at `10.20.50.10`), NTP server (`SRV-NTP` at `10.20.50.20`), three routers, and distribution switch are functional and forwarding events, the three access switches (`SW-ACC-01`, `SW-ACC-02`, `SW-ACC-03`) currently lack Syslog forwarding and NTP settings. Additionally, debug timestamps need to be enabled across all routers. Once Hussein applies the missing configuration commands to the switches and enables debug timestamps, the infrastructure will be 100% complete and ready for Sprint 2 closeout.

## Follow-up Review

- **Verification date:** [Pending Hussein's fixes]
- **Version or commit reviewed:** [Pending updated commit ID]
- **Final Outcome:** [Pending final re-audit]
- **Notes:** Verification of corrections will occur once updated configuration files are committed to the repository.