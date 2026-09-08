# SOP \- Scrumban Workflow

# Purpose

This Standard Operating Procedure (SOP) establishes a Scrumban workflow for the detection engineering team.

In this document, “Boards”, refers to two Azure DevOps boards being used to track tasks within the organization. This can also be used alongside an ID of a story, feature or epic with the format of `<boad_abbr>-<task_id>` such as `TDO-12345`.

Detection Engineering Board: Private Azure DevOps board (link removed for public release).

Automation (SOAR) Board: Private Azure DevOps board (link removed for public release).

# Scope

This SOP applies to all Azure DevOps task tracking performed by the detection engineering team.

# Introduction

The goal of this SOP is to provide a guideline for a Scrumban workflow, combining the simple, continuous, visual flow of Kanban with just enough structure from Scrum to keep the team aligned on work being requested and completed. This approach also sets the team up for expansion into a full Scrum workflow as the team requires it.

The structure should be thought of as a “Folder” structure.  Epics capture the broad team goals and initiatives, Features track the lifecycle of development and stories are assignable, bite sized estimates of work that needs to be completed.

# Epics - Team Goals

Epics should map to a **team objective **with a defined scope. All work completed should align to one of these goals, showing how your day to day has contributed but also showing the effort put in for each goal.

## Guidelines

- Each epic should have a description explaining the business or operational justifications and expected results.
- A Target date field in ADO gives a soft deadline without locking into full sprint commitments. These Epics can also be used to chunk quarterly goals and help prioritize quarterly stories.
- The percent complete and story count calculations can be used as indicator for how much work has been completed for that goal.

# Features - Alert Logic & Automation Lifecycle Tracking

In the detection engineering workflow, the features will represent any development being performed, allowing a high-level tracking of where each alert or automation is without having to track down the individual stories that have been created. These will typically be created in the “Planning” stage of the development lifecycles.

## Guidelines

- The description should include what is being built and why it matters.
- This should generally include similar information to the confluence documentation and is set during the planning stage of the lifecycle.
- Link or state any relevant threat intel, ticket numbers, or prior related work for context.
- Use the Target Date to set an estimated completion date.
- Alert lifecycle will be tracked on the detection engineering board, and Automations will be tracked on the automation board.
- Each feature should be linked to a parent Epic explicitly so the stats and tracking works correctly.
- Because the features track the entire lifecycle, future breakfix stories should be linked to the original feature.
- The feature board columns should follow the appropriate board lifecycle stages allowing for the quick glance of all in development features. *\<TODO: Link to each lifecycle high level\>*.
- You should link the feature to the confluence documentation for the automation or logic being developed. The feature should contain a summary of the same information within requests.
- The names of the features should match the automation or logic documentation.

# Stories - Progress & Task Tracking

Stories are the individual tasks that define what actually needs to be done. For most development tasks, there should be at least one story for every stage of the development lifecycle but can easily be broken up into smaller tasks that can be assigned separately.

## Guidelines

- Story descriptions should be a clear action statement describing exactly what needs to be done. It should have enough information that someone picking this up should know where to start without asking anyone.
- It is generally a good idea to have an expected output or deliverable.
- Include any specific constraints or context that may affect how the work should be completed.
- Ensure all stories are linked to either a parent Feature or Epic.
- Estimate story points in days. This does not have to be perfect and is used to track how long to expect something to take. The timeframe may change to hours as the team becomes more efficient.
- Once the story is complete, set the story points to exactly how many days it took.
- The story should be moved to the appropriate stage to effectively track in progress task.
- New stories could be requests from other stakeholders or from the team itself. These should be reviewed by a team member before being moved into a backlog. Any questions should be raised on the team meeting.
- Utilize comments to document work completed on the task.

## Prioritization

Every story in the backlog should have a priority. The priority should be a rating of 1-4, set at the time of creation or planning and revisited during backlog reviews. Priority drives the order in which tasks are completed. For people looking for a task, they can use the backlog tab to easily identify what needs completed by what is higher priority or at the top of the list.

| **Priority** | **Label** | **Description** |
| --- | --- | --- |
| 1 | Critical | Should be completed as soon as possible. Directly impacts production or a stakeholder commitment with a hard date. Work should generally stop on lower priority items. |
| 2 | High | Important, time-sensitive task with clear business or operational value. |
| 3 | Normal | Planned, valuable task with no immediate deadline pressure. |
| 4 | Low | Nice-to-have tasks or work that is too vague to act on. Tasks here should either be promoted or closed in backlog reviews. |

# Tag & Title Convention

Tags are generally recommended to easily filter and track work of a certain type. Tags can help you quickly identify what type of work it is without looking at the details. Due to some views not showing tags, it is generally a good idea to have some tags within the title such as “Request | Logic | Domain Admin Additions“. This shows that it is a “Request” for “Logic” with a title for the request. For features you would generally have a simpler title structure that is just the type of work as well as the technology such as “Playbook | Playbook Name“ or “Playbook | Technology | Playbook Name“.

| **Tag** | **Target** | **Description** |
| --- | --- | --- |
| Request | Stories | Tasks that originated from a stakeholder. |
| Logic | Stories, Features | Detection logic related creation or modifications. |
| Automation | Stories, Features | Any automation related work |
| Playbook | Stories, Features | SOAR playbook specific work |
| Block | Stories, Features | SOAR Block specific work |
| Custom Action | Stories, Features | SOAR Custom action specific work |
| Tuning | Stories | Anything that requires modifying logic, adding/modifying filters or managing agentic memories to reduce false positives. |
| Research | Stories | Any work that may require investigation or research. |
| Enrichment | Stories | Any work that is changing existing logic/automations/documentation to provide more information. |
| Reporting | Stories | Work that involves generating reports from the detection engineering team. |
| Review | Stories | For any work that needs reviewed by the team or a specific individual. |
| Decom | Stories | Work related to decommissioning something in production. |

# Backlog Review

The backlog should be the **single source of truth** for all pending and planned work. A healthy backlog should not be a dumping ground and thus requires periodic grooming and reprioritization. Keeping it clean is a shared responsibility but changes due to operations or recent incidents. If an item lingers in the backlog too long, it may need to have its priority upgraded or simply closed because it no longer has value.  The goal of the review is to **ensure the top of the backlog is always ready to be picked up**.

## What Gets Reviewed

- Any new Requests that have questions regarding its priority, or value.
- Existing backlog items should be re-ranked if priorities have shifted, updating the Priority of the individual story.
- Items that have been sitting in the backlog longer than 60 days without movement should be reviewed, re-prioritized or simply closed and deferred.
