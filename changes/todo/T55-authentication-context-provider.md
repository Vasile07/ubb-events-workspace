# T55 - Authentication context provider

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T55
- **Type:** deployment
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** UC-01, UC-02, UC-03
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T55-authentication-context-provider`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 8 — Authentication context provider

**Description:** Create `AuthProvider` and `useAuth()` exposing:
- `user` (id, firstName, lastName, derived `fullName`, email, memberships with `organizationId`, organization name and role)
- `token` (kept in `shared/lib/tokenStorage`) and `status` (loading | authenticated | anonymous)
- `login(email, password)` → `POST /api/auth/login`
- `register(payload)` → `POST /api/auth/register` with `firstName`, `lastName`, `email`, `password`
- `logout()` (client-side only: clear the token, clear the TanStack Query cache, redirect to `/login`)
- `refreshUser()` → `GET /api/users/me`
- helpers `isMemberOf(organizationId)` and `isManagerOf(organizationId)`.

Restore the session on startup by calling `/users/me` when a token exists; show a full-page loader meanwhile.
Register the API client's unauthorized handler here (`setUnauthorizedHandler` from task 3) so an expired token logs the user out and shows a toast. The auth layer depends on the API client, never the other way around.
Add tests for restore-on-reload, login failure (no logout loop) and logout.
**Depends on:** 1, 3, 6.

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


