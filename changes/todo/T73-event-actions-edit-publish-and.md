# T73 - Event actions: edit, publish and delete

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T73
- **Type:** deployment
- **Owner:** Cernautan Teofan
- **Requirement(s):** 
- **Use case(s):** UC-07, UC-08, UC-17
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T73-event-actions-edit-publish-and`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 26 — Event actions: edit, publish and delete

**Description:** Add the manager actions to the event workspace header.
- Edit event → route `/org/:organizationId/events/:eventId/edit` reusing `EventForm` in edit mode (frame `Org – Edit event` 49:740) with `PUT .../events/{eventId}`; disable it for completed/canceled events.
- Publish (DRAFT only): primary button "Publish event" next to Edit and Delete as in frame `Org – Event: Tasks (draft)` (49:897) with the draft warning banner; clicking opens the confirmation modal from frame `Org – Publish event confirmation` (49:1055) and calls `PATCH .../events/{eventId}/publish`; show backend validation messages (missing end date etc.) in the modal.
- Delete event: Danger button opening the modal from frame `Org – Delete event confirmation` (16:1061) that shows the number of affected participants and calls `DELETE .../events/{eventId}` (soft delete; the backend refuses ended events, show that message).

Invalidate event, events list and discover queries and show toasts. Hide all of these for plain members.
**Depends on:** 23, 25.

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


