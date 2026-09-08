# CHRON\-DE\-TA0002\-T1110\.001\-T1110\.003\-001 \| Detection of Successful Logins After Brute Force Failures

### **1. METADATA**

**Analytic Title: Detection of Successful Logins After Brute Force Failures**  
**ID: DE-TA0002-T1110.001-T1110.003-001**  
**Version: 1.0**  
**Status: \[\] Experimental \[X\] Test \[ \] Production**  
**Author: Lucian Peckham**  
**Date Created: **  
**Date Modified: **  

### **2. DETECTION OVERVIEW**

**Description:**  
This detection identifies successful logins following a series of failed login attempts within a specific timeframe. The detection excludes trusted networks and whitelisted accounts to reduce false positives. This behaviour may indicate a successful brute force attack or credential stuffing attempt.

**Hypothesis:**  
We hypothesise that an adversary attempting to gain unauthorised access to an account (MITRE Techniques T1110.001 - Password Guessing and T1110.003 - Credential Stuffing) will generate multiple failed login attempts followed by a successful login. This detection correlates these events to identify suspicious activity.

**MITRE ATT&CK® Mapping:**

- **Tactic(s):** Credential Access (TA0006)
- **Technique(s):** Brute Force (T1110)
- **Sub-technique(s):** Password Guessing (T1110.001), Credential Stuffing (T1110.003)

### **3. ANALYTIC DETAILS (Summiting the Pyramid)**

**Analytic Robustness Level (Select One & Justify):**

**Justification: This analytic detects behaviour indicative of brute force attacks or credential stuffing attempts, which are common techniques used by adversaries. It correlates failed and successful login events within a specific timeframe to identify potential compromise.**

- **Level 1: Ephemeral**
- **Level 2: Core to Adversary-Brought Tool**
- **Level 3: Core to Pre-Existing Tool**
- **Level 4: Core to Some Implementations of a (Sub-)Technique**
- **Level 5: Core to a (Sub-)Technique (Invariant Behaviour)**

**Data Source & Event Robustness:**

- **Platform: Office 365**
- **Source: Office 365 Security Logs**
- **Event(s): Failed Login Attempts, Successful Login Attempts**

**Event Robustness Column (Select One):**

- **Host-Based: \[ \] Application (A) \[ \] User-Mode (U) \[ \] Kernel-Mode (K)**

**Justification: These events are generated at the application level when login attempts are made to Office 365 accounts.**

**Final Summiting Score: 3A**

### **4. LOGIC & IMPLEMENTATION**

**Detection Logic (Pseudocode or SIEM Query):**

```py
rule successful_logins_after_brute_force_failures {
  // This rule matches single events. Rules can also match multiple events within
  // some time window. For details about how to write a multi-event rule, see
  // https://cloud.google.com/chronicle/docs/detection/yara-l-2-0-overview#single-event_versus_multi-event

  meta:
    // Allows for storage of arbitrary key-value pairs of rule details - who
    // wrote it, what it detects on, version control, etc.
    // The "author" and "severity" fields are special, as they are used as
    // columns on the rules dashboard. If you'd like to be able to sort based on
    // these fields on the dashboard, make sure to add them here.
    // Severity value, by convention, should be "Low", "Medium" or "High"
    author = "Lucian Peckham"
    description = "Detects Successful brute force attacks where if there is 10+ login failures within 10 minutes followed by a succeful login within a 1 hour time period, exxcluing trusted networks and whitelisted accounts"
    severity = "Medium"
    mitre_attack = "T1110.001, T1110.003"
    // references = ""


  events:
    // First we begin by seein gthe failed logons, this will later be used to compare the successful logins
    $failure.metadata.log_type = "OFFICE_365"
    $failure.security_result.action = "BLOCK"
    $failure.security_result.summary = /login.*fail|authentication.*fail|invalid.*credential/ nocase

    // were wanting these results to have trusted networks tuned out.
    not re.regex($failure.principal.ip, /^10\.|^172\.(1[6-9]|2[0-9]|3[01])\.|^192\.168\./) // RFC1918
    not re.regex($failure.extracted.fields["ip.isp.organization"], /Zscaler|Amazon\.com|Cloudflare|Akamai/) nocase

    //  We will then look at excluding whitelisted events
    not $failure.principal.user.userid = /^svc-|^a-/ nocase

    // successful login attempts
    $success.metadata.log_type = "OFFICE_365"
    $success.security_result.action = "ALLOW"
    $success.security_result.summary = /login.*success|authentication.*success|logon.*success/ nocase

    //  Same Exclusion for success events
    not re.regex($success.principal.ip, /^10\.|^172\.(1[6-9]|2[0-9]|3[01])\.|^192\.168\./) // RFC1918
    not re.regex($success.extracted.fields["ip.isp.organization"], /Zscaler|Amazon\.com|Cloudflare|Akamai/) nocase
    not $success.principal.user.userid = /^svc-|^a-/ nocase


    // Correlation portion: focusing on the users
    $failure.principal.user.userid = $success.principal.user.userid 
    $userid = $failure.principal.user.userid 

    // Time Sequencing: success within 1 hour after failures
    $failure.metadata.event_timestamp.seconds < $success.metadata.event_timestamp.seconds 
    $success.metadata.event_timestamp.seconds <= $failure.metadata.event_timestamp.seconds + 3600 // adjust at timeframe needed

  match:
    $userid over 10m

  outcome:
    $risk_score = max(75)

    // Aggregate event fields in match windows (required)
    $compromised_user   = array_distinct($userid)
    $failure_count      = count($failure.metadata.id)

    $first_failure_ts   = min($failure.metadata.event_timestamp.seconds)
    $last_failure_ts    = max($failure.metadata.event_timestamp.seconds)

    $success_ts         = min($success.metadata.event_timestamp.seconds)
    $failure_ips        = array_distinct($failure.principal.ip)
    $success_ips        = array_distinct($success.principal.ip)

    $attack_duration_seconds = max($success.metadata.event_timestamp.seconds - $failure.metadata.event_timestamp.seconds)

  condition:
    #failure >= 10 and $success
}
```

