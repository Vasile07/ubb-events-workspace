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
- **Tests:** `../../repos/ubb-events-frontend/cypress/e2e/membership-requests.cy.ts`; frontend typecheck, lint, and production build

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 32 — Membership requests screen

**Description:** Implement page `Organization Screens`, frame `Org – Membership requests` (16:734): pill tabs Pending / Accepted / Declined with counts, table (applicant avatar, full name and email, requested date, status badge, Accept and Decline buttons on pending rows) and the info banner.
Data from the detailed endpoint `GET /api/organizations/{organizationId}/membership-requests/details?status=...`; actions with `PATCH .../membership-requests/{requestId}` (`ACCEPTED` | `DECLINED`) with per-row loading state.
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

- **Relevant code:** `../../repos/ubb-events-frontend/src/features/organizations/OrganizationWorkspace.tsx` loads manager-only pending requests and owns the workspace/sidebar badge; `src/features/organizations/membershipRequestQueries.ts` centralizes its query key/invalidation; `src/shared/api/organizations.ts` defines organization and membership-request calls; `src/shared/api/types.ts` has a loose request response type; `src/App.tsx` maps `/org/:organizationId/requests` to a placeholder through the manager guard. Toast/query mutation patterns exist in `src/features/events/EventWorkspace.tsx` and `src/features/organizations/OrganizationDetailsPage.tsx`.
- **Related completed tasks:** T68 (organization workspace shell; merged and present on frontend master); T15 (GET requests contract); T16 (PATCH decision contract).
- **Constraints / assumptions:** Frontend-only repo per task metadata. The backend now provides manager-only `GET /api/organizations/{organizationId}/membership-requests/details?status=PENDING|ACCEPTED|DECLINED`, returning `requestId`, `firstName`, `lastName`, `email`, `requestedAt`, and detailed status. Accepted/declined history is retained separately; the existing PATCH decision endpoint still takes uppercase `ACCEPTED`/`DECLINED` enum values in its JSON body. No avatar URL is included, so use initials/neutral avatar treatment. Frontend currently has no typed wrapper for the detailed endpoint or decision PATCH. T68 sidebar uses the simpler membership-requests query; its pending count must remain compatible and refresh after decisions.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the manager-only requests route as a data-driven screen inside the existing T68 organization workspace. Add typed frontend API functions for the detailed request list and decision mutation, then use TanStack Query to load one status tab at a time, keep counts in sync, and invalidate the workspace pending badge and organization members after a successful decision. Render the Figma frame as an accessible tablist and table with initials avatars, explicit loading/error/empty states, per-row mutation loading, and success/error toasts. Keep authorization in the existing route guard and leave backend business rules unchanged.

### Steps

1. **API/types:** Define the detailed request status and response types (`requestId`, applicant name/email, nullable `requestedAt`, status) and add `getMembershipRequestDetails(organizationId, status)` plus `decideMembershipRequest(organizationId, requestId, decision)` wrappers. Serialize the decision as the backend enum values `ACCEPTED` and `DECLINED`; keep the existing simple pending-list function/query key used by T68.
2. **Screen data flow:** Create the membership requests page for `/org/:organizationId/requests`, parse and validate the route id, track the active Pending/Accepted/Declined tab, and query the detailed endpoint for all three statuses so every tab count is available. Expose retryable loading/error feedback and the required per-tab empty copy.
3. **UI:** Reproduce frame `Org – Membership requests` (16:734) within the existing workspace main area: title and info banner, pill tabs with counts, semantic table headers, initials avatar fallback, applicant name/email, formatted requested date with a safe fallback for null, status badge, and Accept/Decline controls only for pending rows. Add accessible labels, keyboard-focusable tabs/buttons, and responsive table behavior using existing design tokens/components.
4. **Mutations and refresh:** Track the request id currently being accepted or declined so only that row enters a loading/disabled state. On success invalidate the detailed queries, the existing sidebar pending-request query, and the organization members query; update visible counts through refetch and show a success toast. On failure preserve the current rows/counts and show an error toast. Prevent duplicate submissions while a row mutation is active.
5. **Routing and guard:** Replace the `/org/:organizationId/requests` placeholder with the new page while retaining `RequireAuth`, `RequireOrgManager`, `AppShell`, and `OrganizationWorkspace`. Confirm non-manager access remains blocked and non-organization routes are unchanged.
6. **Verification:** Write Cypress tests before implementation for API URLs/status parameters, manager rendering, all tabs and counts, table fields, empty/loading/error states, decision payloads, per-row loading, toast behavior, query refresh, sidebar badge refresh, members refresh, keyboard/accessibility behavior, and manager guard regression. Run the focused spec plus frontend typecheck, lint, build, and affected/full Cypress suites as available; record exact red and green evidence in this task file.

### Files to create / modify

- `../../repos/ubb-events-frontend/src/shared/api/types.ts` — detailed request status/response and decision types.
- `../../repos/ubb-events-frontend/src/shared/api/organizations.ts` — detailed-list and decision API wrappers.
- `../../repos/ubb-events-frontend/src/features/organizations/MembershipRequestsPage.tsx` — tabbed manager screen, table, states, mutations, and refresh behavior.
- `../../repos/ubb-events-frontend/src/features/organizations/membershipRequestQueries.ts` — shared detailed query keys/invalidation helpers, preserving the existing pending-list key.
- `../../repos/ubb-events-frontend/src/App.tsx` — route the manager-protected requests path to the new page.
- `../../repos/ubb-events-frontend/src/styles/global.css` (or the existing organization stylesheet) — screen/table/tab/banner/responsive styles using project tokens.
- `../../repos/ubb-events-frontend/cypress/e2e/membership-requests.cy.ts` — focused API, UI, state, mutation, refresh, and accessibility coverage.

