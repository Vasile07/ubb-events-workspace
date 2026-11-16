# T71 - Create event flow

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T71
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** N/A (not provided)
- **Use case(s):** UC-06
- **Workflow:** N/A (not provided)
- **Drive doc / ADR:** N/A (not provided)
- **Repo(s):** frontend
- **Branch:** `T71-create-event-flow`
- **PR link(s):** [Frontend PR #24](https://github.com/Vasile07/ubb-events-frontend/pull/24)
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/create-event-flow.cy.ts`; frontend `typecheck`, `lint`, `build`, and Cypress suites

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 24 — Create event flow

**Description:** Wire `EventForm` (create mode) to `/org/:organizationId/events/new` (managers only).
On submit call `POST /api/organizations/{organizationId}/events` once, sending the event data together with the list of tasks. The endpoint is atomic: either the event and all its tasks are created, or nothing is created and the backend returns the list of errors. There is no partial state and no retry mechanism: on error keep the form as it is, show every returned error on the matching field (event fields and task rows, using task 23), let the user correct them and submit again.
Buttons: "Save as draft" (primary) and Cancel (confirmation modal when the form is dirty).
On success the event is a DRAFT: show a toast and redirect to the event workspace (Tasks tab) where it can be published.
**Depends on:** T70 — Event form component.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are present on the T71 base branch before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` already declares `/org/:organizationId/events/new`, and `src/features/organizations/OrganizationEventsPage.tsx` links managers to it; inspect its route element for remaining placeholder behavior. T70 supplies `src/features/events/EventForm.tsx`, `eventFormTypes.ts`, `eventFormValidation.ts`, event API helpers, and task row field errors. `src/shared/api/client.ts` contains the API request/error handling pattern. `changes/completed/T23-create-an-event-with-tasks.md` records the related create-with-tasks requirement.
- **Related completed tasks:** T23 — Create an event with tasks.
- **Constraints / assumptions:** Frontend-only; use the existing form from T70 and keep create-flow API wiring in the route/workflow layer. T70 is in active Frontend PR #23 and is not merged yet; the user confirmed its current branch already contains the needed code and authorized implementation before merge. Create `T71-create-event-flow` from `T70-event-form-component`. The frontend repo is clean on that base branch and has no unpushed commits.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Replace the manager-only placeholder at `/org/:organizationId/events/new` with a create-page container that composes T70's `EventForm` and owns the atomic create mutation. Add a typed POST helper, map `ApiError.fieldErrors` into nested form errors, preserve values on failure, and navigate to the returned DRAFT event's Tasks workspace with a success toast. Keep authorization in the existing manager guard and presentation/validation in `EventForm`.

### Steps

1. **Route/UI:** Add a create-event page and replace the route placeholder while retaining auth and manager guards; pass the organization id and initial draft to `EventForm`.
2. **API:** Add a typed `createOrganizationEvent` helper for one `POST /api/organizations/{organizationId}/events` request containing event fields and ordered tasks, using `apiFetch` and the T23 response contract (returned id and DRAFT status).
3. **Submit/errors:** Wire the mutation to `EventForm.onSubmit`; disable duplicate submits, map all backend event and `tasks[index].*` errors, show a safe top-level error when no field details exist, and retain the draft on failure.
4. **Cancel/success:** Track dirty state, confirm before abandoning dirty values, and on 201 show a toast then navigate to `/org/{organizationId}/events/{eventId}/tasks`.
5. **States/accessibility:** Cover loading/submitting, server errors, modal focus/keyboard, toast status, labels and disabled actions. Do not add retry or partial-save behavior.
6. **Verification:** Write Cypress tests first, capture behavioral red evidence, then run focused and full frontend checks (`typecheck`, `lint`, `build`, Cypress) and record results.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/CreateEventPage.tsx` — route container, mutation, dirty/cancel modal, toast and redirect.
- `repos/ubb-events-frontend/src/features/events/api.ts` and/or `types.ts` — typed create request/response and POST helper.
- `repos/ubb-events-frontend/src/App.tsx` — replace the new-event manager placeholder.
- Existing shared modal/toast styles only if the current primitives cannot support the confirmation state.
- `repos/ubb-events-frontend/cypress/e2e/create-event-flow.cy.ts` — focused route/API/state/accessibility tests.

### Tests to write

- Cypress verifies manager-only access, complete form sections, exact single POST payload with ordered tasks, submitting disabled state, and successful DRAFT toast plus Tasks-tab redirect.
- Cypress stubs atomic 400 responses with event and nested task errors, verifies every matching field and summary entry, retains all entered values, and allows correction/resubmission.
- Cypress covers clean and dirty Cancel behavior, confirmation keyboard/focus handling, malformed/unauthorized/server-error states, and accessible labels/relationships.
- Write these tests before page/API wiring, then record the failing focused run as valid red evidence and rerun focused plus wider checks after implementation.

### Risks / open questions

- The request uses the direct event object shape documented in T23 (`date`, `startTime`, `endTime`, `capacity`, ordered `tasks`). The backend DTO/OpenAPI was not available in this frontend checkout, so verify the field names against it during review or integration.
- T70 is in active PR #23 and remains unmerged, but the user confirmed its current branch contains the dependency code and authorized proceeding from it. Create the T71 branch from `T70-event-form-component`.
- The T70 form now exposes optional dirty-state and submit-label props for the create route; task add/remove also marks the form dirty.
- Reuse existing toast, modal, auth guards, API error mapping, and event workspace route; no retry or optimistic partial state.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/create-event-flow.cy.ts` (2 tests).
- **Result:** `$env:CYPRESS_CACHE_FOLDER='C:\\Users\\Teofan\\AppData\\Local\\Cypress\\Cache'; npx cypress run --spec cypress/e2e/create-event-flow.cy.ts --reporter spec --browser electron` — 0 passing, 2 failing.
- **Failure reason:** Both tests reached the manager-only route but timed out because `[data-testid="event-form"]` and `[name="title"]` were absent; the route still rendered the placeholder. This was a behavioral failure caused by the missing create-flow implementation. The initial run was blocked by Cypress cache permissions and was rerun with the approved escalation path.

### Green (after implementation)

- **Result:** Focused `create-event-flow.cy.ts` passed 4/4 after implementing the route container, POST helper, draft redirect, error mapping, value preservation, dirty-cancel confirmation, task add/remove dirty tracking, and dialog focus/Escape behavior. `npm run typecheck` and `npm run lint` passed after the review fixes; the earlier `npm run build` also passed.
- **Full suite of the affected repo:** Final run `npx cypress run --reporter spec --browser electron` completed 24 specs / 136 tests: 134 passing and 2 failing. `routing-and-guards.cy.ts` passed 10/10 after the route-map update. The remaining unrelated failures are `organizations-list.cy.ts` — “renders cards, membership actions, and filters by name” (line 41 visibility assertion) and “handles loading and no-match states” (line 60 visibility assertion). The new create-event spec passed 4/4.
- **Post-merge check:** After merging updated `origin/master` (including T70 PR #23) into T71, `npm run typecheck`, `npm run lint`, and `npm run build` passed; `create-event-flow.cy.ts` passed 4/4 and `routing-and-guards.cy.ts` passed 10/10.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “The task's stated dependency prerequisites are merged before implementation.” | Changed the criterion to require the dependency code on the T71 base branch; updated the context and risk to use `T70-event-form-component` before merge. | “I think we can move to implementation and branch creation, even before the T70 merge. we can make it on the current code which contains the T70 changes that are not merged to master yet (Made by me). after implementation we create the branch from the t70 branch.” |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** The route container owns the atomic POST, server error mapping, dirty cancel confirmation, toast, and redirect; `EventForm` remains responsible for draft state and validation. The request maps the form's start date to the backend event `date` and sends numeric capacity and task assignee ids.
- **Alternatives considered:** Keeping the existing placeholder or placing API mutation logic inside `EventForm` was rejected to preserve the form's reusable presentation contract and route-level workflow ownership.
- **Limitations:** The exact backend DTO was not available in the frontend repository; the request follows the T23 documented direct event object contract (`date`, `startTime`, `endTime`, `capacity`, and ordered `tasks`).

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


