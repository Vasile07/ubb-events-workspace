# T75 - Event registrations tab

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T75
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** `REG-05` — Members view registrations; `ORG-05` — Restrict organization actions by role
- **Use case(s):** UC-12
- **Workflow:** View event registrations (UC-12)
- **Drive doc / ADR:** [Project requirements, use cases, workflows, API specification, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi) — Task 35, UC-12 “View event registrations”
- **Repo(s):** frontend
- **Branch:** `T75-event-registrations-tab`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/34
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-registrations-tab.cy.ts`; frontend `npm run typecheck`, `npm run lint`, and `npm run build`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 28 — Event registrations tab

**Description:** Implement page `Organization Screens`, frame `Org – Event: Registrations` (15:589): two stat tiles (active registrations / capacity, places remaining), the info banner and the participants table (avatar, full name, email, registered at, status).
Data from `GET /api/organizations/{organizationId}/events/{eventId}/registrations` (members and managers; a 403 shows the 403 page). Canceled registrations are not listed.
If the backend does not return a registration timestamp, remove that column and tell the planner.
Add search by name/email on the client, an empty state ("No registrations yet", Users icon) and loading/error states.
**Depends on:** T72 — Event workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` routes `/org/:organizationId/events/:eventId/registrations` through `EventWorkspace` to a placeholder page. `src/features/events/EventWorkspace.tsx` fetches event details and renders the shared event header/tabs. `src/features/events/api.ts` contains existing event and current-user registration API calls; `src/features/events/types.ts` defines registration types. `src/features/events/MyRegistrationsPage.tsx` and `cypress/e2e/my-registrations-screen.cy.ts` provide API/query/state and accessible list patterns. `src/shared/ui/FeedbackComponents.tsx` contains `EmptyState` with a Users icon, `Skeleton`, and error/retry components; shared table components are in `src/shared/ui/LayoutComponents.tsx`.
- **Related completed tasks:** T72 — Event workspace shell (`changes/completed/T72-event-workspace-shell.md`), which establishes the shared event route wrapper and is marked merged.
- **Constraints / assumptions:** Frontend-only task, branch metadata is `T75-event-registrations-tab`. The frontend was clean on `master` tracking `origin/master`; fetch/pull succeeded before branch creation and reported `Already up to date`. The stated endpoint is organization-scoped and 403 must use the existing 403 page pattern. Backend DTO `EventRegistrationResponse` currently returns only `firstName`, `lastName`, and `email`; the timestamp column is therefore removed as the Description directs. The response also lacks avatar and per-row status fields; canceled rows are omitted by the endpoint. Existing frontend commands in `package.json` include `npm run typecheck`, `npm run lint`, `npm run build`, and `npm run cypress:run`; use the Vite/Cypress instructions in `AGENTS.md`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement a route-scoped registrations page inside the existing T72 `EventWorkspace`. Add a typed organization-event registrations query, derive the active/capacity and remaining-place statistics from the event detail, and render the organization-screen table with client-side name/email filtering. Reuse the existing auth/error, loading, empty-state, icon, table, and feedback components so the page remains presentation-only and the backend remains the source of authorization and registration status.

The verified backend `EventRegistrationResponse` contains only `firstName`, `lastName`, and `email`; it has no registration timestamp, avatar, status, or registration id. The UI therefore removes the `Registered at` column as required by the task, uses initials for the avatar, and treats returned rows as active registrations. Per the user's correction, canceled-registration counts are not needed and the canceled-count tile is excluded from this task.

### Steps

