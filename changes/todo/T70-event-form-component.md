# T70 - Event form component

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T70
- **Type:** deployment
- **Owner:** Brudea Armand
- **Requirement(s):** 
- **Use case(s):** UC-06, UC-07
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T70-event-form-component`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 23 — Event form component

**Description:** Build the reusable `EventForm` from page `Organization Screens`, frames `Org – Create event` (15:202) and `Org – Edit event` (49:740): sections "Event details" (title, description textarea, location), "Schedule & capacity" (start date, start time, end date, end time, capacity, registration information) and, in create mode, "Tasks" (rows of description + assignee select + remove button + "Add another task").
Right column in create mode: Event lifecycle card and "Before you publish" checklist; in edit mode: the information card and current status card.
Validation: required title, description, location, start and end; end after start (Error state on End time with "End must be after the start"); capacity ≥ 1; in edit mode capacity cannot be below the active registrations (show the backend message as shown in the edit frame).
The assignee select lists members from `GET /api/organizations/{organizationId}/members`.
Backend errors: the form accepts `fieldErrors` with nested paths (e.g. `tasks[1].description`, `tasks[2].assigneeId`) and shows each message on the exact task row, plus a short error summary at the top of the form listing everything that must be corrected. Form values must never be lost after a failed submit.
Props: `mode`, `initialValues`, `onSubmit`, `submitting`, `fieldErrors`.
**Depends on:** 4, 5, 21.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

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