### Tests to write

- **Acceptance criterion 1 (behavior/API):** Cypress stubs the detailed endpoint for each status and verifies exact organization-scoped URLs, status query values, rendered applicant fields/date/status, tab counts, and accepted/declined PATCH payloads.
- **Acceptance criterion 2 (dependency/authorization):** Cypress verifies the screen is available through the merged T68 workspace and manager guard, while a member is redirected/forbidden and does not issue detailed-request calls.
- **Acceptance criterion 3 (states/accessibility):** Cypress covers loading, retryable list error, per-tab empty state, nullable date fallback, row-level mutation loading, failed mutation preservation, semantic tab/table landmarks, accessible names, keyboard tab/button use, and visible focus.
- **Refresh behavior:** Cypress verifies successful acceptance and decline invalidate/refetch the detailed lists, update counts and the T68 sidebar pending badge, refresh organization members, and show a toast; failed decisions leave the existing data intact.

### Risks / open questions

- The imported source description named the simple membership-requests list URL and lowercase decisions; implementation follows the new detailed endpoint and uppercase decision enum contract supplied by the backend.
- `requestedAt` is nullable in the backend DTO; display an em dash when it is absent.
- No avatar URL is provided, so derive deterministic initials from first/last name and use the shared avatar styling rather than inventing image data.
- The detail endpoint has its own `MembershipRequestDetailsStatus` values, while the sidebar still uses the simpler pending-list endpoint. Query keys and invalidation must remain distinct but be invalidated together after decisions.
- Load all three status queries to provide counts on every tab; each query needs its own error/loading treatment so a single failed tab does not hide the others.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/membership-requests.cy.ts` covers loading the detailed status queries, rendering applicant details, decision payload/loading/toast, failed mutation feedback, empty/error states, and manager-only access.
- **Command:** `$env:CYPRESS_CACHE_FOLDER = (Join-Path (Get-Location) 'node_modules/.cache/cypress-t79'); npx cypress run --browser electron --spec cypress/e2e/membership-requests.cy.ts --reporter spec`
- **Result:** Before product implementation, Cypress reached the app and reported **1 passing, 4 failing**. The passing test was `does not expose the screen to a plain member`. The failing tests were `loads all status URLs, counts tabs, and renders applicant details`, `sends a decision payload, shows row loading, and refreshes with a success toast`, `preserves rows and shows an error toast when a decision fails`, and `renders empty and retryable error states`.
- **Failure reason:** The protected requests route still rendered its placeholder, so the page, tabs, row controls, decision behavior, and request states expected by those tests were absent. This is a behavioral failure caused by the planned screen not existing. An earlier runner attempt failed at Cypress startup; that setup failure is not used as red evidence.

### Green (after implementation)

- **Focused command:** `$env:CYPRESS_CACHE_FOLDER = (Join-Path (Get-Location) 'node_modules/.cache/cypress-t79'); npx cypress run --browser electron --spec cypress/e2e/membership-requests.cy.ts --reporter spec`
- **Result:** **7 passing, 0 failing.** Covers all three status queries/counts, keyboard tabs, accept/decline requests and query refresh, failure feedback, empty/error states, and manager-only access.
- **Typecheck:** `npm run typecheck` — passed.
- **Lint:** `npm run lint -- --ignore-pattern '**/.cypress-cache/**'` — passed. The first plain lint run traversed a generated Cypress cache under the repository root; the cache was moved under ignored `node_modules` before the successful lint run.
- **Build:** `npm run build` — passed.
- **After the CSS token correction:** `npm run typecheck`, `npm run lint -- --ignore-pattern '**/.cypress-cache/**'`, and `npm run build` all passed again; `rg -n -- '--text-inverse-soft|--color-white' src/styles/global.css` found no remaining occurrences.
- **Full suite of the affected repo:** `npx cypress run --browser electron --reporter spec` — **187 passing, 3 failing across 32 specs**. T79 passed 7/7. Failures were two tests in `organizations-list.cy.ts` and one stale selector in `routing-and-guards.cy.ts`, which still expected the replaced requests placeholder. The route guard selector was updated to the new page test id after this run; the full suite has not yet been rerun after that test adjustment. Targeted reruns did not produce Cypress output in the current shell, so the remaining organizations-list failures are unverified and could not be attributed to T79.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | implementation | There are errors in `global.css` where `--text-inverse-soft` and `--color-white` cannot be resolved. | Replaced those undeclared custom properties with the defined `--color-blue-200` and `--bg-surface` tokens. | Use existing theme tokens so CSS tooling can resolve the values. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use the manager-only detailed endpoint for the three status tabs. Keep T68's lightweight pending query as a separate source for its sidebar badge, and invalidate both query families plus organization members after a successful decision. Use applicant initials because the response does not include an avatar URL; show an em dash for a missing request date.
- **Alternatives considered:** Loading only the selected tab would reduce requests but would leave the counts unavailable on the other tabs, so the page loads all three status queries to show counts consistently.
- **Limitations:** The full Cypress suite had three failures, including two in the unrelated organizations list; those two could not be reproduced with a targeted rerun because Cypress exited without reporting output. The full suite needs a successful rerun after the route guard selector update. The task has not been reviewed by the user and has not passed Gate 2.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [ ] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [ ] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``