**Known False Positives:**

- Pending Further Testing

**Exclusion Strategy:**

- Filter trusted networks and whitelisted accounts.
- Add whitelist logic based on user roles, departments, or known service accounts.

### **5. VALIDATION & RESPONSE (Funnel of Fidelity)**

**Testing Procedures:**

Test Case 1 (Functional Synonym):

- **Tool/Procedure:** Simulate brute force attack using multiple failed login attempts followed by a successful login.
- **Result:** \[ \] Detected \[ \] Not Detected

Test Case 2 (Procedural Synonym):

- **Tool/Procedure:** Use credential stuffing tools to simulate attack.
- **Result:** \[ \] Detected \[ \] Not Detected

Test Case 3 (Sub-Technical Synonym):

- **Tool/Procedure:** Red Team tools (e.g., Hydra, Medusa).
- **Result:** \[ \] Detected \[ \] Not Detected

**Triage & Investigation Steps (Manual Playbook):**

**Initial Triage:**

* [ ] Identify the account and IP address involved.
* [ ] Check if the account is part of a known service or automation.
* [ ] Review login history for context.

**Investigation:**

* [ ] Query network logs for correlated traffic.
* [ ] Review related events (e.g., 4624, 4648).
* [ ] Inspect process ancestry using EDR tools.

**Response & Remediation:**

* [ ] If confirmed malicious, disable account and isolate host.
* [ ] Notify IR team and perform forensic analysis.
* [ ] Reset credentials and monitor for further activity.

### **6. AUTOMATION & RESPONSE PLAYBOOK (SOAR)**

**Alert Trigger Condition:**  
IF 'detection\_logic' RETURNS 'true'

**Alert Severity:**  
Default: Medium

**Enrichment Steps:**

- **Action:** Get-UserDetails
- **Input:** event.AccountName
- **Output:** user.title, user.department
- **Action:** Get-HostDetails
- **Input:** event.DeviceName
- **Output:** host.os, host.tags

**Containment Steps:**

- **Action:** Disable-User
- **Input:** event.AccountName
- **Execute:** Requires approval

**Notification Steps:**

- **Action:** Escalate to Tier 2
- **System:** SOAR
- **Assignee:** SOC Tier 2
- **Content:** Summary of enriched alert and actions taken.

### **7. FRAMEWORK MAPPINGS**

**MITRE Engage™ Mapping:**

- **Goal:** Disrupt (G0009)
- **Approach:** Detect (A0001)

**D3FEND Mapping:**

- **Tactic:** Detect (D3-DET)
- **Technique:** Credential Access Pattern Analysis