1. **Contract and API layer:** Confirm the organization-scoped endpoint and existing `403`/`404` error mapping. Add `getEventRegistrations(organizationId, eventId)` and a frontend type matching the three-field backend response. Reuse the event-detail data already loaded by `EventWorkspace` for capacity and active registration count.
2. **Tests first:** Add a focused Cypress spec and minimal compile stubs for the registrations route. Capture behavioral red tests for the page/table, statistics, search, empty state, loading/error/retry, and forbidden response before implementing the UI.
3. **Presentation layer:** Create the registrations page beneath `EventWorkspace`. Fetch registrations with a stable query key, use event detail plus returned active registrations for the active/capacity and remaining-place stat tiles, show loading and retryable error states, render the info banner, and render the accessible participant table. Use initials as the avatar fallback, omit `Registered at` because the API does not return it, and label returned rows as active because the endpoint excludes canceled registrations.
4. **Client interaction and states:** Add case-insensitive search over full name and email, preserve the empty state text `No registrations yet` with the Users icon when the unfiltered or filtered result is empty, and ensure narrow layouts, keyboard focus, table headers, and status/error announcements are accessible. Render the existing 403 page for status 403 and a retry action for other failures.
5. **Verification:** Run the focused Cypress spec to record red evidence, implement until green, then run `npm run typecheck`, `npm run lint`, `npm run build`, and the wider available Cypress suite. Record exact commands and outcomes in the Test log.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/api.ts` — organization-event registrations request.
- `repos/ubb-events-frontend/src/features/events/types.ts` — registration response type matching the backend contract.
- `repos/ubb-events-frontend/src/features/events/EventRegistrationsPage.tsx` (or the established events feature location) — query, statistics, search, table, and UI states.
- `repos/ubb-events-frontend/src/App.tsx` — replace the registrations placeholder while preserving T72 guards and workspace wrapper.
- `repos/ubb-events-frontend/src/styles/global.css` — scoped statistics, banner, search, table, empty/loading/error, and responsive styles.
- `repos/ubb-events-frontend/cypress/e2e/event-registrations-tab.cy.ts` — focused API, rendering, search, state, error, and accessibility coverage.

### Tests to write

- **Acceptance criterion 1:** stub `GET /api/organizations/3/events/42/registrations` and event detail; assert the registrations route renders the info banner, active/capacity and remaining-place tiles, participant avatar initials, full name, and email. Assert the `Registered at` column is absent because the backend response has no timestamp.
- **Acceptance criterion 1/API authorization:** assert the exact organization-scoped URL, a successful member/manager response, and a 403 response rendering the existing forbidden page. Ensure canceled registrations are not rendered when the backend omits them.
- **Acceptance criterion 2:** visit through the merged T72 event workspace route and assert the shared header/tabs and existing organization auth guard remain intact.
- **Acceptance criterion 3:** assert loading, server-error plus retry, empty list with `No registrations yet` and Users icon, no-match search result, case-insensitive name/email search, semantic table headers, keyboard focus, and a usable narrow viewport.

### Risks / open questions

- **Confirmed:** the backend DTO and endpoint currently return only first name, last name, and email. There is no registration timestamp, avatar URL, or per-row status. The timestamp column must be removed and rows can be labeled active because the endpoint excludes canceled registrations.
- The canceled-count tile is explicitly out of scope per the user's correction; no canceled-count API contract is needed.
- The task is frontend-only; adding a backend DTO field/endpoint would require a separate approved backend task or scope change.
- The frontend's `vite.config.ts` sets port 3000, while the general workspace instructions name 5173; Cypress verification used an explicit `baseUrl` override for the repository's configured port.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-registrations-tab.cy.ts` (two focused Cypress scenarios covering rendering/search, empty/loading/error/403 states).
- **Result:** `npx cypress run --browser electron --spec cypress/e2e/event-registrations-tab.cy.ts --config baseUrl=http://127.0.0.1:3000 --reporter spec` — 0 passing, 2 failing. `renders stats, participants, search, and omits unavailable timestamp` could not find `[data-testid=event-registrations-page]`; `shows empty, loading, retryable error, and forbidden states` could not find `[data-testid=event-registrations-empty]`.
- **Failure reason:** The tests reached the running app and failed because the registrations page and its states are still the existing placeholder route. This is behavioral red evidence for the missing T75 functionality, not a compilation or Cypress setup failure.

### Green (after implementation)

- **Result:** Focused `npx cypress run --browser electron --spec cypress/e2e/event-registrations-tab.cy.ts --config baseUrl=http://127.0.0.1:3000 --reporter spec` — 2 passing, 0 failing. `npm run typecheck`, `npm run lint`, and `npm run build` — all passed.
- **Full suite of the affected repo:** `npx cypress run --browser electron --config baseUrl=http://127.0.0.1:3000 --reporter spec` — 31 specs, 183 tests: 180 passing, 3 failing. Initial failures were in unchanged `organizations-list.cy.ts` (2 hidden-span visibility assertions) and `routing-and-guards.cy.ts` (1 route-map assertion).
- **Follow-up after fixing the route marker:** `npx cypress run --browser electron --spec "cypress/e2e/routing-and-guards.cy.ts,cypress/e2e/organizations-list.cy.ts" --config baseUrl=http://127.0.0.1:3000 --reporter spec` — routing/guards 10/10 passed, including the route-map assertion; organizations list 1/3 passed and its same 2 hidden-span visibility assertions failed. `npx cypress run --browser electron --spec cypress/e2e/event-registrations-tab.cy.ts --config baseUrl=http://127.0.0.1:3000 --reporter spec` — 2/2 passed.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “the canceled-count tile should not be part of the plan” | Removed the canceled-count tile from the task description, approach, implementation steps, and tests. | “we do not need such information” |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Reused the T72 event workspace route and organization-scoped registrations endpoint. Active/capacity and remaining places come from the event detail (`registrationCount`, `capacity`); participant rows come from the registrations endpoint. Since the backend returns only names and email, initials are used for avatars, rows are labeled Active, and the Registered at column is omitted. Search is client-side and case-insensitive. Canceled counts are out of scope per the user's correction.
- **Alternatives considered:** A backend response change for timestamps, avatar URLs, row status, or canceled counts was not pursued because this task is frontend-only and those values are not needed for the approved scope. A shared table component was not used because the participant avatar/name cell and row test hooks require a small semantic table here.
- **Limitations:** The route-map assertion initially failed because the new page lacked the expected `page-` test marker; the marker was restored and the routing/guards spec now passes 10/10. Two hidden-span visibility assertions remain failing in unchanged `organizations-list.cy.ts`; their baseline status was not established in this task. The frontend repository's Vite port is 3000, so Cypress verification used an explicit `baseUrl` override. The backend does not provide registration timestamps, avatar URLs, or row status beyond the endpoint's active-only semantics.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — frontend `194786c`
- [x] PR created — [Frontend PR #34](https://github.com/Vasile07/ubb-events-frontend/pull/34)
- [x] PR reviewed
- [x] **Gate 3:** merged by user — Frontend PR #34
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`


