# Requirements Traceability Matrix & Review Process (SCRUM-80)

## 1. Overview & Process Guidance
This document establishes end-to-end traceability across the CYBEROPS-07 project lifecycle[cite: 11]. It maps every master requirement (D1–D10) and acceptance test (T-01–T-15) from the project scope to its corresponding Jira Epic, subtasks, evidence file paths in GitHub, and validation status.

### Operating Rules for Team Members:
1. **Scope Mapping:** Every Jira Story or Subtask created must trace back to at least one master requirement or test ID.
2. **Evidence Linking:** Subtasks cannot be marked `Done` in Jira without linking relative file paths to logs, documentation, or `.pkt` configurations in the repository.
3. **Living Lifecycle Document:** This matrix is updated at the conclusion of each sprint review to reflect current progress, decisions, and discovered limitations.

---

## 2. Requirements Traceability Matrix (RTM)

| Deliverable / Test ID | Scope Requirement Description | Jira Epic | Jira Subtask | Evidence / File Path in Repository | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **D1 / T-01** | Enterprise Network Baseline & Centralized Logging | SCRUM-23 | SCRUM-75<br>SCRUM-77 | `docs/network_segmentation_and_roles.md`<br>`docs/baseline_config_requirements.md`<br>`topology.pkt` | Complete |
| **D2 / T-04, T-05** | Security Event Catalogue (15+ Deliberate Events) | SCRUM-24 | SCRUM-81 | `docs/event_catalogue.md` | In Progress |
| **D3 / T-06** | Event Taxonomy & Triage Procedure | SCRUM-24 | SCRUM-82 | `docs/event_taxonomy_and_triage.md` | In Progress |
| **D4 / T-07** | Escalation Matrix (Authority & Timeframes) | SCRUM-24 | SCRUM-83 | `docs/escalation_matrix.md` | In Progress |
| **D5 / T-02, T-03** | Correlation Workbook & Clock-Skew Demo | SCRUM-25 | TBD | `workbooks/correlation_workbook.xlsx` | Planned |
| **D6 / T-08, T-09, T-10** | Three Staged Multi-Stage Incident Reconstructions | SCRUM-25 | TBD | `reports/incidents/` | Planned |
| **D7 / T-11, T-12, T-13** | NIST SP 800-61 / ATT&CK Aligned Playbooks | SCRUM-26 | TBD | `playbooks/` | Planned |
| **D8 / T-14** | Week 8 Blind Exercise Live Incident Report | SCRUM-26 | TBD | `reports/blind_exercise_report.md` | Planned |
| **D9 / T-15** | Monitoring Coverage Gap Assessment | SCRUM-26 | TBD | `reports/coverage_gap_assessment.md` | Planned |
| **D10** | Final Report, Presentation, & Video Demo | SCRUM-26 | TBD | `final_delivery/` | Planned |

---

## 3. Decision, Change, & Known Limitation Record

### A. Architecture & Scope Decisions
- **DEC-01 (Sprint 1):** Selected Cisco Catalyst 3560 for `SW-DIST-01` to handle Inter-VLAN routing natively.
- **DEC-02 (Sprint 1):** Allocated `10.20.50.10` for `SRV-SYSLOG` and `10.20.50.20` for `SRV-NTP` on a dedicated infrastructure services segment (`10.20.50.0/24`).

### B. Recorded Simulator Limitations & Technical Workarounds
- **LIM-01 (Packet Tracer Timezone Syntax):** IOS `clock timezone UTC 0 0` threw syntax errors in Packet Tracer 8.x; standardized on `clock timezone UTC 0` across all baseline templates.
- **LIM-02 (Layer 3 SVI Down/Down State):** SVI interfaces on `SW-DIST-01` stay administratively down until Layer 2 VLAN definitions (`vlan <id>`) are created in the switch VLAN database.
- **LIM-03 (Lack of Native SIEM/IDS):** Cisco Packet Tracer cannot host external security platforms; all correlation will be conducted manually via raw Syslog analysis and structured Excel workbooks.

---

## 4. Peer Review & Validation Sign-Off

- **Author:** 
- **Peer Reviewer:** 
- **Review Date:** 
- **Review Outcome:**
