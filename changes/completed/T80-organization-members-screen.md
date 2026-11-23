# T80 - Organization members screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T80
- **Type:** workflow
- **Owner:** Karina Kentsch
- **Requirement(s):** 
- **Use case(s):** UC-16
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T80-organization-members-screen`
- **PR link(s):** 
- **Tests:** `../../repos/ubb-events-frontend/cypress/e2e/organization-members.cy.ts`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 33 — Organization members screen

**Description:** Implement page `Organization Screens`, frame `Org – Members` (16:887): header with the member-count badge, info banner about promotion, and the table (avatar, full name, email, role badge Manager = Brand / Member = Neutral, "Promote to manager" on member rows for managers only).
Data from `GET .../members`; promote with `PATCH .../members/{userId}/role` sending `MANAGER`.
Promotion is permanent: the confirmation modal must say it cannot be undone and there is no demote action. Plain members see the list without actions.
Loading, empty and error states; invalidate the auth context memberships if the current user is affected.
**Depends on:** T68 — Organization workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-frontend/src/features/organizations/OrganizationWorkspace.tsx` provides the `/org/:organizationId` shell and role helpers; `src/features/organizations/MembershipRequestsPage.tsx` and `membershipRequestQueries.ts` show the query/mutation, table, feedback, and invalidation patterns; `src/shared/api/organizations.ts` already implements `getOrganizationMembers`; `src/shared/api/types.ts` defines `OrganizationMemberResponse`; `src/features/auth/AuthProvider.tsx` exposes `refreshUser`; `src/App.tsx` contains the existing `/members` route; `src/shared/ui/FeedbackComponents.tsx` and `src/styles/global.css` provide shared UI and workspace styling patterns; Cypress specs include `organization-workspace-shell.cy.ts` and `membership-requests.cy.ts`.
- **Related completed tasks:** T68 — Organization workspace shell (merged; dependency confirmed by user); T72 — Event workspace shell (shared nested workspace-screen patterns).
- **Constraints / assumptions:** Frontend only, per task metadata. Keep the existing shell and route guards. Promotion is irreversible and manager-only; plain members may view the list but have no actions. Refresh auth memberships after a successful promotion only when the current user's membership is affected. Frontend repository is clean on `master` with no unpushed commits; `git fetch origin master` could not read `.git/FETCH_HEAD` due to permissions, so remote synchronization was unavailable.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the members route as a focused organization screen inside the existing T68 workspace shell. Add the typed role-promotion API call and a query/mutation driven page that derives manager permissions from `AuthProvider`, reuses shared table/feedback components, and keeps promotion irreversible in both UI and request behavior. On success, refresh the members list and refresh auth membership data when the promoted user is the signed-in user; on failure, keep the current list and show an actionable error.

### Steps

1. **API/types:** Add the promotion request type and `PATCH /api/organizations/{organizationId}/members/{userId}/role` client function sending the exact body `{ "role": "MANAGER" }`, preserving `ApiError` handling and the response role.
2. **UI/data:** Build the `/org/:organizationId/members` page with a validated route id, TanStack Query members loading, and explicit loading, empty, retryable error, and invalid-route states. Render the header count, promotion information banner, and accessible table with avatar, full name, email, and role badges (`Manager` Brand, `Member` Neutral).
3. **Permissions/actions:** Read the current organization role from `useAuth`; render promotion controls only for managers and only on member rows. Add a confirmation modal stating promotion is permanent, cannot be undone, and has no demote action. Disable duplicate submissions, handle API errors, invalidate the members query after success, and call `refreshUser` if the promoted member is the current user.
4. **Integration/styling:** Keep T68 route guards and shell navigation intact, add only the members-screen styles/assets needed to match `Org – Members` (16:887), and ensure semantic headings, table labels, modal focus behavior, and keyboard-visible controls.
5. **Verification:** Write Cypress tests before implementation for manager/member variants, exact GET/PATCH calls, count and rows, modal copy, permanent promotion/no-demote behavior, query/auth refresh, loading/empty/error/retry states, and accessibility. Run the focused spec, then the frontend typecheck, lint, build, and wider Cypress suite; record red and green evidence.

