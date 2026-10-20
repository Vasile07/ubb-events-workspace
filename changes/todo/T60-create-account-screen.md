# T60 - Create account screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T60
- **Type:** deployment
- **Owner:** 
- **Requirement(s):** 
- **Use case(s):** UC-01
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T60-create-account-screen`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 13 — Create account screen

**Description:** Implement page `Student Screens`, frame `Create account` (13:62) using `AuthLayout`: **First name** and **Last name** side by side, email, password (helper "Min. 8 characters with upper & lower case, a number and a symbol"), the "Create account" button and a link to sign in (`/login`; it is only a route path, the login page does not need to exist yet).
Request: `POST /api/auth/register` with `firstName`, `lastName`, `email`, `password`.
Client validation:
- first name, last name and email required (trimmed), valid email format
- password: minimum 8 characters, at least 1 lowercase letter, 1 uppercase letter, 1 number and 1 special character. Implement it as a pure `validatePassword` helper returning the list of unmet rules (unit tested) and show which rules are missing in the field's error text.

Show the Error state on the email field for "email already registered" (409, as drawn in the frame), map other validation errors from `ApiError.fieldErrors` to the matching fields (including `firstName`, `lastName`, `password`) and use a toast for network errors.
On success sign the user in (or redirect to `/login`) and go to `/events`. Disable the button with a loading state while submitting.
**Depends on:** 4, 5, 8, 9.

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


