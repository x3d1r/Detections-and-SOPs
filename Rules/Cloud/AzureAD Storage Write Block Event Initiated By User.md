# AzureAD Storage Write Block Event Initiated By User

# 1. METADATA

**Analytic Title:** AzureAD Storage Write Block Event Initiated By User

**ID:** DE-TA0040-T1562.007-004

**Version:** 1.0

**Status:**

- Experimental
- Test
- Production

**Author:** Lucian Peckham

**Date Created:** TBD

**Date Modified:** 2026-08-03

---

# 2. DETECTION OVERVIEW

## Description

Detects user-attributed Azure Storage account write operations that were blocked by a security or governance control.

The rule evaluates Microsoft Azure Activity events where:

- The activity is associated with Azure Storage.
- The operation is `MICROSOFT.STORAGE/STORAGEACCOUNTS/WRITE`.
- The resulting security action is `BLOCK`.
- A principal user display name is populated.

This activity was identified from behavioural artifacts associated with engagement activity.

A blocked storage account write may indicate an attempt to create or modify an Azure Storage account in a way that violates an Azure Policy, organisational restriction, security control or other platform enforcement mechanism.

Although the requested action was blocked, the attempt remains security-relevant because it may identify:

- An unauthorised configuration change.
- An attempted weakening of storage security controls.
- Use of a compromised Azure user account.
- An administrator operating outside an approved change process.
- A misconfigured deployment or automation process.
- An attempt to create or modify infrastructure in a prohibited region.
- An attempt to apply a configuration that violates organisational policy.

The detection returns the affected storage resource, principal location, operation type and number of matching source events.

## Hypothesis

We hypothesize that an adversary with access to an Azure user account may attempt to create or modify an Azure Storage account to support collection, staging, persistence, command and control, data transfer or other cloud-based activity.

Where the requested configuration conflicts with Azure Policy or another security control, the operation may be blocked.

This behaviour can be observed through Azure Activity events where:

- The log type is `AZURE_ACTIVITY`.
- The vendor is Microsoft.
- The product event type is `MICROSOFT.STORAGE/STORAGEACCOUNTS/WRITE`.
- The security action is `BLOCK`.
- The event contains a populated principal user display name.

## Security Significance

Azure Storage accounts can contain business data, application data, logs, backups, deployment artifacts and other sensitive information.

Storage account write operations may include attempts to create a new storage account or modify the configuration of an existing account.

A blocked write operation may indicate that an identity attempted to perform an action that was not permitted by the environment’s security or governance controls.

The operation itself was blocked, but investigation is still required to determine:

- What configuration was attempted.
- Which user initiated the operation.
- Whether the user was authorised.
- Which policy or control blocked the operation.
- Whether the same user attempted other storage or resource changes.
- Whether similar activity later succeeded.
- Whether the affected identity or source context is suspicious.
- Whether the event resulted from legitimate deployment activity.

## MITRE ATT&CK Mapping

The blocked write event does not independently identify the adversary’s objective. The correct mapping depends on the configuration that was attempted.

### Primary Contextual Mapping

**Tactic:** Defense Evasion

**Technique:** Impair Defenses

**Sub-technique:** Disable or Modify Cloud Logs, T1562.008

This mapping should only be applied when investigation confirms that the attempted storage account modification would have disabled, redirected, weakened or otherwise affected cloud logging.

### Potential Secondary Mapping

**Tactic:** Defense Evasion

**Technique:** Impair Defenses

**Sub-technique:** Disable or Modify Cloud Firewall, T1562.007

This mapping should only be applied where the attempted storage change involved network rules, firewall configuration, public access controls or trusted service access.

### Conditional Mapping

**Tactic:** Collection

**Technique:** Data from Cloud Storage

**Technique ID:** T1530

This mapping may apply where the attempted storage account change was intended to enable access to data held in cloud storage.

### Conditional Mapping

**Tactic:** Exfiltration

**Technique:** Exfiltration to Cloud Storage

**Technique ID:** T1567.002

This mapping may apply where the attempted activity involved creating or modifying storage to stage or transfer data outside an approved location.

### Identity Context Mapping

