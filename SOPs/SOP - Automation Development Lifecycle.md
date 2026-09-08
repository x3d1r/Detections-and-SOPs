# SOP \- Automation Development Lifecycle

```
graph TD

	new[Intake & Validation] --> plan[Planning] --> develop[Development] --> test[Testing] --> review[Peer Review] --> Deployment
	prod[Production]
	review -->|Recommend Fixes| develop
    subgraph Deployment
	    direction LR
	    dplan[Plan Deploy Datetime] --> dannounce[Announce Change Datetime] --> dmove[Move to Production Folder] --> dtrigger[Set Trigger] -->|If applicable| ddisable[Disable Current Playbook] --> dvalidate[Validate Playbook Deployment] --> dreview[Peer Review Deployment] --> notify[Announce Completion]
	    dtrigger --> dvalidate
    end
    Deployment --> prod[Production] --> Closure
```

# Purpose

This Standard Operating Procedure (SOP) establishes the full lifecycle of Google Secops (GSO) Automations (Playbooks, Blocks, and Custom Actions) from request, through production deployment, post-deployment validation, version archiving and documentation.

  
In this document, “Boards”, refers to two Azure DevOps boards being used to track work within the organization. This can also be used along side an ID of a story, feature or epic with the format of `<boad_abbr>-<task_id>` such as `TDO-12345`.

Detection Engineering Board: Private Azure DevOps board (link removed for public release).

Automation (SOAR) Board: Private Azure DevOps board (link removed for public release).

  
In this document, “Confluence” refers to the detection engineering confluence space.

Detection Engineering Confluence: Private Confluence space (link removed for public review).

# Scope

This SOP applies to any automation development performed by the detection engineering team.

# Lifecycle

## Stage 0 - Intake & Validation

This stage captures all new requests before they enter the planning pipeline. This stage’s primary goal is to ensure the request is de-duplicated and worth the team's time.

All Items enter here regardless of source: analyst request, leadership direction, threat intel feed, post-incident finding, vendor recommendation or self-identified gap.

### S0.1 Request Submission

Any team member or stakeholder submits a request by creating a ticket in the “New” column of the detection engineering board. The ticket requires minimal information, but the request should have enough context for the team to evaluate it.

Information should include:

- **Automation Goal **- What behavior or activity should this automate?  
- **Impact **- What is the potential impact of this automation? 
- **Expected Output** - What do you expect this playbook to output?  
- **Additional Context** - Any background information, related incidents, support details?

A template has been setup that can be accessed at when modifying a story, goto the top right triple-dots \> Templates \> “New Playbook / Playbook Enhancement Request”. This Template will delete replace the contents and title with the structure as well as tag the story with “Request” and “Automation”.

Accessing the Playbook Request Template.

### S0.2 Duplicate & Conflict Check

Before any research begins, check for the following:

- **Does this already exist in production? **A playbook, block, or action may cover this use case partially or fully.
- **Is there an open ticket for this already?** Check the backlog and in-progress columns for overlapping scope.
- **Does this conflict with something already in flight? **Something in development may overlap with this request, could it be an added feature?

if a duplicate or conflict is found, the new ticket should be closed with a link to the existing item and the requestor notified. If it is a partial overlap - consider merging the request into an existing one as a comment or subtask.

### S0.3 Research & Feasibility

Once deduped and confirm new, the team can perform some light research to ensure this is worth the team's time. Some questions that should be answered:

- **Does this request fall under Automation or the detection engineering team?** Sometimes people put requests in the wrong board, or think the detection engineering team is involved in log ingestion. 
- **Is it technically feasible?** Do we have the required integrations or APIs?
- **Has it been done before?** Can we find an existing integration? or would this require custom actions or integrations?
- **What is the effort level?** Will it small(hours), medium(days), Large(weeks or months). This should be a gut-check estimate. Consider consulting team members.

Research findings should be added as a comment to the ticket. For some requests, this may take 30 minutes. For complex requests this could take a day or two.

### S0.4 Intake Validation & Prioritization

The team should review intake tickets on weekly standups. Each ticket should receive an outcome:

