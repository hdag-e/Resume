# Requirements Traceability Matrix & Review Process (SCRUM-80)

## 1. Overview & Operating Rules
This document establishes end-to-end traceability across the CYBEROPS-07 project lifecycle. It maps every Jira Parent Story and Child Subtask to its core scope requirement, Git evidence location, and validation status.

### Operating Rules for Team Members:
1. **Scope Mapping:** Every Jira subtask must map to at least one core scope deliverable or test case.
2. **Evidence Linking:** Subtasks cannot be marked `Done` in Jira without linking relative file paths to logs, documentation, or `.pkt` models in the repository.
3. **Living Lifecycle Document:** Update this table whenever new subtasks move to `In Progress` or `Complete`.

---

## 2. Sprint 1 Requirements & Traceability Matrix

| Parent Story | Subtask ID | Task Description | Scope Mapping | Evidence / File Path in Repository | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SCRUM-23** | | **Establish enterprise network baseline** | Appx D & E | N/A (Parent Story) | In Progress |
| | **SCRUM-74** | Define the enterprise network addressing plan | Appx D & E | `docs/network_segmentation_and_roles.md` | In Progress |
| | **SCRUM-75** | Define device roles and network segmentation | Appx D & E | `docs/network_segmentation_and_roles.md` | In Progress |
| | **SCRUM-76** | Document baseline network architecture | Appx D & E | `docs/network_segmentation_and_roles.md` | In Progress |
| | **SCRUM-77** | Define baseline device configuration requirements | Appx B | `docs/baseline_config_requirements.md` | In Progress |
| **SCRUM-24** | | **Finalize project requirements & acceptance criteria** | Appx A | N/A (Parent Story) | In Progress |
| | **SCRUM-78** | Review project scope and required deliverables | Appx A | `docs/requirements_traceability_matrix.md` | In Progress |
| | **SCRUM-79** | Define measurable project acceptance criteria | Charter | `docs/requirements_traceability_matrix.md` | In Progress |
| | **SCRUM-80** | Establish requirements traceability and review process | Charter / RTM | `docs/requirements_traceability_matrix.md` | In Progress |
| **SCRUM-25** | | **Set up shared repository and project structure** | Infra Setup | N/A (Parent Story) | In Progress |
| | **SCRUM-81** | Initialize shared project repository | Infra Setup | Shared GitHub Repository | In Progress |
| | **SCRUM-82** | Establish repository folder structure and naming conventions | QA Standards | Repository Folder Tree (`/docs`, `/reports`) | In Progress |
| | **SCRUM-83** | Establish contribution and version-control workflow | QA Standards | `CONTRIBUTING.md` / Git Workflow | In Progress |
| **SCRUM-26** | | **Build and verify Packet Tracer topology** | Appx D | N/A (Parent Story) | In Progress |
| | **SCRUM-84** | Build the enterprise network topology in Packet Tracer | Appx D | `topology.pkt` | In Progress |
| | **SCRUM-85** | Configure baseline device interfaces and addressing | Appx D & E | `topology.pkt` | In Progress |
| | **SCRUM-86** | Configure baseline routing and switching behavior | Appx D & E | `topology.pkt` | In Progress |
| | **SCRUM-87** | Verify topology configuration against the approved design | Appx D | Verification Audit Logs | In Progress |
| **SCRUM-27** | | **Verify baseline network connectivity** | Test T-01 | N/A (Parent Story) | In Progress |
| | **SCRUM-88** | Define baseline connectivity test matrix | Test T-01 | `docs/connectivity_test_matrix.md` | In Progress |
| | **SCRUM-89** | Execute baseline connectivity tests | Test T-01 | Ping & ICMP Reachability Logs | In Progress |
| | **SCRUM-90** | Document and resolve baseline connectivity issues | Test T-01 | Troubleshooting & Bug Log | In Progress |
| **SCRUM-28** | | **Establish project evidence and documentation structure** | Quality Assurance | N/A (Parent Story) | In Progress |
| | **SCRUM-91** | Establish project evidence repository structure | QA Standards | Repository Directory Layout | In Progress |
| | **SCRUM-92** | Define project documentation standards and templates | QA Standards | `docs/templates/` | In Progress |
| | **SCRUM-93** | Establish project decision, risk, and review records | QA Standards | Section 3 of RTM File | In Progress |

---

## 3. Decision, Change, & Known Limitation Record

### A. Architecture & Scope Decisions
- **DEC-01:** Selected Cisco Catalyst 3560 for `SW-DIST-01` to perform Layer 3 Inter-VLAN routing natively[cite: 10, 11].
- **DEC-02:** Allocated `10.20.50.10` for `SRV-SYSLOG` and `10.20.50.20` for `SRV-NTP` on a dedicated infrastructure services segment (`10.20.50.0/24`)[cite: 10].

### B. Recorded Simulator Limitations & Technical Workarounds
- **LIM-01 (Packet Tracer Syntax):** Standard IOS `clock timezone UTC 0 0` threw syntax errors in Packet Tracer 8.x; standardized on `clock timezone UTC 0` across all baseline templates[cite: 10, 11].
- **LIM-02 (Layer 3 SVI Down/Down State):** SVI interfaces on `SW-DIST-01` stay administratively down until Layer 2 VLAN definitions (`vlan <id>`) are created in the switch VLAN database[cite: 10, 11].

---

## 4. Peer Review & Validation Sign-Off

- **Author: Hasan Dagdelen (hd352)** 
- **Peer Reviewer:** 
- **Review Date:** 
- **Review Outcome:** 