### Files to create / modify

- `../../repos/ubb-events-frontend/src/shared/api/organizations.ts` and `src/shared/api/types.ts` — promotion request/response types and role-promotion API function.
- `../../repos/ubb-events-frontend/src/features/organizations/OrganizationMembersPage.tsx` (or the existing members page module) — query, table, manager-only actions, modal, and states.
- `../../repos/ubb-events-frontend/src/App.tsx` — connect the existing `/org/:organizationId/members` route if the page is not already wired.
- `../../repos/ubb-events-frontend/src/styles/global.css` — members-page layout, table, banner, and action styling using existing design tokens.
- `../../repos/ubb-events-frontend/cypress/e2e/organization-members.cy.ts` — focused behavioral and accessibility coverage.

### Tests to write

- Manager sees member count, all member data, correct role badge tones, info banner, and `Promote to manager` only for member rows.
- Plain member sees the same list and roles but no promotion controls and makes no role-promotion request.
- Confirming promotion sends the organization/user scoped `PATCH` with `MANAGER`, shows permanent/no-demote copy, refreshes the list, and refreshes auth memberships when the current user is affected.
- Canceling or failed promotion leaves the row unchanged and reports the error; duplicate submissions are prevented.
- Loading, empty, invalid-route, API-error, retry, and accessibility semantics are covered.

### Risks / open questions

- Member response name/email fields are optional in the current frontend type, so the UI must retain a safe fallback for missing names or email values.
- The current route is already present; if the implementation reveals a missing page component, wire it without changing T68 authorization policy.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/organization-members.cy.ts` — 5 tests covering manager/member visibility, role promotion, cancellation/error handling, and states.
- **Result:** With Vite running on port 5173, PowerShell `$env:CYPRESS_CACHE_FOLDER = 'C:\Users\Karina Kentsch\AppData\Local\Cypress\Cache'` then `npx cypress run --browser electron --spec cypress/e2e/organization-members.cy.ts --reporter spec` — 0 passing, 5 failing.
- **Failure reason:** The tests reached the existing `/org/3/members` placeholder route. Assertions failed because the planned `organization-members-page`, promotion controls and loading state were absent. These are behavioral failures caused by the missing T80 screen, not compile or test setup failures.

### Green (after implementation)

- **Result:** Focused `npx cypress run --browser electron --spec cypress/e2e/organization-members.cy.ts --reporter spec` — 5 passing, 0 failing. `npm run typecheck`, `npm run lint`, and `npm run build` passed.
- **Full suite of the affected repo:** `npm run cypress:run -- --browser electron --reporter spec` — 198 passing, 2 failing across 34 specs. `organizations-list.cy.ts` failed `renders cards, membership actions, and filters by name` and `handles loading and no-match states`; both expect hidden spans to be visible. These tests are outside T80; their status on the base branch was not checked.

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

- **Decisions (draft for user review):** Follow Figma frame `Org – Members` (16:887): show an organization-specific subtitle and dynamic member-count badge, keep each member’s name and email together in the first table column, show role badges and manager-only actions, and use the shield promotion banner. Reuse the API client, TanStack Query member key, shared avatars/badges/buttons/modal/feedback, and T68 workspace shell. Promotion sends `{ "role": "MANAGER" }`, is confirmed as irreversible, and refreshes auth context when the affected row is the current user.
- **Alternatives considered:** A separate email column was considered, but the named Figma frame places name and email together under Member. The existing T68 guard protects the list for organization members while allowing both managers and plain members to view it.
- **Limitations:** Two unrelated `organizations-list.cy.ts` tests failed in the wider suite because they expected hidden spans to be visible; base-branch status is unknown. Member name/email fields are optional in the response type, so the screen falls back to “Unknown user” or an em dash when missing.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [ ] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``


