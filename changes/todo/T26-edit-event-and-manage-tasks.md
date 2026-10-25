# T26 - Edit event and manage tasks

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T26
- **Type:** workflow
- **Owner:** Blaj Horatiu 
- **Requirement(s):** EVT-02, EVT-06, ORGZ-02
- **Use case(s):** UC-07
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T26-edit-event-and-manage-tasks`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Edit event and manage tasks” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** EVT-02, EVT-06, ORGZ-02. **Depends on:** Organization-scoped authorization, Init Registration entity, Event lifecycle model, Create an event with tasks.
- `PUT /api/organizations/{organizationId}/events/{eventId}`: edit the event details with permission and state checks and validation. Capacity cannot drop below the active registrations. A canceled event cannot be edited. Any manager of the organization can edit.
- Add, edit, reassign and remove tasks under `.../events/{eventId}/tasks` and `.../tasks/{taskId}`, rejecting a non-member assignee.

Dependencies from the backlog: 9, 21, 22, 23.

**Validation rules moved here from T28 (publish event).** Publishing only checks that the event starts in the future, so editing an event must enforce these on the updated values (decided by Blaj Horatiu in T28, Human Corrections #2). They're the same rules as T23 (create); **reuse T23's validation method** in `EventServiceImpl` rather than duplicating it.

- `title` is required and not blank.
- `description` is required and not blank.
- `date` and `startTime` are required, and the start (`date` + `startTime`) must be **in the future**, measured with the injected `java.time.Clock` (`ClockConfig`, Europe/Bucharest).
- `endTime` is required and must be **after `startTime`** (same day; events can't cross midnight).
- `location` is required and not blank.
- `capacity` is required and must be **> 0**, and (from the backlog) **not below the number of active registrations**.
- Collect **every** problem as an `ErrorDetail(field, message)` and throw `BusinessRuleException("…", errors)` → 400, using the field names `title`, `description`, `date`, `startTime`, `endTime`, `location`, `capacity`.

**Notes from the work already done:**

- A **CANCELED** event can't be edited (backlog) → `ConflictException` (409), consistent with T28's "a canceled event cannot be published".
- **Editing a PUBLISHED event:** these rules apply too, because publishing no longer re-validates the details. Whether a published event can still be edited at all is a T22/UC-07 decision; ask the user.
- Task assignees must be **ACTIVE members** of the event's organization, the same check as T23.
- Reassigning or removing tasks also covers the T27 limitation: tasks stay assigned to users who left the organization until someone reassigns or removes them.
- **Code:** put it in the existing `EventController`/`EventService` (root `/api/organizations`) and `EventTaskController`/`EventTaskService`, with `@RequireOrgPermission(EffectiveOrgRole.MANAGER)`. An unknown or foreign event or task gives 404 (`NotFoundException`). Follow the same DTO, test and branching conventions as noted in T23.

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
|   |                               |                |                              |     |

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