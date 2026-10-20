# T62 - Event details screen (participant)

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T62
- **Type:** deployment
- **Owner:** 
- **Requirement(s):** 
- **Use case(s):** UC-10, UC-11, UC-15, UC-18
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T62-event-details-screen-participant`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 15 — Event details screen (participant)

**Description:** Implement page `Student Screens`, frames `Event details` (13:417) and `Event details – registered` (13:538) as one route with two states.
Left: hero (status badge, organization name, title), info tiles for Date & time and Location only, "About this event" with the registration information block.
Right: registration card (places left, progress bar, "X of Y places taken", action) and the organizer card. Capacity appears only in the right card; there is no deadline and no registered-on date.
Actions:
- Register `POST /api/events/{id}/registrations`
- Cancel registration `DELETE /api/events/{id}/registrations/me` after a confirmation modal

Both update the UI optimistically and invalidate the event, my-registrations and discover queries; show success/error toasts.
Handle 409 (already registered, show the current status), full, started, completed and canceled events by disabling the button with an explanatory text.
When registered, also render the Announcements card from `GET /api/events/{id}/announcements` (hide on 403, empty state when none).
**Depends on:** 5, 6, 7, 10.

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