**Tactic(s):**

- Initial Access
- Persistence
- Privilege Escalation
- Defense Evasion

**Technique:** Valid Accounts

**Sub-technique:** Cloud Accounts, T1078.004

This mapping may apply if investigation confirms that a compromised or improperly used Azure account initiated the write attempt.

The current rule does not confirm account compromise, intent or the precise storage configuration that was attempted. MITRE mappings should be refined after the blocked policy and requested configuration are identified.

---

# 3. ANALYTIC DETAILS

## Analytic Robustness Level

**Selected Level:** Level 3, Core to Pre-Existing Tool

### Justification

The detection is based on Azure control-plane audit events generated when a storage account write operation is attempted and blocked.

The analytic does not depend on a particular attacker-controlled executable, script name, command line or administrative interface. The operation could be initiated through:

- The Azure portal.
- Azure CLI.
- Azure PowerShell.
- Azure Resource Manager.
- An infrastructure-as-code deployment.
- A CI/CD pipeline.
- A custom application.
- Another management interface using Azure APIs.

The event is generated by a pre-existing cloud platform and represents an attempted Azure resource operation.

The detection is assessed as Level 3 because it relies on specific UDM-normalised values:

- `AZURE_ACTIVITY`
- `Microsoft`
- `MICROSOFT.STORAGE/STORAGEACCOUNTS/WRITE`
- `BLOCK`

Changes to source logging, parser behaviour, event naming or action normalisation could affect coverage.

The rule also identifies user initiation by checking whether a user display name is populated. This does not provide an invariant distinction between interactive user activity and automation.

- Level 1: Ephemeral
- Level 2: Core to Adversary-Brought Tool
- **Level 3: Core to Pre-Existing Tool**
- Level 4: Core to Some Implementations of a Sub-technique
- Level 5: Core to a Sub-technique, Invariant Behaviour

## Data Source and Event Robustness

**Platform:** Microsoft Azure

**Source:** Azure Activity Logs ingested into Google SecOps

**Log Type:** `AZURE_ACTIVITY`

**Vendor:** Microsoft

**Operation:** `MICROSOFT.STORAGE/STORAGEACCOUNTS/WRITE`

**Security Action:** `BLOCK`

**Relevant Event Fields:**

- `metadata.log_type`
- `metadata.vendor_name`
- `metadata.product_event_type`
- `metadata.id`
- `metadata.event_timestamp`
- `security_result.action`
- `security_result.summary`
- `security_result.description`
- `principal.user.userid`
- `principal.user.user_display_name`
- `principal.user.email_addresses`
- `principal.ip`
- `principal.ip_geo_artifact.location.country_or_region`
- `principal.application`
- `target.resource.name`
- `target.resource.resource_type`
- `target.resource.product_object_id`

## Event Robustness Column

### Host-Based

- Application, A
- User-Mode, U
- Kernel-Mode, K

### Justification

The event is generated by the Azure control plane and recorded through Azure Activity logging.

It is not endpoint user-mode or operating system kernel telemetry. The event records an attempted operation against an Azure resource and the resulting control decision.

The event’s reliability depends on:

- Azure generating the Activity Log event.
- The event being successfully collected.
- The event being normalised into the expected UDM fields.
- The operation name being retained.
- The blocked result being mapped consistently.
- The principal identity being populated.
- The target resource being identified correctly.

### Network-Based

- Protocol Payload, P
- Protocol Header, H

Network-based scoring is not applicable because the detection evaluates Azure control-plane audit telemetry rather than network traffic.

**Final Summiting Score:** 3A

---

# 4. LOGIC AND IMPLEMENTATION

## Detection Logic


