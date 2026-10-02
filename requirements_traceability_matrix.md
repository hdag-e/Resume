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
| | **SCRUM-74** | Define the enterprise network addressing plan | Appx D & E | `Documentations/network_addressing_plan.md` | In Progress |
| | **SCRUM-75** | Define device roles and network segmentation | Appx D & E | `Documentations/network_segmentation_and_roles.md` | In Progress |
| | **SCRUM-76** | Document baseline network architecture | Appx D & E | `Documentations/baseline_network_architecture.md` | In Progress |
| | **SCRUM-77** | Define baseline device configuration requirements | Appx B | `Documentations/baseline_config_requirements.md` | In Progress |
| **SCRUM-24** | | **Finalize project requirements & acceptance criteria** | Appx A | N/A (Parent Story) | In Progress |
| | **SCRUM-78** | Review project scope and required deliverables | Appx A | `Documentations/requirements_traceability_matrix.md` | In Progress |
| | **SCRUM-79** | Define measurable project acceptance criteria | Charter | `Documentations/requirements_traceability_matrix.md` | In Progress |
| | **SCRUM-80** | Establish requirements traceability and review process | Charter / RTM | `Documentations/requirements_traceability_matrix.md` | In Progress |
| **SCRUM-25** | | **Set up shared repository and project structure** | Infra Setup | N/A (Parent Story) | Complete |
| | **SCRUM-81** | Initialize shared project repository | Infra Setup | Root (`README.md`, `CONTRIBUTING.md`) | Complete |
| | **SCRUM-82** | Establish repository folder structure and naming conventions | QA Standards | Repository Directory Tree | Complete |
| | **SCRUM-83** | Establish contribution and version-control workflow | QA Standards | `CONTRIBUTING.md` | Complete |
| **SCRUM-26** | | **Build and verify Packet Tracer topology** | Appx D | N/A (Parent Story) | In Progress |
| | **SCRUM-84** | Build the enterprise network topology in Packet Tracer | Appx D | `Configurations/CYBEROPS07_Topology_v1.0.pkt` | In Progress |
| | **SCRUM-85** | Configure baseline device interfaces and addressing | Appx D & E | `Configurations/CYBEROPS07_Topology_v1.0.pkt` | In Progress |
| | **SCRUM-86** | Configure baseline routing and switching behavior | Appx D & E | `Configurations/CYBEROPS07_Topology_v1.0.pkt` | In Progress |
| | **SCRUM-87** | Verify topology configuration against the approved design | Appx D | `Evidence and Testing/topology_verification.png` | In Progress |
| **SCRUM-27** | | **Verify baseline network connectivity** | Test T-01 | N/A (Parent Story) | In Progress |
| | **SCRUM-88** | Define baseline connectivity test matrix | Test T-01 | `Documentations/connectivity_test_matrix.md` | In Progress |
| | **SCRUM-89** | Execute baseline connectivity tests | Test T-01 | `Evidence and Testing/test_t01_syslog_verification.png` | In Progress |
| | **SCRUM-90** | Document and resolve baseline connectivity issues | Test T-01 | `Documentations/troubleshooting_log.md` | In Progress |
| **SCRUM-28** | | **Establish project evidence and documentation structure** | Quality Assurance | N/A (Parent Story) | In Progress |
| | **SCRUM-91** | Establish project evidence repository structure | QA Standards | `/Evidence and Testing/` Folder | In Progress |
| | **SCRUM-92** | Define project documentation standards and templates | QA Standards | `CONTRIBUTING.md` | In Progress |
| | **SCRUM-93** | Establish project decision, risk, and review records | QA Standards | Section 3 of RTM File | In Progress |

---

## 3. Decision, Change, & Known Limitation Record

### A. Architecture & Scope Decisions
- **DEC-01:** Selected Cisco Catalyst 3560 for `SW-DIST-01` to perform Layer 3 Inter-VLAN routing natively.
- **DEC-02:** Allocated `10.20.50.10` for `SRV-SYSLOG` and `10.20.50.20` for `SRV-NTP` on a dedicated infrastructure services segment (`10.20.50.0/24`).

### B. Recorded Simulator Limitations & Technical Workarounds
- **LIM-01 (Packet Tracer Syntax):** Standard IOS `clock timezone UTC 0 0` threw syntax errors in Packet Tracer 8.x; standardized on `clock timezone UTC 0` across all baseline templates.
- **LIM-02 (Layer 3 SVI Down/Down State):** SVI interfaces on `SW-DIST-01` stay administratively down until Layer 2 VLAN definitions (`vlan <id>`) are created in the switch VLAN database.

---

## 4. Peer Review & Validation Sign-Off

- **Author: Hasan Dagdelen (hd352)** 
- **Peer Reviewer:** 
- **Review Date:** 
- **Review Outcome:** 