| **Decision** | **Meaning** | **Next Action** |
| --- | --- | --- |
| **Approve** | Valid, feasible, worth doing | Move to Backlog; Stage 1 Planning begins when pulled. |
| **Defer** | Valid but not a current priority | Leave in intake with a defer note and maybe an estimated revisit date. |
| **Needs More Info** | Insufficient detail to evaluate; Template was not used | Notify requestor. Lack of Change or expansion should result in a deny. |
| **Reject** | Out of scope, not feasible, or already covered. | Close ticket with reason; notify requestor. |

For all items, Add a tag for the Decision: Approve, Defer, MoreInfo, or Reject.  
For approved items, the ticket is promoted to the backlog with an initial priority set 1 - 4, 1 being critical, add a rough estimate for Story points in days and link to the appropriate epic.

### S0.5 Break-fixes

Break-fixes typically bypass some stages entirely unless they require major changes to an integration, playbook or action. If something is interfering with operations, it should never wait in an intake queue and takes priority. Tag as “Breakfix”.

## Stage 1 - Planning

The goal of this stage is to setup the appropriate documentation and tasks, further research the topic and plan the implementation of the playbook, block or custom action. This will typically be the last stop for the requester’s story; a comment should be added and the original linked with the new feature/story and the original closed after planning is complete. All subsequent stories will be linked to the feature.

### S1.1 Feature & Story Creation

If the playbook, block or custom action is new, create a new feature with the request information as well as any research or planning results. You can create all the stage stories now or create them as you develop. All subsequent stories should be linked to this feature to track how much effort the feature required.

If the request is an enhancement or break-fix, a feature may exist for the automation and new stories can be used to track the progress of the enhancement through the lifecycle, linking the stories to the feature. 

### S1.2 Begin Documentation

If this is new request, begin the confluence documentation. This should be in the appropriate sub folder and will be used later for others to understand the element being created. This should include the following:

Expected basic documentation:

- **Stage **- Typically this would be in *Development *until it is in *Production.*
- **Target Date** (*conditional*) - If in development, what is the current estimate for deployment?
- **Use Case Summary** - What problem does this solve?
- **Dependencies **- What other blocks, tools, integrations, custom actions, APIs, etc do you plan on using? This may change as development continues and should be updated
- **Trigger **- Is this a manual playbook? Is it a block that is used in other playbooks? Does it trigger on all alerts of a certain type?
- **Expected Outcome** - What does it do? Containment, enrichment, escalation, case closure, workflow guardrails etc.?
- **Changelog** - This should be a table of all changes in stages or modifications in the future. This should include:
    - **Date **- When was it changed?
    - **Description **- Like the commit message or a short summary of changes. This could also just be the change of stages.
    - **Developer **- Who did the change?
    - **Reviewer **- Who reviewed the change?

## Stage 2 - Development

Build the thing! Development should occur in the development environment then be moved to production. The folder in Secops does not control the environment and is simply a folder structure. Setting the environment determines what blocks, integrations and data are accessible. Many of the integrations do not have a development environment and this caution is required when developing something that could change production data.

### S2.1 Playbook & Block Standards

These standards are relatively fluid and may require changing as new methods are discovered.

- Ensure playbook names line up with documentation to ensure it can be found easily in all platforms.
- Ensure all actions within a block or playbook have self-documenting names.
- Ensure all actions within a block or playbook have descriptions, describing what it is doing, with expected inputs and outputs. Another engineer should not have to ask what is being done.
- If a section could be repeatable or used in other playbooks, consider breaking out into a separate block. New documentation for the block is required.
- Remediation actions should be gated, requiring an analyst to confirm that action should be taken.

### S2.2 Custom Action Standards

These standards are relatively fluid and may require changing as new methods are discovered.

- All functions and Script blocks should be well documented following the [PEP-257 Docstring Conventions](https://peps.python.org/pep-0257/).
- Ensure the script name lines up with documentation to ensure it can be found easily in all platforms.
- Ensure the Description of the Custom Action is the same as the Use Case Summary from documentation and reference any Stories or Feature IDs.
- Ensure Parameters have descriptive names and descriptions.
- Ensure Parameter types are correct to hide sensitive data within the UI.

### S2.3 Version Control & Automation Backups

Due to the lack of auditing and version control options within Google Secops, there are very little controls to prevent or audit someone modifying or deleting a playbook, block or custom action. It is recommended that the team utilize Secops Version control as well as a Git repository for historical backups and version control. 

Any commit message or versioning comments should likely follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) and include a type, scope and a message. Utilizing breaking change symbols may also be useful for quick a quick glance.

