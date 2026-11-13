# SOP \- Threat Detection – End‑to‑End Detection Lifecycle

This SOP defines a clear, end-to-end threat detection lifecycle, from signal creation through alerting, analyst action, automation, and effectiveness validation. It explains how detections move from initial hypothesis through validation, deployment, and continuous improvement, and how each stage ensures measurable risk reduction for the organisation. 

**Document Control** 

- Document Owner: Detection Engineering 
- Process Owner: Detection Engineering  
- Approvers: Detection Engineering, Other Stakeholder
- Version: 1.1   |  Last Reviewed: 07/09/2026   |   Next Review: DD/MM/YYYY 
- Review Frequency: Quarterly 

**Roles and Responsibilities **

Detection Engineering Analyst — detection development, testing, documentation, peer-review participation. 

Detection Engineer — rule QA, detection design review, production deployment approval.

Detection Engineering Lead — prioritisation, resource allocation, final approval. 

SOC Analyst — detection feedback, false-positive reporting, missed-detection identification. 

**Workflow Gates** 

Each stage has entry and exit criteria and a named owner. A stage may not begin until its entry criteria are met, and may not close until its exit criteria are satisfied. Each stage maps to an Azure DevOps (ADO) card state. 

|  |  |  |  |
| --- | --- | --- | --- |
| **Stage**  | **Entry Criteria**  | **Exit Criteria**  | **Owner**  |
| Intake  | Request received via Kanban  | Request categorised & prioritised  | Detection Engineering |
| Analysis  | Request assigned  | Design & ATT&CK mapping complete  | Detection Engineering |
| Development  | Design approved  | Logic authored in source control  | Detection Engineering |
| Testing  | Logic complete  | Functional + negative + retrohunt passed  | Detection Engineering |
| Peer Review  | Testing passed  | Review approved (not own work)  | Reviewer / Detection Engineering  |
| Enablement  | Approval granted  | Detection live in production  | Detection Engineering  |
| Monitoring  | Detection live  | 30-day validation complete  | Detection Engineering / Analyst  |

A high‑level overview of this lifecycle should accompany the flowchart, explaining how detections move from initial hypothesis through validation, deployment, and continuous improvement, and how each stage ensures measurable risk reduction for the organisation.

---

## Pre‑requisites

When creating detections, the following references and templates must be understood and applied:

- Detection engineering methodology and workflow:    
A Step-by-Step Guide to the Detection Engineering Template (private Confluence link removed for public release)
- Standard detection template and source of truth:    
Detection Template | Source: Def Con Group (private Confluence link removed for public release)

By utilising these guides and detection templates, we ensure:

- A clear understanding of **why** the detection is being created
- Confidence that the detection will be **effective and scalable**
- Consistent application of detection engineering standards

These artefacts help us:

- Assess the **fidelity** of the alert
- Grade the detection with reference to the **triangle of pain**
- Evaluate and document what makes a **high‑quality detection**

---

## Detection Process – Step‑by‑Step

### 1. Threat Use Case Definition

Define the detection in terms of **attacker behaviour**, not tools, products, or log sources.

- Behaviour‑based focus:
    - The detection should describe what the attacker is doing, not just which tool or event is observed.
- Example behaviours:
    - Credential abuse
    - LSASS memory dumping
    - Phishing‑based initial access

**Output:**    
Documented **Threat Detection Hypothesis**

---

### 2. Telemetry and Signal Validation

Confirm that the required telemetry exists and is suitable for detection.

- Validate log coverage and quality:
    - Required logs exist for all relevant systems
    - Logs are reliable, complete, and timely
    - The behaviour can be **consistently observed** in the data
- Key question:    
“If this behaviour occurs, will we effectively see it?”

This step ensures the detection is technically feasible **before** any logic is authored.

**Output:**    
Confirmed **telemetry and signal sources which feeds into the Detection Template**

**Analysis Checklist: ** 

- Existing detections, playbooks and SOAR automations reviewed 

- MITRE ATT&CK techniques identified; data-source availability confirmed 

- Historical incident review completed; detection success criteria documented

**Required Deliverables: **detection design notes, logic pseudocode, data-source validation, ATT&CK mapping. 

---

### 3. Detection Authoring (Using Detection Template)

All detections **must be documented** using the **Detection Template | Source: Def Con Group**.

At minimum, the following sections must be completed:

#### Required Sections

- **Metadata**
- **Detection Overview and Hypothesis**
- **MITRE ATT&CK Mapping**
- **Analytic Robustness Level (Summiting the Pyramid)**
- **Data Source and Event Robustness**
- **Detection Logic**
- **Known False Positives**
- **Exclusion Strategy**
- **Validation and Testing**
- **SOC Triage Steps**
- **Automation and Response (the engagement)**

