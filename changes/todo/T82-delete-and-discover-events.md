# T82 - Delete and discover events

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T82
- **Type:** workflow
- **Owner:** 
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** 
- **Repo(s):** backend
- **Branch:** `T82-delete-and-discover-events`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Update backend event deletion and event discovery behavior:

- When deleting a draft event, remove it from the database.
- When deleting a published event, retain it and set its status to `CANCELED`.
- Include canceled events in the results returned by event discovery.
- Include each discovered event's `capacity` and `spotsTaken` (registration count) in the summary response so the frontend can render event capacity and filter events open for registration.

## Acceptance criteria

<!-- Proposed from the user's request; confirm or edit. -->

- [ ] **Proposed:** Deleting a draft event removes it from persistence.
- [ ] **Proposed:** Deleting a published event retains the record with status `CANCELED`.
- [ ] **Proposed:** `GET /api/events` includes canceled events in its results.
- [ ] **Proposed:** Discovery summaries include `capacity` and `spotsTaken` (registration count) for each returned event.
- [ ] **Proposed:** Backend tests cover both deletion branches and discovery of canceled events.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks. Only what is relevant. -->

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
