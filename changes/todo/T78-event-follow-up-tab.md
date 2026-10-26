# T78 - Event follow-up tab

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T78
- **Type:** deployment
- **Owner:** Cernautan Teofan
- **Requirement(s):** 
- **Use case(s):** UC-13
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T78-event-follow-up-tab`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 31 — Event follow-up tab

**Description:** Implement page `Organization Screens`, frame `Org – Event: Follow-up` (16:591): info banner, form with Summary and Feedback textareas and "Save follow-up", plus the "Event recap" card (registered participants, tasks, announcements counts, computed from existing queries).
Read with `GET .../events/{eventId}/follow-up` (404 = none yet, show the empty form) and create with `POST` on the same path (any manager of the organization, only after the event has ended).
Before the event ends show the form in a locked state with an explanation; plain members see the saved follow-up read-only or an empty state. After saving switch to a read view with an "editing is not available" note (no update endpoint) and a success toast.
**Depends on:** 25.

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


