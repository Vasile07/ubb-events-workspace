# T50 - API client and error handling

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T50
- **Type:** deployment
- **Owner:** 
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T50-api-client-and-error-handling`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 3 — API client and error handling

**Description:** Build the HTTP layer in `shared/api` and a small `shared/lib/tokenStorage` module (`getToken`, `setToken`, `clearToken`, backed by localStorage).
The API client:
- prefixes the base URL, reads the token from `tokenStorage` and adds the `Authorization: Bearer` header, serializes JSON
- parses the backend's shared error format (status, message, list of errors) into a typed `ApiError` exposing `status`, `message` and `fieldErrors` (map field path → message). Field paths must support nested/indexed paths such as `tasks[1].description` so forms can show errors on the exact row
- exposes `setUnauthorizedHandler(handler)`. The client calls the handler on any 401, except for requests to the login and register endpoints (wrong credentials must not trigger a logout). The default handler is a no-op: the client must NOT import anything from `features/auth` or the router. The auth layer registers the handler in task 8, so the dependency goes 8 → 3 only and there is no circular dependency.

Generate or hand-write TypeScript DTO types from the OpenAPI spec for every endpoint used in the app.
Configure TanStack Query defaults (no retry on 4xx, 1 retry on network errors, sensible stale time).
Add tests for error parsing (including nested field paths) and for the 401 handler behavior.
Done when `GET /api/organizations` is called through the client, a forced 400 shows up as a typed `ApiError` and a forced 401 calls a test handler.
**Depends on:** 1.

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


