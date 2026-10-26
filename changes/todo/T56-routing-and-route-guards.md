# T56 - Routing and route guards

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T56
- **Type:** deployment
- **Owner:** Brudea Armand
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T56-routing-and-route-guards`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 9 — Routing and route guards

**Description:** Define the route map.
- Public: `/login`, `/register`.
- Authenticated: `/events`, `/events/:eventId`, `/my-registrations`, `/my-tasks`, `/organizations`, `/organizations/:organizationId`, `/profile`.
- Workspace (members only): `/org/:organizationId/events`, `/org/:organizationId/events/new`, `/org/:organizationId/events/:eventId/edit`, `/org/:organizationId/events/:eventId/{tasks|registrations|chat|announcements|follow-up}`, `/org/:organizationId/requests`, `/org/:organizationId/members`.

Guards:
- `RequireAuth` (redirect to `/login` remembering the target)
- `RedirectIfAuthenticated`
- `RequireOrgMember` and `RequireOrgManager` (render the 403 page when the role is insufficient; manager-only: create/edit/publish/delete event, membership requests, promote, follow-up).

Default redirect to `/events`. Placeholder pages are fine until the screens exist.
**Depends on:** 8.

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


