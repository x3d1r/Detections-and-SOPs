# SOP \- Agentic Memory Review & Acceptance

# Agentic Memory Review & Acceptance

SIEM platform  —  Detection Engineering

|  |  |
| --- | --- |
| **Document title** | Standard Operating Procedure — Agentic Memory Review & Acceptance (SIEM Platform) |
| **Document owner** | Detection Engineering |
| **Author** | Lucian Peckham — Detection Engineer |
| **Applies to** | Detection engineering peer reviewers/approvers |
| **Platform** | SIEM Platform — Settings \> Content \> Agentic Memory |
| **Supporting register** | [Example] |
| **Version** | 1.0 (Draft) |
| **Date** | 26 August 2026 |
| **Review cycle** | Quarterly, or on SIEM platform release affecting AM behaviour |

*Internal Use Only*

  

# Contents


  

# 1. Purpose & Scope

This Standard Operating Procedure (SOP) defines how the detection engineering team reviews, accepts, edits, and denies Agentic Memories (AMs) in the SIEM platform. It exists to ensure AMs are consistent, deterministic, and safe — avoiding blind spots and reducing low-value “whack-a-mole” tuning.

The SOP applies to all analysts who review AMs: detection engineering peer reviewers and approvers, APAC and AMER SOC analysts, and SOC I staff performing Level 1 review. Every AM decision — approve, deny, or edit — must follow the criteria and workflow set out here.

### Where AMs live

Agentic Memories are managed in the SIEM platform under Settings \> Content \> Agentic Memory. Memories and Model Guidance are merged into this single system. Decisions are tracked in the Agentic Memory Register (Agentic Memory Register.xlsx) before being actioned in the SIEM platform.

# 2. Background & Guiding Principles

### Memories are secondary to rule tuning

Direct rule tuning (for users, processes, IPs) is the primary method for exclusions. Agentic memories are a secondary option, reserved for more complex decisions that require additional pivots or searches during an investigation. If a decision can be handled by a rule or automation, it should be — not by a memory.

### Filter on behaviours, not single indicators

Single-indicator or indicator-pair memories create a whack-a-mole of approvals that do not meaningfully reduce alerts and are taxing on both approvers and the AI. Prefer memories that describe behaviours acceptable for most users over ones that pin a single username/IP combination.

### Memories must be stateless and deterministic

A good AM states a fact or a guaranteed “When X happens, always do Y” outcome. It must not rely on time, baselining, or judgement. Anything requiring the AI to weigh context or make a judgement call belongs in Rules & Automation.

# 3. Agentic Memory Types

Every memory is applied using one of three methods. Choose the most precise method that satisfies the use case, and prefer Conditional over Global wherever possible.

|  |  |
| --- | --- |
| **Type** | **Behaviour & guidance** |
| **Global** | Applied to every IR investigation, every time. No criteria. Use only for broad, always-true context — can affect hundreds of tickets. Global can only be set when the memory is first added, not during editing. |
| **Conditional** | Activates only when user-defined criteria are met (e.g. a specific Rule ID). Most precise and preferred for targeted context. |
| **Adaptive** | Uses RAG to apply the memory when an investigation reaches a defined similarity threshold. Monitor for over-application to unrelated alerts. |

# 4. Acceptance Criteria

Use the following criteria to decide whether a memory should be approved, edited, or denied. These criteria are the core of this SOP and should be applied to every memory in the register.

## 4.1 When to APPROVE

A memory is a good candidate for approval when it describes a deterministic fact or a guaranteed outcome. Approve if the memory meets one or more of the following:

|  |  |
| --- | --- |
| **Approve if the memory…** | **What it means** |
| **Binary / always-true facts** | Deterministic statements with no ambiguity or interpretation required. |
| **Roles compared to identities** | Describes a role or account type rather than a single person (e.g. “\_cs accounts are contractor service / administrator accounts”). |
| **Vendor or platform behaviour** | Reflects legitimate, documented product or service activity. |
| **Documented architecture** | Matches how the environment is genuinely built and configured. |
| **Fits “When X happens, always do Y”** | Can be expressed as a clean, deterministic conditional with a guaranteed outcome. |

## 4.2 When to DENY or tune out

Deny a memory when it cannot be justified against the approval criteria on the basis of context, or trust in a specific identity. Deny (or revert for widening) if the memory exhibits any of the following:

|  |  |
| --- | --- |
| **Deny / tune out if the memory…** | **Rationale** |
| **Over-narrowing / named artifacts** | Named users, specific hostnames, explicit SIDs. IPs are only acceptable when tied to a specific server(s) or a formal IP range. Where possible, revert to the submitting CSIRT cohort to widen the request, or widen manually if intent is clear. |
| **Judgement-based wording** | Contains should, may, unless, if abnormal, if no other alerts, verify first, or confirm with owner. These signal a judgement call that belongs in Rules & Automation, not a memory. Absolutes like “always close for X person” / “never escalate for X person” are also denied. |
| **Temporal / stateful logic** | Time windows (e.g. “last 30 days”), baselining, first-seen logic, frequency analysis, or rarity/anomaly assessment. Agentic memory must be stateless and cannot make adjustment calls based on elapsed time. |
| **Guaranteed statements** | “This can never be malicious”, “this alert is always false”, “no investigation required under any circumstances” — these create blind spots and are denied. |

## 4.3 Wording red flags (quick check)

If any of these words or phrases appear in a memory, treat it as judgement-based and route it to Rules & Automation rather than approving as-is:

