# SOP \- Threat Detection Workflow

**Purpose: **The purpose of this is to define and understand the workflow for Detection Operations, Highlighting sources, evaluation of requests, development of Pseudocode, Version Control, approval and peer review, Implementation to production and the process of continuous improvement

## **Detection Ingest Workflow Visualisation:**

## **Proposed Kanban Workflow:**


---

## 1. Information Ingest

**Sources include:**

- Detection Kanban
- Projects
- Internal Review
- Incident Findings
- Engineering
- CSIRT
- SIEM platform
- GTS
- Architecture

---

## 2. Initial Request Evaluation

**Action**

Evaluate each incoming request against the following criteria:

- **Relevance**: Does the request align with current threat models?
- **Fidelity**: Is the alert likely to produce actionable intelligence?
- **Scalability**: Can the detection be implemented across systems?

**Outcome**

- If the request is **worthwhile**:
    - Approve and move to **Pseudocode Development**.
- If the request is **not worthwhile**:
    - Close the request.

these actions should be peer reviewed with the detection engineering team before any rejection or accepting the proposed request. 

---

## 3. Pseudocode Development

**Actions**

- Develop pseudocode for the detection logic, ensuring alignment with:
    - Current threat models
    - Alert fidelity standards

**Collaboration**

- By utilising the Detection Kanban, we can effectively:
    - Track work in progress
    - Surface blockers
    - facilitate channels for communication between teams

---

## 4. Development Version Creation

**Actions**

- Translate pseudocode into a functional detection rule or script.
- Test the detection in a **development environment** to ensure it:
    - Executes as intended
    - Produces expected alerts under test conditions
- Share progress and test results with relevant stakeholders.

---

## 5. Peer Review

**Actions**

Before implementation to Production, we need to have process of approval by peers focusing on key factors such as:

- **Logic accuracy**: Does the rule do what it is intended to do?
- **Alignment with security objectives**: Does it support current security priorities?
- **Quality of outcome**:
    - Potential for false positives, what strategy is put in place to reduce false positives
    - Potential for false negatives **(Funnel of Fidelity)**

Document the process of the review with constructive feedback or solutions and come to resolution.

---

## 6. Promotion to Production

**Actions**

Once approved via peer review:

- Promote the detection rule to the defined production systems, where applicable:
    - SIEM platform
    - Google SecOps
    - Source systems (e.g. WAF, Firewall, EDR tooling)

**Monitoring**

- Monitor the performance of the detection rule, including:
    - Alert volumes over time
    - Context quality provided for responders
- Collect metrics on:
    - **Alert fidelity**
    - **Response times** (e.g. time to acknowledge, time to action)

---

## 7. Continuous Improvement

**Actions**

- Conduct regular reviews of existing detection rules to ensure they remain:
    - Effective
    - Relevant to current threats
    - Efficient in terms of noise vs. value
- Use feedback from:
    - Incident findings
    - Post-incident reviews
    - Operator feedback  
to refine both:
    - The individual detections
    - This SOP and related workflows

---

## 8. Proving Success

**Key Metrics**

- **Turnaround time **
    - Average time from request submission to completion (production deployment).
- **Alert metrics**
    - Performance of representative alerts, gathering data from SecOps and focusing on:
        - Fidelity (true positive vs false positive rate)
        - Time to action (TTA) 
- **Collaboration metrics**
    - Number and quality of cross-team interactions, including:
        - Hand-offs between kanban items (Engineering, CSIRT, Architecture, etc.)
        - Feedback loops that resulted in rule updates or process changes