**For Playbooks, Blocks and Custom Actions within Secops**, engineers should utilize “Saved Versions” to keep track of major changes to production playbooks. This option also gives you the ability to quickly restore older versions of the playbook and block as well as a comment that can be used to track why something was changed. To ensure there is good change management - the comment should look something like `fix(SOS-12345): updating trigger`. This denotes that the change was a fix for Story 12345 on the SOAR board, as well as a simple description.


**For keeping track of Playbooks, Blocks and Custom Actions outside of Secops**, the best option is to utilize Azure DevOps Git to keep track of these changes. Playbooks and blocks can be exported via the hamburger icon then clicking export. This will give you the JSON representation of the playbook or block. This can then be used with git to keep track of those changes. 

Exporting a Playbook Json object using the hamburger menu

Since there will be multiple types of objects in the git repo - Git commit messages should follow a conventional commit style similar to `fix(Playbook-Auscert Playbook V2): SOS-12345 - updating trigger`. This denotes the git commit only changed the Auscert Playbook V2 Playbook for the Story 12345 on the SOAR board.

## Stage 3 - Testing

Testing is a mandatory step for all work types and ensures the new or modified automations function as expected. Your goal in this stage is to attempt to break your newly created automation across every branch, failure conditions, identify edge cases and document the results. Unacceptable failures at this stage should result in more development time.

Initial test cases should be identified or created that map the specific branches or functions of the automation. If your playbook has 4 branches, each branch should have a case tied to it to test that specific branch from start to finish. If a case does not exist, attempt to create a case that would result in that branch or functions execution. 

Attempt to identify or create edge cases that your automation should be able to handle. For example, if your playbook utilizes input such as a username or email, create a test case to test to test usernames with special or unexpected characters, capitalization, or domains appended to it.

### S3.1 Simulator Mode

Simulator button and Banner that states the block or playbook is in Simulation mode.


Within Secops, Playbooks and Blocks can be switched into “Simulation” mode allowing you to run a test case through a playbook or Block. Once in Simulator mode, you can select a case at the bottom of the screen.

Simulation mode test case selector

After a simulation has run, you can then see the results of each step allowing you to identify any potential issues and see the raw values at each step.

### S3.2 Custom Action Testing

For Custom Actions, Secops gives you the option of providing inputs in the “Testing“ tab. These inputs can be statically defined or extracted from a selected Test case. After you input is added to the “Parameters” section (1), you can hit the “Play Item” button (2). Your results will be in the lower result section (3) allowing you to see either JSON output or output messages from your custom action.


### S3.3 Generating Simulation Test Cases

The “Ingest alert as test case” button

In Secops, any existing alert and be turned into a test case. This feature makes the alert show up in simulation and testing mode and can be used as input for your playbooks, blocks and custom actions. Simple:

1. Open the alert item within the case
2. Click on the 3 dots at the top right
3. Click on “Ingest alert as test case“
4. Find the newly created test case in simulation mode!

### S3.4 Testing Documentation

There should be clear documentation in the testing story that shows all branches were tested and edge cases were identified and tested. If a branch or edge case fails, that should result in a new story being created to fix those issues. If all was successful and identified edge cases were successful, the testing story can be closed, and peer review can verify the alert is ready for production. There is also a chance for the reviewer to identify edge cases. The developer should also ensure the documentation in confluence is up to date including all dependencies.

## Stage 4 - Peer Review

Peer review is the quality gate before deployment. The goal is to ensure standards are met for a production automation with a second set of eyes. This stage must be completed and signed off before any deployment activity can begin. This step may be skipped for operation breaking fixes where speed is required to restore production functionality such as a change in API.

### S4.1 Reviewer Checks

The reviewer is accountable for verifying both the technical implementation as well as the documentation. Their checks should cover:

