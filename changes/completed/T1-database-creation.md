# T1 - Database creation

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T1
- **Type:** architecture
- **Owner:** Karina Kentsch
- **Requirement(s):** R7, R8
- **Use case(s):** 
- **Workflow:** architecture
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** -
- **PR link(s):** docs/entities-specs
- **Tests:** 
## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Database creation” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** architecture. **Requirements:** R7, R8. **Depends on:** none.
- Choose the database and set up a local instance.
- Write the schema script (DDL), with no migration tool.
- Write `docs/entities/<entity>.md` in the workspace for every entity: User, Organization, Membership, Category, Event, Registration, FollowUp, Message, Task, Announcement. Each spec lists the fields, types, constraints and relationships.
- The organization identifier is `organization_id` (there is no separate `organizationId`). Membership `role` is intended to represent MEMBER or MANAGER; nullable `status` represents PENDING (approval not processed) or ACTIVE (active membership). Membership requests are represented by membership status; there is no separate MembershipRequest entity.
- Events belong to an organization and a category. Event `status` is DRAFT or PUBLISHED for active events, and CANCELED for canceled/deleted events. The schema has a `date`, `start_time`, and nullable `end_time`; it has no separate `deleted` field or creator field.
- Registration has event and user references. The current schema does not define a registration status or enforce one row per user and event.
- Seed script for the application administrator: a few organizations, each with a first manager (a user with a hashed password and a MANAGER membership). It is reused by deployment and by the tests.
- Needs a workspace change: a `docs/` folder, readable by the agents (see "Workspace changes needed").

Task specification: create the entity specifications under `/docs/entities/`, one file per entity: `User.md`, `Organization.md`, `Membership.md`, `Category.md`, `Event.md`, `Registration.md`, `FollowUp.md`, `Message.md`, `Task.md`, and `Announcement.md`. Each file records the entity fields, types, constraints, and relationships based on the current database schema. MembershipRequest is represented through membership status. Message is the entity stored in the `messages` table.

Dependencies from the backlog: none.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** 
- **Related completed tasks:** 
- **Constraints / assumptions:** 

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

### Steps

1. 

### Files to create / modify

### Tests to write

### Risks / open questions

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** 
- **Result:** 
- **Failure reason:** 

### Green (after implementation)

- **Result:** 
- **Full suite of the affected repo:** 

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The original entity list and key-field description did not match the Supabase schema and clarified entity/status structure. | Updated the task specification to use the current schema: added Category, represented membership requests through Membership.status, named the messages/tasks entities Message and Task, and aligned organization, membership, event, and registration details. | To make T1 describe the database structure provided by the user. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** 
- **Alternatives considered:** 
- **Limitations:** 

## Status

- [ ] Task read and context gathered
- [ ] Plan written
- [ ] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [ ] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [ ] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`
