# Detection Engineering Methodology

This document defines the engineering standard every rule in this repository
must meet. It is the reason the content is trustworthy: detections here are
**scored, tested, and tuned**, not just written.

---

## 1. Naming Convention

```
{ENGINE}-DE-{TACTIC}-{TECHNIQUE}-{SEQUENCE}
```

- **ENGINE** — the query engine the rule runs in (`CHRON`, `MDE`, future `SENT`).
- **DE** — constant marker for Detection Engineering.
- **TACTIC** — MITRE ATT&CK tactic ID (`TA0003`).
- **TECHNIQUE** — MITRE technique + sub-technique (`T1098.001`).
- **SEQUENCE** — `001`, `002`… for multiple rules against the same technique.

The **prefix encodes the engine**; the **folder encodes the domain**. This lets
CI lint by language while analysts browse by operational scope.

---

## 2. Rule Template (7 Sections)

Every rule is a self-contained markdown entry with these sections, in order:

| # | Section | Purpose |
|---|---|---|
| 1 | Metadata | ID, version, status (`Experimental`/`Test`/`Production`), author |
| 2 | Detection Overview | What it detects, the hypothesis, ATT&CK mapping |
| 3 | Analytic Details | Robustness score via *Summiting the Pyramid* |
| 4 | Logic & Implementation | Commented query + exclusion strategy |
| 5 | Validation & Response | *Funnel of Fidelity* results + manual triage playbook |
| 6 | Automation & Response | SOAR enrichment, containment, notification |
| 7 | Framework Mappings | MITRE Engage + D3FEND |

A rule is not mergeable until sections 1–5 are complete. Section 6–7 are
required for `Production` status.

---

## 3. Summiting the Pyramid — Robustness Scoring

Every rule is scored on two axes. The goal is to prefer detections anchored to
behavior the adversary **cannot easily change**.

### 3a. Analytic Robustness Level

| Level | Definition | Adversary cost to evade |
|---|---|---|
| 1 | Ephemeral | Trivial |
| 2 | Core to adversary-brought tool | Low |
| 3 | Core to pre-existing tool | Moderate |
| 4 | Core to some implementations of a (sub-)technique | High |
| 5 | Core to a (sub-)technique (invariant behavior) | Very high |

### 3b. Event Robustness (telemetry source)

| Class | Codes |
|---|---|
| Host-Based | Application `A` · User-Mode `U` · Kernel-Mode `K` |
| Network-Based | Protocol Payload `P` · Protocol Header `H` |

### 3c. Final Score

Combine both: **`{RobustnessLevel}{EventCode}`**.

> Example: `4A` = Level 4 (core to some implementations of the sub-technique)
> observed via Application-level audit telemetry.

Rules at Level 1–2 must carry an explicit note on expected adversary evasion.
Level 4–5 rules are preferred for `Production`.

---

## 4. Funnel of Fidelity — Validation

A rule is only as good as its proof. Each rule is tested through three
progressively harder synonym classes:

| Test | Definition | Question answered |
|---|---|---|
| **1. Functional Synonym** | Same technique, same/similar tooling as observed | Does it catch what we actually saw? |
| **2. Procedural Synonym** | Same technique, different API calls / procedure | Does it survive a tool change? |
| **3. Sub-Technical Synonym** | Same sub-technique, different procedure entirely | Does it generalize? |

**Result recording:** each test is marked `Detected` / `Not Detected`. A rule
stays in `Test` until Test 1 passes against observed engagement activity; it
moves toward `Production` as Tests 2–3 pass.

> Where activity was directly observed in an engagement, Test 1 is anchored to
> that real tradecraft. Later re-tests are reissued as engagements recur.

---

## 5. Exclusion Strategy

We apply **context-rich exclusions, not weak indicators**. Exclusions target
known-legitimate context (approved onboarding, change-managed rotation,
platform-engineering ownership) rather than blunt filters that adversaries can
mirror.

Each rule documents:
- **Known false positives** (observed benign triggers)
- **Exclusion logic** (the precise fields/values excluded)
- **Candidate hardening** (future context features to add)

This is the primary driver of the measured false-positive reduction.

---

## 6. Framework Mappings

Every production rule maps to three frameworks so it serves detection,
deception, and defense engineering simultaneously:

- **MITRE ATT&CK** — tactic / technique / sub-technique
- **MITRE Engage** — goal + approach (e.g., Disrupt / Detect)
- **D3FEND** — tactic + technique (e.g., `D3-DET`)

---

## 7. Detection Lifecycle

Rules move through a governed lifecycle, documented in [`SOPs/Detection-Lifecycle.md`](SOPs/Detection-Lifecycle.md):

```
Experimental → Test → Production → Tuned / Deprecated
```

Promotion requires:
- `Experimental → Test`: logic complete + Test 1 (functional synonym) passes
- `Test → Production`: Tests 2–3 pass + SOAR playbook + framework mappings
- `Production → Tuned`: exclusion strategy applied, FP rate within threshold
