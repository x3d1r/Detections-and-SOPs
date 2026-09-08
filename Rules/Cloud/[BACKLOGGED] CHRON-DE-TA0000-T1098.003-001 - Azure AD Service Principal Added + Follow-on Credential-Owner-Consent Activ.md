# \[BACKLOGGED\] CHRON\-DE\-TA0000\-T1098\.003\-001 \| Azure AD Service Principal Added \+ Follow\-on Credential/Owner/Consent Activity \(48h Correlation\)

### **1. METADATA**

- **Analytic Title:** Azure AD Service Principal Added + Follow-on Credential/Owner/Consent Activity (48h Correlation)
- **ID:** CHRON-DE-TA0000-T1098.003-001\]
- **Version:** 1.0
- **Status:** \[ \] Experimental \[x\] Test \[ \] Production
- **Author:** Lucian Peckham
- **Date Created:** 
- **Date Modified:** 

### **2. DETECTION OVERVIEW**

**Description:**

This analytic detects an **Azure AD service principal being added** and correlates it with related **high-impact application/service principal actions** such as **adding credentials**, **certificates/secrets management**, **adding an owner**, or **consent to application** within a **48-hour window**. The goal is to surface persistence-style behavior aligned to the engagement artifacts where service principal/application objects were modified for continued access. 

**Hypothesis:**

We hypothesize that an adversary performing **cloud persistence via service principal/application manipulation** will execute the operation chain:

**Add service principal → (Add service principal credentials OR certificates/secrets management OR add owner OR consent to application)**

This activity can be observed through **Azure AD Directory Audit** events where the same **target object ID** (`product_object_id`) receives these event types within a bounded time window (48h), which is higher-risk behavior outside approved identity/app management processes.

**MITRE ATT&CK® Mapping:**

- **Tactic(s):** Persistence (TA0003)
- **Technique(s):** Account Manipulation (T1098)
- **Sub-technique(s):** Additional Cloud Credentials (T1098.003)

### **3. ANALYTIC DETAILS (Summiting the Pyramid)**

**Analytic Robustness Level (Select One & Justify):**

*Justification:* \[Explain why this level was chosen. Is the detection based on an invariant behavior, a specific tool artifact, or something *in between?\]*

* [ ] Level 1: Ephemeral
* [ ] Level 2: Core to Adversary-Brought Tool
* [ ] Level 3: Core to Pre-Existing Tool
* [x] Level 4: Core to Some Implementations of a (Sub-)Technique
* [ ] Level 5: Core to a (Sub-)Technique (Invariant Behavior)

**Data Source & Event Robustness:**

- **Platform:** Azure
- **Source:** Azure AD Directory Audit
- **Event(s):** Add service principal” + correlated audit event types (credential/secret/owner/consent)
- **Event Robustness Column (Select One):**
    - **Host-Based:** \[ \] Application (A) \[X\] User-Mode (U) \[ \] Kernel-Mode (K)

  **(A)** (SaaS/control-plane audit log source)
    - **Network-Based:** \[ \] Protocol Payload (P) \[ \] Protocol Header (H)

**Final Summiting Score:** \[e.g., 4K, 3U\]

### **4. LOGIC & IMPLEMENTATION**

**Detection Logic (Pseudocode or SIEM Query):**

-- Provide the query here.  
-- Comment heavily to explain each logical step of the query.

```
rule soc_AzureAD_Service_Principal_Added_Followed_by_Consent {

  meta:
    author = "Lucian Peckham"
    description = "Detects Azure AD service principal being added and correlates follow-on credential/secret/owner/consent activity on the same object_id within 48h; built from engagement artifacts and paired with an approved-user baseline playbook."
    severity = "High"

  events:
    // Anchor event: Service principal added (Azure AD Directory Audit)
    $app.metadata.product_name = "Azure AD Directory Audit"
    $app.metadata.vendor_name = "Microsoft"
    $app.metadata.product_event_type = "Add service principal"
    $app.security_result.action = "ALLOW"

    // Correlation key: target object id
    $app.target.resource.product_object_id = $object_id

    // Correlated events: additional actions on SAME object id
    $owner.metadata.product_name = "Azure AD Directory Audit"
    $owner.metadata.vendor_name = "Microsoft"
    $owner.target.resource.product_object_id = $object_id

    // Match follow-on activity types (case-insensitive)
    (
        re.regex($owner.metadata.product_event_type, "Add Service principal credentials") nocase
        OR re.regex($owner.metadata.product_event_type, "Certificates and secrets management") nocase
        OR re.regex($owner.metadata.product_event_type, "Add owner to application") nocase
        OR re.regex($owner.metadata.product_event_type, "Consent to application") nocase
    )

  match:
    // Correlate by object_id over 48 hours
    $object_id over 48h

  outcome:
    $risk_score = 35
    $event_count = count_distinct($app.metadata.id)
    $principal_county_region = array_distinct($app.principal.ip_geo_artifact.location.country_or_region)
    $target_application = array_distinct($app.target.resource.name)
    $target_application_id = array_distinct($object_id)
    $owner_event_type = array_distinct($owner.metadata.product_event_type)
    $app_event_type = array_distinct($app.metadata.product_event_type)

  condition:
    $app and $owner
}
```