This ensures every detection answers:

- *What behaviour is this detecting?*
- *Why is it hard for attackers to evade?*
- *What should SOC do when it fires?*

**Output:**    
Completed **detection write‑up**

---

### 4. False Positive Evaluation (Mandatory)

False positive evaluation is a **mandatory gate** in the lifecycle.

This includes:

- Identifying legitimate administrative or business activity that may match the pattern
- Identifying automation or tooling that may generate benign matches
- Estimating the **SOC impact** (volume, complexity, and investigation effort)

False positives are **not a failure**, but they **must be understood, documented, and accepted**.

**Output:**    
Documented **false positive assessment**

---

### 5. Stakeholder Review and Approval

Detections must be reviewed and approved by the following key stakeholders:

- Lucian Peckham
  
Stakeholders are asked to **explicitly confirm** that:

- The behaviour represents a **meaningful risk**
- The false positive trade‑off is **acceptable**
- The detection should be **enabled in production**

Approval is explicitly requested; silence or non‑response is **not** considered approval.

**Output:**    
Recorded **stakeholder agreement**

**Peer Review Evidence: ** 

Reviewer must confirm: logic accuracy, ATT&CK-mapping accuracy, data-source suitability, testing completed, documentation updated. 

Reviewer must not review their own work. Review outcome recorded in ADO

---

### 6. Detection Enablement

Once approved:

- The detection is enabled in the appropriate **production environment(s)**
- SOC is briefed on:
    - Detection intent and expected behaviour
    - Required triage steps
    - Any known limitations or caveats
- An initial **monitoring period** begins to observe behaviour and impact

**Output:**    
**Live detection** in production

**Deployment requires: **peer-review approval, testing completion, documentation completion, change-record reference. 

**Rollback Plan — all deployments must include: **previous-version reference, rollback procedure, owner responsible for rollback. 

**Post-Deployment Validation: **within the first 30 days, review alert volume, false positives, and missed detections; collect SOC analyst feedback. Findings documented in ADO. 

---

### 7. SOC Alert Handling

When the detection generates an alert:

- Analysts follow the **documented triage steps**
- Context and enrichment data are reviewed
- A clear **disposition** is recorded for each alert

SOC notes must explicitly state:

- Why the alert was assessed as **benign** or **malicious**
- What **evidence** supported the decision

**Output:**    
Analyst **disposition and investigation notes**

---

### 8. Automation

Where automation is defined:

- Enrichment, containment, and/or notification actions are executed
- All automated actions are **logged and auditable**

During operation, analysts should provide feedback on whether:

- Investigation effort was **reduced**
- Automation provided **meaningful context**
- Automation **behaved as expected** (no unintended impact)

**Output:**    
Recorded **automation outcome** and feedback

---

### 9. Effectiveness Validation

Detection effectiveness is periodically reviewed with respect to:

- **Alert accuracy** (precision and usefulness of fired alerts)
- **SOC feedback** on workload, clarity, and value
- **Automation impact** (time saved, quality of response)
- Ongoing alignment to the **original detection hypothesis**

Key question:

> “Did this detection actually improve our ability to detect or respond?”

We need to effectively measure the value provided from said alerts, gather analytics from the whole process 

**Output:**    
Documented **effectiveness assessment**

---

### 10. Continuous Improvement

Based on operational outcomes and effectiveness reviews, detections may be:

- **Tuned** (logic adjustments, thresholds, exclusions)
- **Expanded** (coverage extended to additional behaviours or data sources)
- **Downgraded** (severity or visibility reduced)
- **Disabled** (retired if no longer valuable or appropriate)

Detections that do not demonstrably contribute to risk reduction **must not** remain enabled.

**Output:**    
Documented **detection lifecycle decision**

---

**Glossary** 

- ADO — Azure DevOps 

- ATT&CK — MITRE ATT&CK Framework 

- Detection Engineering — the detection engineering team 

- SOC — Security Operations Centre 

- SOAR — Security Orchestration, Automation and Response 

- UDM — Unified Data Model (Google SecOps) 

- YARA-L — detection rule language used in Google SecOps 

- Retrohunt — historical search against previously collected telemetry 

- SIEM platform — detection/response platform 

- CSIRT — Computer Security Incident Response Team 


Related items:

Responsible manager for process:

Static or printed copies of this may not be the current version.  See version history below and consult Confluence for current version. 

Copyright © 2024. All rights reserved.
