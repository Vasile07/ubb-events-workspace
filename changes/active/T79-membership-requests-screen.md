# T79 - Membership requests screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T79
- **Type:** workflow
- **Owner:** Karina Kentsch
- **Requirement(s):** 
- **Use case(s):** UC-05
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T79-membership-requests-screen`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 32 — Membership requests screen

**Description:** Implement page `Organization Screens`, frame `Org – Membership requests` (16:734): pill tabs Pending / Accepted / Declined with counts, table (applicant avatar, full name and email, requested date, status badge, Accept and Decline buttons on pending rows) and the info banner.
Data from `GET /api/organizations/{organizationId}/membership-requests?status=...`; actions with `PATCH .../membership-requests/{requestId}` (accepted | declined) with per-row loading state.
After an action update the row/tab counts, the sidebar pending badge and the members query, and show a toast.
Managers only (guard from task 9). Empty state per tab ("No pending requests") and error state.
**Depends on:** T68 — Organization workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/features/organizations/OrganizationWorkspace.tsx` loads manager-only pending requests and owns the workspace/sidebar badge; `src/features/organizations/membershipRequestQueries.ts` centralizes its query key/invalidation; `src/shared/api/organizations.ts` defines organization and membership-request calls; `src/shared/api/types.ts` has a loose request response type; `src/App.tsx` maps `/org/:organizationId/requests` to a placeholder through the manager guard. Toast/query mutation patterns exist in `src/features/events/EventWorkspace.tsx` and `src/features/organizations/OrganizationDetailsPage.tsx`.
- **Related completed tasks:** T68 (organization workspace shell; merged and present on frontend master); T15 (GET requests contract); T16 (PATCH decision contract).
- **Constraints / assumptions:** Frontend-only repo per task metadata. T15 describes request fields `membershipId`, `userId`, `role`, `status`, with status filtering defaulting to `PENDING` and explicit persisted statuses `PENDING`/`ACTIVE`. T16 accepts lowercase `accepted`/`declined`; accepted requests become `ACTIVE`, declined request rows are deleted. The current T68 sidebar request query returns pending requests and already has an invalidation seam. User clarification is needed because the requested Accepted/Declined tabs, applicant identity/contact/date/avatar, and post-decline state are not all represented by those backend contracts.

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

- [x] Task read and context gathered
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