**Known False Positives:**

- Normal enterprise app onboarding or service principal lifecycle events.
- Approved identity engineering or automation accounts managing secrets/certs.

**Exclusion Strategy:**

- Exclude by **approved actor identity baseline** (your stated design: playbook checks approved users/identities) rather than weak attributes like display names or generic “admin” patterns.
- Exclude known approved automation identities by immutable identifiers (e.g., object ID / app ID) where possible.

### **5. VALIDATION & RESPONSE (Funnel of Fidelity)**

**Testing Procedures:**

Engagement activity will be utilised to prove hypothesis. 


- **Test** Case **1 (Functional Synonym):**
    - **Tool/Procedure:** \[e.g., Execute Tool A\]
    - **Result:** \[ \] Detected \[ \] Not Detected
- **Test Case 2 (Procedural Synonym):**
    - **Tool/Procedure:** \[e.g., Execute Tool B, which uses different API calls for the same operations\]
    - **Result:** \[ \] Detected \[ \] Not Detected
- **Test** Case **3 (Sub-Technical Synonym):**
    - **Tool/Procedure:** \[e.g., Execute Tool C, which uses a different procedure for the same sub-technique\]
    - **Result:** \[ \] Detected \[ \] Not Detected

**Triage & Investigation Steps (Manual Playbook):**

\[Provide a clear, step-by-step checklist for the responding analyst.\]

1. **Initial Triage (Is this a true positive?):**
    - **Initial Triage (first 60 seconds):**
        1. Confirm which follow-on event types fired (`$owner_event_type`) and the target (`$target_application`, `$target_application_id`).
        2. Validate actor against the **approved-user baseline (list will  be updated)** (if not approved → treat as suspicious).
        3. Review geo context (`$principal_county_region`) for obvious outliers.
2. **Investigation (What is the context?):**
    - Pull the full audit trail for the same `product_object_id` across the 48h window.
    - Enumerate service principal/application credentials via PowerShell/Graph methods if portal visibility is incomplete. 
    - Verify new owners against expected app governance.
    - If consent activity occurred, determine whether it succeeded and what permissions were requested.

**Response & Remediation:**

If confirmed malicious:

- Remove newly added credentials and revert unauthorized ownership changes. 
- Contain by disabling the service principal/app where appropriate.
- Preserve evidence: export relevant audit logs and object configuration state.

*(Engagement reporting notes include that credentials added via Microsoft Graph may not show in portal credential view and recommends enumerating/removing with cmdlets such as Get-AzADSpCredential / Remove-AzADSpCredential.)*

### **6. AUTOMATION & RESPONSE PLAYBOOK (SOAR)**

This section provides a structured playbook for automated systems to perform enrichment, containment, and notification actions.

### Alert Trigger Condition & Severity

**Trigger:** Rule condition met (`$app and $owner`)  
**Default Severity:** High (meta.severity)

### Enrichment Steps

1. **Action:** Get-UserDetails

- **Input:** Actor identity from alert event context
- **Output:** display name, UPN, department/org, role membership, privileged group membership 

1. **Action:** Baseline-Approved-User-Check

- **Input:** Actor identity
- **Output:** approved/not-approved + baseline match record (if any) *(matches your rule description intent)*

1. **Action:** Get-ServicePrincipal/App-Details

- **Input:** `$target_application_id`
- **Output:** owners, credential metadata, app/service principal properties

1. **Action:** Get-Audit-Context-Window

- **Input:** `$target_application_id` + 48h window
- **Output:** related audit events and counts for the same object

### Triage Logic (Automated)

- IF `approved_identity == false` AND (`owner_event_type` contains credential/secrets management) → THEN escalate priority / route to Cloud IR
- IF `approved_identity == true` → THEN continue with standard change validation workflow
- IF geo outlier detected → THEN flag “geo anomaly” for analyst review

### Containment Steps

- Disable service principal / revoke sessions / remove credential / remove owner  
**Execute:** false (manual approval required)

### Notification Steps

- Create ticket and attach enrichment outputs + correlated event types + target app/object id.  
(Structure aligns to the existing playbook pattern documented for Azure AD detections.)

### **7. FRAMEWORK MAPPINGS**

**MITRE Engage™ Mapping:**

- **Goal:** \[e.g., Disrupt (G0009)\]
- **Approach:** \[e.g., Detect (A0001)\]

**D3FEND Mapping:**

- **Tactic:** **DET0531** 
- **Technique:** \[e.g., Process Spawn Analysis (D3-PSA)\\
