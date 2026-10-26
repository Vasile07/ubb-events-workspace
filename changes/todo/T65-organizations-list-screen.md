# T65 - Organizations list screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T65
- **Type:** deployment
- **Owner:** Karina Kentsch
- **Requirement(s):** 
- **Use case(s):** UC-04
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T65-organizations-list-screen`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 18 — Organizations list screen

**Description:** Implement page `Student Screens`, frame `Organizations` (14:699): header, search field and a 3-column grid of organization cards (icon tile, status badge, name, description, action).
Data from `GET /api/organizations` including the user's status.
Actions:
- MEMBER → "Open organization" (to `/org/:organizationId/events`)
- PENDING → disabled "Waiting for approval"
- NONE → "Request membership" calling `POST /api/organizations/{organizationId}/membership-requests` (no body), then switch the card to Pending without a reload and show a toast; handle an already-pending 409 gracefully.

The search filters by organization name only, on the client. Equal-height cards with the action pinned to the bottom. Loading, empty ("No organizations match") and error states.
**Depends on:** 6, 10.

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


