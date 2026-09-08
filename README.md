# detection-rules

> Enterprise-validated detection content for **Google SecOps (YARA-L)**,
> **Microsoft Defender for Endpoint (KQL)**, and **Azure AD / Entra ID** —
> engineered through a documented methodology and proven against real
> adversary activity in multi-billion-dollar environments.

![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%2CK-red)
![Platforms](https://img.shields.io/badge/Platforms-Chronicle%20%7C%20MDE%20%7C%20AzureAD-blue)
![Validation](https://img.shields.io/badge/Validated-Red%20Team%20Engagements-orange)
![License](https://img.shields.io/badge/License-MIT-green)

## Why this exists

Most public detection repos are rule dumps. This one is an **engineering
discipline made visible**. Every rule here was written, scored for robustness,
tested against live adversary tradecraft, and tuned until false positives
dropped to sustainable levels — then documented so another analyst (or an AI
agent) can reproduce that reasoning.

I built and tested these detections in real-world scenarios across enterprise
engagements, and measured the outcome: **a [X]% reduction in false-positive
volume** and **a [Y]% increase in true-positive fidelity** on the techniques
covered, compared to baseline vendor content.

## What makes this different

| Dimension | Typical repo | This repo |
|---|---|---|
| Provenance | Lab / Atomic Red Team only | **Live red-team engagements** in enterprise environments |
| Robustness | Unstated | **Scored** via *Summiting the Pyramid* (e.g., `4A`) |
| Validation | "Should fire" | **Funnel of Fidelity** — functional, procedural, sub-technical synonyms |
| Tuning | None | **Context-rich exclusion strategy** documented per rule |
| Mapping | ATT&CK only | **ATT&CK + MITRE Engage + D3FEND** |
| Automation | None | **SOAR enrichment + containment playbook** per rule |

## Measured impact

- **[X]%** false-positive reduction on covered techniques after exclusion tuning
- **[Y]%** true-positive fidelity uplift vs. out-of-box vendor rules
- **[N]** detections validated against observed adversary activity in engagements
- **[M]** rules promoted from `Test` → `Production` status
- Validated across **3 platforms**: Chronicle, Defender for Endpoint, Azure AD

> Replace `[X] [Y] [N] [M]` with your real figures. If exact numbers aren't
> tracked yet, use conservative ranges (e.g., "30–40%").

## Structure

Rules are grouped by **detection engine** (matching the ID prefix), with each
rule stored as a self-contained markdown knowledge base entry:

```
detection-rules/
├── README.md
├── METHODOLOGY.md
├── CHRON/                     # Google SecOps / Chronicle (YARA-L)
│   ├── TA0002/  TA0003/  TA0004/  TA0009/
├── MDE/                       # Microsoft Defender for Endpoint (KQL)
│   ├── TA0002/  TA0005/  TA0006/  TA0011/
├── CLOUD/                     # Azure AD / Entra ID (YARA-L or KQL)
│   └── ...
└── SOPs/                      # Detection + automation lifecycle SOPs
    ├── Detection-Lifecycle.md
    ├── Threat-Detection-Workflow.md
    └── Automation-Development-Lifecycle.md
```

## Naming convention

```
{ENGINE}-DE-{TACTIC}-{TECHNIQUE}-{SEQUENCE}
```

| Token | Meaning | Example |
|---|---|---|
| `ENGINE` | Query language / platform | `CHRON`, `MDE` |
| `DE` | Detection Engineering | constant |
| `TACTIC` | MITRE tactic ID | `TA0003` (Persistence) |
| `TECHNIQUE` | MITRE technique.subtechnique | `T1098.001` |
| `SEQUENCE` | Iteration for same technique | `001` |

**Example:** `CHRON-DE-TA0003-T1098.001-001` = first Chronicle rule for
Account Manipulation → Additional Cloud Credentials (Persistence).

> The **prefix encodes the engine**; the **folder encodes the operational
> domain**. A YARA-L rule targeting Azure AD audit logs lives in `CLOUD/`
> but keeps a `CHRON-` prefix because it runs in Chronicle.

## Coverage

| Engine | Domain | Techniques covered |
|---|---|---|
| CHRON (YARA-L) | Windows / Identity | T1110.001/.003, T1053.005, T1553.004, T1135, T1098.001 |
| MDE (KQL) | Endpoint | T1059.001, T1547.001, T1197.001, T1003.001, T1071.001 |
| CLOUD | Azure AD / Entra ID | T1098.003, consent abuse, service-principal persistence |

## Anatomy of every rule

Each rule is a 7-section knowledge base entry (see [`METHODOLOGY.md`](METHODOLOGY.md)):

1. **Metadata** — ID, version, status, author
2. **Detection Overview** — description, hypothesis, ATT&CK mapping
3. **Analytic Details** — *Summiting the Pyramid* robustness score
4. **Logic & Implementation** — commented query + exclusion strategy
5. **Validation & Response** — *Funnel of Fidelity* test results + triage playbook
6. **Automation & Response** — SOAR enrichment / containment / notification
7. **Framework Mappings** — MITRE Engage + D3FEND

## OPSEC / sanitization note

All rules are re-created from sanitized engagement work. Customer names,
tenant identifiers, internal hostnames, and proprietary correlation logic have
been removed. Field names use standard **UDM** (Chronicle) and **Defender
schema** conventions only. No client or engagement is identifiable.

## Using these rules

- **Import:** YARA-L rules into Chronicle; KQL into Defender hunting / custom detections.
- **Tune:** Apply the documented exclusion strategy before enabling alerting.
- **Validate:** Re-run the Funnel of Fidelity test cases in your environment.

## Roadmap

- [ ] Expand YARA-L ⇄ KQL parity where logic overlaps
- [ ] Add Sentinel (ASIM) translations
- [ ] Publish companion Sigma rules for portability
- [ ] Wire rules into an agentic-triage RAG corpus

## Author

Lucian Peckham — Threat Detection & Automation Engineer

## License

MIT — see [LICENSE](LICENSE). Attribution appreciated.
