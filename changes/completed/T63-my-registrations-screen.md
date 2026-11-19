# T63 - My registrations screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T63
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** Source task 16 — My registrations screen (no separate requirement ID is recorded).
- **Use case(s):** UC-18
- **Workflow:** View the signed-in user's persisted event registrations by lifecycle and cancel an active registration; organizer-canceled events remain visible, while participant-canceled registrations are deleted and omitted.
- **Drive doc / ADR:** [Project requirements, use cases, architecture, and API documentation](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban) (general project reference; no T63-specific section is recorded).
- **Repo(s):** frontend
- **Branch:** `T63-my-registrations-screen`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/33 (reviewed and merged by user).
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/my-registrations-screen.cy.ts`; `npm run typecheck`; `npm run lint`; `npm run build`; focused and full Cypress suites

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 16 — My registrations screen

**Description:** Implement page `Student Screens`, frame `My registrations` (14:355): pill tabs Active / Canceled / Past with counts, info banner and rows (date block, title, organization, location, date/time, status badge, actions).
Data from `GET /api/users/me/registrations` (only the user's own persisted registrations). The Canceled tab shows registrations for events that were canceled by their organizer after the user registered. When the user cancels their own registration, the backend deletes that row, so it does not appear on this page.
Actions:
- View event
- Cancel registration (confirmation, then `DELETE`) on active rows
- No registration mutation actions for organizer-canceled events
- No registration mutation actions on completed events

Empty state per tab, see frame `My registrations – empty` (48:829, EmptyState Ticket with "Discover events"). Loading and error states.
**Depends on:** T53 — Feedback components: toast, empty state, skeleton, error state; T62 — Event details screen (participant).

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] Active, organizer-canceled, and past registrations are grouped into the correct tabs with counts; participant-canceled registrations are absent because the API deletes them.
- [ ] Active registrations can be viewed and canceled after confirmation; organizer-canceled and completed registrations have no registration mutation actions.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` (route currently a placeholder); `src/features/events/api.ts` (event-scoped register/cancel APIs, no my-registrations API yet); `src/features/events/EventDetailsPage.tsx` and `src/features/events/types.ts` (query/mutation patterns and event/registration types); `src/shared/ui/FeedbackComponents.tsx` (empty, loading, error and toast primitives); `cypress/e2e/event-details.cy.ts` (API stubbing and interaction patterns).
- **Related completed tasks:** T53 (shared feedback components; merged PR recorded); T62 (participant event details; merged; PR: https://github.com/Vasile07/ubb-events-frontend/pull/20; its task status/PR metadata is stale).
- **Constraints / assumptions:** Frontend only. T53 and T62 are listed dependencies; task criteria require both prerequisites merged before implementation. User clarified that participant-canceled registrations are deleted and must not appear on this page. The endpoint retains registrations for events later canceled by an organizer. Frontend `master` was clean and up to date after fetch.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Build a dedicated authenticated `MyRegistrationsPage` under the events feature and replace the current `/my-registrations` placeholder. Keep server communication in the events API module, derive tab membership and action availability from the persisted registration plus event lifecycle data, and use the existing React Query, AppShell, feedback components, modal and toast patterns established by T53/T62. The API contract is `GET /api/users/me/registrations`, returning `{ registrationId, event }` entries with the public event summary; participant cancellation deletes the entry, while organizer cancellation leaves it available for the Canceled tab.

### Steps

1. **UI/API types and client (frontend):** Define the `RegistrationDetailsResponse` equivalent (`registrationId` plus `EventSummaryResponse`) and add `getMyRegistrations` using `/api/users/me/registrations`. Reuse the existing event cancellation DELETE call for active rows. Keep response mapping in the API/types layer and do not add business rules to the client.
2. **Screen and route (frontend UI):** Implement the page with Active, Canceled and Past pill tabs and counts, the information banner, accessible registration rows (date block, title, organization, location, date/time, lifecycle badge and contextual actions), and links to `/events/:eventId`. Replace the placeholder route with the authenticated page inside `AppShell`.
3. **Lifecycle actions (frontend UI/state):** Add the cancellation confirmation modal and optimistic/query-backed DELETE handling with success and error toasts. Permit cancellation only on active rows; organizer-canceled and completed/past rows have no registration mutation controls. Because the backend deletes participant-canceled registrations, remove the row from the page entirely after a successful cancellation and never render a re-registration action.
4. **Feedback and accessibility (frontend UI):** Add tab-specific empty states with the Ticket illustration and “Discover events” action, loading skeletons, retryable error state, semantic tab/button/link labels, focusable dialogs and live toast feedback. Ensure counts and status text update when a mutation settles.
5. **Verification (frontend):** Write Cypress tests before implementation for API rendering and tab counts, correct Active/Canceled/Past grouping, absence of participant-canceled rows, each tab’s empty state, loading/error/retry states, event navigation, cancellation confirmation and DELETE request, organizer-canceled and completed-event action absence, and accessible roles/labels. Run the focused tests, then typecheck, lint, build and the full Cypress suite.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/MyRegistrationsPage.tsx` — page, tabs, rows, feedback states and mutations.
- `repos/ubb-events-frontend/src/features/events/api.ts` — current-user registrations query and existing cancellation call.
- `repos/ubb-events-frontend/src/features/events/types.ts` — registration list and lifecycle types, once matched to the backend contract.
- `repos/ubb-events-frontend/src/App.tsx` — route the page through the authenticated shell.
- `repos/ubb-events-frontend/src/styles/global.css` — screen-specific layout and responsive styles if existing tokens/components do not cover the frame.
- `repos/ubb-events-frontend/cypress/e2e/my-registrations-screen.cy.ts` — focused API-stubbed end-to-end coverage.

### Tests to write

- `groups API registrations into Active, Canceled and Past with correct counts and row details; omits participant-canceled rows because they are absent from the response`, covering acceptance criterion 1.
- `shows the Ticket empty state and Discover events action independently for each empty tab`, covering the empty-state part of the description and criterion 4.
- `shows loading skeletons and a retryable error state`, covering criterion 4.
- `opens event details from View event`, covering the row action in the description.
- `requires confirmation, sends DELETE for an active registration, removes the row and shows a success toast`, covering criterion 2.
- `renders no registration mutation controls for organizer-canceled and completed/past rows`, covering criterion 2.
- `uses authenticated API data and stubs the dependency-backed feedback components`, covering criterion 3 and the dependency prerequisite.
- `exposes semantic tabs, labels, dialog controls and live feedback`, covering criterion 4’s accessibility requirement and reports API/mutation errors without losing unrelated rows.

### Risks / open questions

- The backend contract currently exposes `RegistrationDetailsResponse(registrationId, event)` and `EventSummaryResponse`; verify the exact serialized field names in the generated API response while writing the frontend fixture.
- “Past” may be represented by the event’s `COMPLETED` status or derived from its end timestamp. Prefer the authoritative status and use the timestamp only if the response contract requires a client-side fallback.
- The Figma plugin exposed the Student Screens page and My registrations empty frame metadata, but the specific My registrations frame could not be fetched after the Starter plan rate limit. Visual details should be checked against the canonical frame before styling.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/my-registrations-screen.cy.ts`. Command: `npm run cypress:run -- --config baseUrl=http://127.0.0.1:3000 --spec cypress/e2e/my-registrations-screen.cy.ts`.
- **Result:** Before implementation, Cypress ran 3 tests and all 3 failed because `/my-registrations` still rendered the placeholder and the planned page selectors were absent.
- **Failure reason:** Behavioral red caused by missing screen and route functionality; Cypress setup and compilation were successful. The command used port 3000 because `vite.config.ts` defines 3000 while `cypress.config.ts` still names 5173.

### Green (after implementation)

- **Result:** Focused Cypress command passed 3/3 tests. `npm run typecheck` passed.
- **Supplemental coverage:** After implementation, expanded `my-registrations-screen.cy.ts` to 6 tests covering stateful row removal and success toast, distinct per-tab empty states, loading skeleton, semantic tab focus/selection and View event navigation. Command: `npm run cypress:run -- --config baseUrl=http://127.0.0.1:3000 --spec cypress/e2e/my-registrations-screen.cy.ts`; result: 6/6 passing. These are post-implementation supplemental checks and do not replace the pre-implementation red evidence above.
- **Full suite of the affected repo:** `npm run lint`, `npm run build`, and `npm run typecheck` pass. Exact full Cypress command: `npm run cypress:run -- --config baseUrl=http://127.0.0.1:3000`. It reached all 30 specs; T63 passed and the route-guard regression check passed on rerun (`routing-and-guards.cy.ts`, 10/10). Two failures remained in the unrelated `organizations-list.cy.ts` spec and were reproduced by `npm run cypress:run -- --config baseUrl=http://127.0.0.1:3000 --spec cypress/e2e/organizations-list.cy.ts`: `renders cards, membership actions, and filters by name` and `handles loading and no-match states`. Both failed at the existing assertion expecting a generic page marker `<span>` to be visible (`organizations-list.cy.ts:41` and `:60`); the failing page is the organizations list and neither test exercises T63 code.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | I just verified the endpoint `/api/users/me/registrations` and it includes the registrations to a canceled event (that was published), if the registration was done before the event cancellation. | Updated the context to distinguish registrations for event-canceled events from participant-canceled registrations. | The user clarified the endpoint retains registrations when an event is canceled after registration. |
| 2 | plan | When a user cancels the registration to an event, that registration won't be visible in the endpoint's response, as we delete the row from db. Use the endpoint as it is, we do not include canceled registrations on this page. | Remove participant-canceled registration rows and re-registration from the proposed screen behavior. The revised plan must use the existing endpoint response. | User clarified that participant-canceled registration rows are deleted and should not appear on this page. |
| 3 | plan | Keep it for organizer-canceled events | Define the Canceled tab as registrations to events canceled by their organizer. | User clarified which canceled items the endpoint and tab should include. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Group persisted registrations by lifecycle: organizer-canceled events are Canceled, ended events are Past, and other eligible registrations are Active. Participant-canceled rows are deleted and disappear from the page. Cancellation uses the existing confirmation modal and DELETE endpoint with query invalidation and toasts.
- **Alternatives considered:** Retaining participant-canceled rows or adding re-registration was rejected because the approved backend contract deletes those rows and exposes no history. A backend schema change is outside this frontend task.
- **Limitations:** Lifecycle grouping depends on event status/date fields in the endpoint response. The Figma plugin exposed the canonical pages and empty-state frame, but a Starter-plan rate limit blocked fetching the main frame’s design context. The full Cypress suite still has two reproduced failures in the unrelated organizations list spec, documented in the Test log.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`