- should, may, unless, if abnormal, if no other alerts
- verify first, confirm with owner
- always close for \[person\], never escalate for \[person\]
- this can never be malicious, this alert is always false, no investigation required
- time-bound phrasing such as “last 7 / 30 days”, “first seen”, “rarely”, “anomalous”

## 4.4 Handling named artifacts

Memories that hard-code named users, hostnames, or SIDs are usually too narrow. Before denying, consider whether the intent can be widened to be globally useful:

1. **Revert to the submitter: **where possible, return the request to the CSIRT cohort who submitted it and ask them to widen the tuning request.
2. **Widen manually: **if the meaning or intent can be reliably derived, widen the memory yourself (e.g. role-based rather than user-based).
3. **Deny if neither is possible: **if the memory cannot be widened and cannot be justified, deny it.

# 5. Structural Requirements for Memories

Approved memories that direct an action must follow the SIEM platform’s current structural requirements:

### Conditional logic for close / escalate

If a memory directs a close or escalate action, it must use the conditional logic structure:

\[Prefix\]: If \[condition the agent must verify through investigation\], respond with action: \[Action\] closeCode: \[Code\].

### Exact match

Use the “EXACT MATCH ONLY” directive when the memory relies on matching specific artifacts, so the AI does not generalise (e.g. filtering all “ExecutionPolicy Bypass” commands when only one approved script path should be excluded).

### Prefix & tags

A prefix is not strictly required, but it adds weight and produces more deterministic behaviour. Tags help reduce ambiguity between memories and should be applied where they aid clarity.

### Retention

Memories can be set for up to 10 years. There is currently no expiry notification, so retention periods should be recorded and standard AM timings agreed and applied consistently. Note that Global application can only be set when the memory is first created.

# 6. Review & Approval Workflow

No memory is actioned in the SIEM platform without peer review and countersign. The register drives the workflow via the Status field.

## 6.1 Roles

- **First reviewer (Level 1 / SOC I): **performs the initial pass, records reasoning, and proposes an outcome.
- **Peer reviewer / approver (Detection Engineering): **evaluates the proposal, resolves questions, and countersigns the final approve/deny before actioning in the SIEM platform.

*The first person and the peer reviewer must be different people. A reviewer may hold the ability to approve or deny, but must still obtain peer review before actioning.*

## 6.2 Step-by-step

1. **Intake. **Memory is pulled into the Agentic Memory Register (manually or via the API automation). Default status is Unreviewed.
2. **First pass. **Reviewer assesses the memory against the Section 4 criteria and decides: propose approve, propose deny, or propose edit.
3. **Propose approve / deny. **Put reasoning in the Edited Memory column (H) and set Status to Propose Approve or Propose Deny.
4. **Propose an edit. **Add the revised wording to the Edited Memory column (H), add a cell comment (right-click \> New Comment) with your reasoning, and set Status to Edited.
5. **Peer review. **A second reviewer evaluates. If they have questions or amendments, they update the edit and @-mention the original reviewer in a comment reply.
6. **Countersign & action. **Once agreed, set Status to Approved or Denied and action the memory in the SIEM platform (approve, edit, or delete).
7. **Duplicates. **Deny duplicates that have already been peer-reviewed; run a dedupe pass of new memories against existing ones where feasible.

## 6.3 Register status values

The Status field in the register uses a fixed dropdown with conditional formatting. Values and their meaning:

|  |  |
| --- | --- |
| **Status** | **Meaning** |
| **Unreviewed** | Default state. Memory pulled into the register and awaiting first pass. |
| **Pending** | Awaiting further information or an external dependency before a decision can be made. |
| **Propose Approve** | First reviewer recommends approval. Reasoning recorded in the Edited Memory column (H). |
| **Propose Deny** | First reviewer recommends denial. Reasoning recorded in the Edited Memory column (H). |
| **Edited** | First reviewer proposes a wording change. Edit added to column H and a cell comment added with reasoning. |
| **Approved** | Peer-reviewed and countersigned. Actioned (approved) in the SIEM platform. |
| **Denied** | Peer-reviewed and countersigned. Actioned (denied/deleted) in the SIEM platform. |

# 7. Common Pitfalls & Watch-outs

- **AI hallucination in generated memories: **platform-generated memories can invent context not present in the original close note (e.g. mislabelling the purpose of \_cs accounts). Always validate against the source close note.
- **Adaptive over-application: **Adaptive memories can apply to alerts unrelated to the original. Consider switching to Conditional where scope is drifting.
- **Unintended Global scope: **Global memories apply to hundreds of tickets. Confirm scope was intended and that wording is genuinely global, not alert-specific.
- **Over-broad command-line filters: **Filtering on parameters (e.g. “ExecutionPolicy Bypass”) risks excluding all such commands. Filter on the approved script path instead, with EXACT MATCH ONLY.
- **Auditability gaps: **Deny actions do not currently capture time and user, and some AMs do not display in the list (searchable only). Record decisions in the register and raise UI bugs with the platform vendor.

# 8. References

- Agentic Memory Register.xlsx — working register for AM decisions.
- Platform Resource Center: Question Mark (top right) \> Resource Center \> search “Memory”.
- Private platform resource site (URL removed for public release) — agentic memory
- Private platform resource site (URL removed for public release) — agentic memory-best-practices
- SIEM platform May 2026 Product Release notes (Agentic Memories update).

*End of document.*