rule soc\_AzureAD\_Storage\_Write\_Block\_Event\_Initiated\_By\_User {

 meta:

 author = "Lucian Peckham"

 description = "Detects Azure Storage account write operations blocked by policy where a principal user display name is populated; built from engagement artifacts"

 severity = "High"

 events:

 $app.metadata.log\_type = "AZURE\_ACTIVITY"

 $app.metadata.vendor\_name = "Microsoft"

 $app.metadata.product\_event\_type =

 "MICROSOFT.STORAGE/STORAGEACCOUNTS/WRITE"

 $app.security\_result.action = "BLOCK"

 re.regex(

 $app.principal.user.user\_display\_name,

 ".+"

 )

 outcome:

 $risk\_score = 60

 $event\_count = count\_distinct(

 $[app.metadata.id](http://app.metadata.id)

 )

 $principal\_country\_region = array\_distinct(

 $app.principal.ip\_geo\_artifact.location.country\_or\_region

 )

 $target\_storage\_resource = array\_distinct(

 $app.target.resource.name

 )

 $app\_event\_type = array\_distinct(

 $app.metadata.product\_event\_type

 )

 condition:

 $app

}


## Plain-Language Logic


IF the event is an Azure Activity event

AND the vendor is Microsoft

AND the operation is a storage account write

AND the security action is BLOCK

AND a principal user display name is populated

THEN generate a high-severity detection

AND return the event count, principal country or region,

target storage resource and Azure operation type.


## Known False Positives

Potential legitimate triggers include:

- An authorised administrator attempting a storage account configuration that violates Azure Policy.
- An infrastructure-as-code deployment containing a non-compliant configuration.
- An approved deployment using an incorrect region, SKU or storage configuration.
- A CI/CD pipeline running under a user-associated identity.
- An administrator testing policy enforcement.
- A development or test deployment.
- A user attempting to create a storage account without the required permissions.
- A user attempting to modify a protected storage account.
- An attempt to enable public access where policy prohibits it.
- An attempt to disable a required security setting.
- An attempt to configure an unsupported network access model.
- An attempt to deploy a storage account in a prohibited location.
- An attempt to use an unapproved encryption configuration.
- An attempt to change resource properties outside an approved template.
- A deployment using an outdated Azure Resource Manager or infrastructure-as-code template.
- A policy assignment that was recently introduced or modified.
- An authorised change that was not tested against current Azure Policy requirements.
- A legitimate operation blocked by role-based access control rather than security policy.

## Exclusion Strategy

Exclusions should only be introduced after the specific cause of the blocked operation has been identified.

Where available, an exclusion should use a combination of:

- Principal user ID.
- Principal application ID.
- Target storage account resource ID.
- Azure subscription ID.
- Resource group.
- Exact operation.
- Policy assignment or policy definition.
- Deployment name.
- Approved source IP range.
- Change or service request.
- Expected maintenance window.
- Known automation identity.
- Expected deployment pipeline.

An event should only be excluded when:

1. The initiating user or workload is known.
2. The target resource is known.
3. The requested configuration has been reviewed.
4. The reason for the block is understood.
5. The activity is associated with an approved process.
6. The operation is expected to recur.
7. The exclusion has documented approval.
8. The exclusion has a defined owner.
9. The exclusion has a review or expiry date.
10. Alternative monitoring remains in place for unexpected activity.

Exclusions should not be based solely on:

- User display name.
- Storage account display name.
- Country or region.
- A broad Azure administrator group.
- The fact that the operation was blocked.
- A broad source IP range.
- The assumption that blocked activity cannot cause harm.

A blocked event may be part of a wider sequence where the user attempts several configurations until one succeeds. Expected users should therefore not be broadly excluded from storage write monitoring.

## Implementation Notes

### Product and Rule Naming

The rule is named:


soc\_AzureAD\_Storage\_Write\_Block\_Event\_Initiated\_By\_User


However, the source log type is:


$app.metadata.log\_type = "AZURE\_ACTIVITY"


The event is an Azure Activity control-plane event rather than an Azure AD audit event.

To align the rule name with the actual event source, consider renaming it to:


soc\_Azure\_Storage\_Write\_Block\_Event\_Initiated\_By\_User


This will help avoid confusion during rule maintenance, investigation and data source troubleshooting.

### Description Accuracy

The current description refers to:


Azure AD storage account writes


Storage accounts are Azure resources, not Azure AD resources.

A more accurate description is:


Detects Azure Storage account write operations blocked by policy where the event contains a principal user display name; built from engagement artifacts.


### User-Initiated Identification

The rule identifies user initiation using:


re.regex(

 $app.principal.user.user\_display\_name,

 ".+"

)


This confirms that the user display name contains at least one character.

It does not independently confirm that:

- The operation was interactive.
- The user directly performed the action.
- The event was not generated through delegated automation.
- The application was not acting on behalf of the user.
- The display name represents a human account.
- The principal was not a service or automation account.

Where available, user attribution should be strengthened using:

- Principal user ID.
- Principal user email address.
- Principal application.
- Authentication type.
- Caller identity.
- Claims or token context.
- User agent.
- Source IP address.
- Service principal identifier.

### Required Principal User ID

The current rule does not capture the principal user ID in the outcome.

A display name may not be unique and may change over time. Where populated, the user ID or UPN should be added to support reliable investigation.

Suggested collection:


$app.principal.user.userid = $principal\_user\_id


Suggested outcome:


$principal\_user = array\_distinct($principal\_user\_id)


Before making this field mandatory, confirm that it is reliably populated across representative events. If it is not consistently available, requiring it may reduce detection coverage.

### Source IP Address

The rule returns the country or region derived from the source IP, but it does not return the source IP itself.

The source IP is more useful for:

- Threat intelligence enrichment.
- Sign-in correlation.
- Known network validation.
- VPN or hosting classification.
- Identifying repeated activity.
- Correlating other Azure operations.

Where available, capture:


$app.principal.ip = $principal\_ip


Then return:


$principal\_ip\_addresses = array\_distinct($principal\_ip)


If the field is not consistently populated, it should not be made a required detection condition.

### Outcome Typo

The current outcome uses:


$principal\_county\_region


This appears to be a spelling issue.

Use:


$principal\_country\_region


### Target Variable Naming

The current outcome uses:


$target\_application


The target is expected to be an Azure Storage resource rather than an application.

A more accurate outcome name is:


$target\_storage\_resource


This helps avoid confusion with Azure application registrations and service principals.

### Block Reason

The current rule checks:


$app.security\_result.action = "BLOCK"


However, it does not return the reason the operation was blocked.

Where available, enrich the detection with:

- Security result summary.
- Security result description.
- Azure status code.
- Sub-status.
- Policy assignment.
- Policy definition.
- Error code.
- Error message.
- Operation result.
- Authorisation failure details.

This context is required to distinguish:

- Azure Policy denial.
- Role-based access control denial.
- Resource lock enforcement.
- Invalid configuration.
- Platform validation failure.
- Other blocked or denied operations.

### Write Operation Scope

The operation:


MICROSOFT.STORAGE/STORAGEACCOUNTS/WRITE


may represent the creation or update of a storage account.

The current rule does not state what property the user attempted to change.

Investigation should determine whether the write involved:

- Creating a storage account.
- Changing network access.
- Changing public access settings.
- Changing encryption settings.
- Changing identity configuration.
- Changing replication or redundancy.
- Changing location or deployment properties.
- Changing tags or other administrative metadata.
- Updating another storage account property.

The exact attempted configuration should be derived from the source event or associated deployment context where available.

### Event Count

The rule uses:


$event\_count = count\_distinct($[app.metadata.id](http://app.metadata.id))


This is valid for returning distinct matching event identifiers.

Because the rule has no match section or time-based grouping, it currently operates as a single-event rule. The event count will generally describe events within the individual detection context rather than repeated attempts over a defined correlation window.

If the intended behaviour is to detect repeated blocked attempts, a separate multi-event rule should group by one or more of the following:

- Principal user.
- Source IP address.
- Target storage account.
- Subscription.
- Resource group.
- Policy.
- Operation type.

### Risk Score and Severity Alignment

The rule metadata specifies:


severity = "High"


The outcome specifies:


$risk\_score = 60


Confirm that a risk score of 60 aligns with High severity in the organisation’s detection risk model.

Severity could be adjusted using enrichment:

- Medium for a single blocked operation linked to an approved deployment.
- High for an unexplained user-initiated blocked write.
- High for repeated blocked operations.
- Critical where the identity is compromised or a similar write later succeeds.
- Critical where the attempted change would have weakened logging, public access, network controls or encryption.