- **Code / Logic Quality** - Ensure the logic meets standards defined in this SOP. Task naming, error paths, playbook structure.
- **Testing Evidence** - Ensure test results are documented, testing results shows that all branches and functions were fully testing. Review any edge cases and attempt to identify any other potential edge cases.
- **Confluence Documentation** - Ensure the documentation meets standards defined in this SOP. Name of the document matches the naming of the automation. Expected output matches the automation output; dependencies are listed correctly.
- **Production Ready** - Document anything that may need to be altered before production. Some of these may require pushing back to development.
- **Version Control & Backups** - If all looks good, ensure the current version is saved within Secops Version Control, and a version has been backed-up within git.

### S4.2 Deployment Risk Identification

The reviewer also has the task of identifying the **Deployment Risk **which should be classified into either “High” or “Low Risk” and determines whether or not a scheduled deployment is required. 

Utilize the criteria defined in the High Risk Criteria and Low Risk Criteria sections (internal Confluence link removed for public review) to determine the risk. Ensure the risk is noted in the Review comments with a brief summary of the components that would be affected by the deployment, if any.

### S4.3 Reviewer Sign-off

Once the review has been completed, the reviewer should update the story, update the confluence changelog and inform the developer that the automation is ready for deployment. 

## Stage 5 - Deployment

The deployment stage is structured around potential risk. Rather than applying the same overhead to all changes, deployments will be classified into either high or low risk, determining which steps are completed. All changes should still be announced after completion in both “Detectioneers” and Global CSIRT chats. The deployment risk should be identified during Stage 4 - Peer Review.

### S5.1 High Risk Criteria

A deployment is high risk if it meets any of the following criteria:

- **Modifies or replaces automations already in production** - an existing playbook, block, integration, API etc.
- **Could affect other automations** - integrations, shared blocks etc.
- Misconfiguration could result in impact to existing production workflows.

### S5.2 Low Risk Criteria

A deployment is Low Risk if it meets all the following criteria:

- **The content is net-new** - it does not modify anything currently in production.
- It will not be referenced or utilized by any existing playbook or block.
- Misconfiguration would not have any impact on existing production workflows.

### S5.3 High Risk Pre-Deployment

High risk deployments require a scheduled change window as well as a change announcement. 

- **Identify a deployment window** - Try to coordinate with anyone that should involved in the change to ensure everyone can make that time.
- **Post an announcement** - The announcement should be in both “Detectioneers” and “Global CSIRT Chat” announcing the change window and subject.

### S5.4 Deploy Process

The deployment process varies depending on the type of automation being implemented and whether or not something is being replaced. All events in this process require you to be logged into Google Secops.

#### S5.4.1 Net-New Playbook and Block Deployment

If the environment of the development block or playbook is set to anything other than “Default Environment”, the block or playbook will need to be recreated with the environment set to “Default Environment”.

1. Move the development block/playbook from the “Development” folder to the “Production” folder.
2. If playbook, set the trigger to the correct value.

#### S5.4.2 Playbook Replacement

If the environment of the block or playbook is set to anything other than “Default Environment”, the block or playbook will need to be recreated with the environment set to “Default Environment”.

1. Move the development playbook from the “Development” folder to the “Production” folder.
2. Move the old playbook from the “Production” folder to the “Development” folder.
3. Set the trigger on the new playbook to the same as the original playbook.
4. Set the trigger from the old playbook to something that should not exist.

#### S5.4.3 Block Replacement

1. Move the development playbookfrom the “Development” folder to the “Production” folder.
2. Move the old playbookfrom the “Production” folder to the “Development” folder.
3. Set the trigger on the new playbook to the same as the original playbook.
4. Set the trigger from the old playbook to something that should not exist.

1. Use the “Playbooks using this block” to get a list of all playbooks that need to be modified.

“Playbooks using this block” button

1. Modify each playbook in that list, replacing the old block with the new block.
2. Confirm the “Playbooks using this block” list is empty in the old block. Ensure the new list matches.


want to include neels doc topics here, cover moving to the prod folder and/or copy pasting actions to the new prod action etc.

keep track of any changes to ensure we can roll back smoothly.

### S5.5 High Risk Post Review

ensure triggers are correct, env is correct, old playbooks are disabled etc

### S5.6 Change Complete Notification and Documentation

tell global we done - update documentation to ensure it is “in-prod” and 

## Stage 6 - Post-Deployment Validation

Smoke test - lets make sure the next couple of days go smooth or whenever a real event uses the playbook. be prepared for rollback.
