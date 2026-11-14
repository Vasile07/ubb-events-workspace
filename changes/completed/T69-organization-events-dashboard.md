# T69 - Organization events dashboard

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T69
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** 
- **Use case(s):** none (supports UC-06, UC-07)
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend, backend
- **Branch:** `T69-organization-events-dashboard`
- **PR link(s):** 
- **Tests:** `EventServiceTest`, `EventControllerApiTest`, `cypress/e2e/organization-events-dashboard.cy.ts`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 22 — Organization events dashboard

**Description:** Implement page `Organization Screens`, frame `Org – Events` (15:10): header with "Create event" (managers only), pill tabs All / Drafts / Published / Completed / Canceled with counts, and the table (title + location, date, status badge, registrations "x / capacity" or "—" for drafts, action).
Data from `GET /api/organizations/{organizationId}/events` (includes drafts and canceled).
Row actions:
- Manage (to the event's Tasks tab)
- "Add follow-up" for completed events (managers)
- View for canceled

States:
- loading → frame `Org – Events (loading)` (49:1175, TableSkeleton)
- empty → `Org – Events (empty)` (49:1081, EmptyState Calendar with "Create event" for managers)
- error → ErrorState.

**Depends on:** T52 — Base UI components: layout and data display; T53 — Feedback components: toast, empty state, skeleton, error state; T68 — Organization workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/features/organizations/OrganizationWorkspace.tsx` and `App.tsx` provide the organization shell and route context; `src/features/events/api.ts` and `types.ts` provide the existing event API/type pattern; `src/shared/ui/FeedbackComponents.tsx` and `LayoutComponents.tsx` provide feedback, tabs, table, and page-header components; `src/shared/lib/eventStatus.ts` and `formatEventDate.ts` provide event display helpers.
- **Related completed tasks:** T52 (base UI components), T53 (feedback components), and T68 (organization workspace shell) are present in `changes/completed/`.
- **Constraints / assumptions:** The task uses React/Cypress on the frontend and Spring/JUnit 5 on the backend. `GET /api/organizations/{organizationId}/events` returns `EventResponse`, which has capacity and lifecycle fields but no registration count; the plan adds a count-only field without returning participant records. The three declared dependencies are complete. Frontend `master` was clean, fetched, and fast-forwarded from `e6eecea` to `412d463`; backend `master` was clean, fetched, and fast-forwarded from `ff83e57` to `ce0fed1`. Matching task branches were created in both repos. The existing backend branch `fix/cors-config-source` and its three local commits were preserved. Figma metadata confirmed `Org – Events` at `15:10`, `Org – Events (empty)` at `49:1081`, and `Org – Events (loading)` at `49:1175`. Requirement, workflow, and Drive/ADR metadata remain unspecified. 

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the events screen inside the existing T68 organization workspace. The organization event list endpoint will return a dedicated response with the existing event fields plus a count-only `registrationCount`; it will not expose participant names or email addresses. The frontend will load that list once, derive the five status counts and selected rows, and use the current organization role for manager-only actions. Reuse the shared UI, feedback, date formatting, API client, and query conventions. Keep the existing route guards and nested event routes intact.

### Steps

1. **Backend red:** Add service and controller API tests proving that `GET /api/organizations/{organizationId}/events` returns each event’s `registrationCount`, including zero, and retains the current event fields and status. Assert the response does not include participant details. Confirm a behavioral red before implementation.
2. **Backend implementation:** Add a dedicated organization-event response DTO and map the existing event list to it with a count from the registration repository. Update the service/controller contract only for the list endpoint; retain existing response types for create, detail, edit, and publish endpoints.
3. **Frontend red:** Add Cypress coverage for the exact endpoint, all five tab counts and filtering, event row fields, manager/member actions, destination URLs, loading, empty, error/retry, and accessibility. Confirm behavioral failures before implementing the screen.
4. **Frontend API and types:** Add the organization-scoped list function and response types, including `registrationCount`, in the events feature using the existing `apiFetch` pattern.
5. **Page and route:** Create `OrganizationEventsPage` to validate/use the route organization id, query events, compute status counts, filter the selected tab, format date plus start time, render title/location/status/registrations, and choose actions by status and role. Replace only the `/org/:organizationId/events` placeholder while preserving T68 shell composition and member/manager guards.
6. **States and styling:** Use `TableSkeleton` while loading, the Calendar `EmptyState` with manager-only Create event when the list is empty, and `ErrorState` with retry on request failure. Add scoped styles matching frame `15:10`, loading frame `49:1175`, and empty frame `49:1081`, using existing tokens and responsive patterns. Preserve semantic tabs/table markup and keyboard access.
7. **Verification:** Run the focused backend service/API tests and backend suite, then the focused Cypress spec, frontend typecheck, lint, build, and wider Cypress suite. Record exact commands and outcomes in the Test log.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/OrganizationEventResponse.java` — list response with existing event properties and `registrationCount`.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/EventService.java` and `Business/src/main/java/com/ubb/events/implementation/EventServiceImpl.java` — list return type and count mapping.
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/EventController.java` — organization list endpoint response type.
- `repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/EventServiceTest.java` and the relevant `EventControllerApiTest.java` — count and safe response contract coverage.
- `repos/ubb-events-frontend/src/features/events/api.ts` and `src/features/events/types.ts` — typed organization event list request/response.
- `repos/ubb-events-frontend/src/features/organizations/OrganizationEventsPage.tsx` — data query, tabs/counts, table, row actions, and states.
- `repos/ubb-events-frontend/src/App.tsx` and `src/styles/global.css` — events route integration and scoped presentation.
- `repos/ubb-events-frontend/cypress/e2e/organization-events-dashboard.cy.ts` — focused workflow coverage.

### Tests to write

- **Backend service:** verify the list maps all event fields and status and obtains the right registration count for each event, including zero.
- **Backend API:** verify authorized organization members receive the list with `registrationCount`, the original event properties remain, and no participant names/emails are returned.
- **Frontend API/UI:** stub `/api/organizations/{organizationId}/events`, verify the exact URL and render events in Draft, Published, Completed, and Canceled states. Check counts, pill selection/filtering, title/location, date/time, status badge, `x / capacity` and draft `—`.
- **Role/actions:** verify managers see Create event and Add follow-up only for completed rows; members do not see those controls. Verify Manage goes to the event Tasks tab and canceled View opens the event view.
- **States/accessibility:** hold the request to assert the loading skeleton; return an empty list to assert the Calendar empty state and manager-only create action; return an error and exercise retry. Verify tab roles/selection, table headers, accessible action names, keyboard focus, and invalid route handling.
- **Regression:** confirm the existing workspace shell, role guards, and nested routes remain intact.

### Risks / open questions

- Registration counts must remain an aggregate only; do not reuse the participant registrations endpoint because it returns names and email addresses.
- The current app uses flat React Router routes. Replace only the events list route element and preserve the T68 shell and existing nested route guards.
- Figma metadata and design context confirmed the target `Org – Events` frame at `15:10`; the empty and loading variants are `49:1081` and `49:1175`. A screenshot was captured as the visual target. No static asset from the frame is required for the events table itself.
- The backend repo had three unpushed commits on `fix/cors-config-source`; at the user’s direction that branch was preserved while switching to updated `master` for the T69 branch.
## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** Backend `EventServiceTest.includesRegistrationCountInOrganizationEventListWithoutParticipantData` and `EventControllerApiTest.memberReceivesAllOrganizationEvents`; frontend `cypress/e2e/organization-events-dashboard.cy.ts` (five acceptance tests).
- **Result:** ` .\gradlew.bat :Business:test --tests com.ubb.events.implementation.EventServiceTest` — 50 tests, 1 failed: `includesRegistrationCountInOrganizationEventListWithoutParticipantData`, Mockito reported `WantedButNotInvoked` for `RegistrationRepository.countByEventId(1L)`. ` .\gradlew.bat :RestServices:test --tests com.ubb.events.controllers.EventControllerApiTest --console=plain` — 27 tests, 1 failed: `memberReceivesAllOrganizationEvents`, JSONPath raised `PathNotFoundException` for `registrationCount`. `npx cypress run --browser electron --spec cypress/e2e/organization-events-dashboard.cy.ts --reporter spec` — 0 passing, 5 failing; each failed because the events dashboard elements/actions were absent from the placeholder route.
- **Failure reason:** The service failure shows list mapping does not request aggregate counts; the API failure shows the list response omits `registrationCount`; the Cypress failures reach the authorized organization route and show the planned dashboard, tabs, actions, empty state, and skeleton are missing. These are behavioral failures rather than compilation or environment failures.
### Green (after implementation)

- **Result:** `npx cypress run --browser edge --spec cypress/e2e/organization-events-dashboard.cy.ts --reporter spec` — 5 passing. `npm run typecheck`, `npm run lint`, and `npm run build` — all passed. Backend ` .\gradlew.bat :Business:test --tests com.ubb.events.implementation.EventServiceTest --console=plain` and ` .\gradlew.bat :RestServices:test --tests com.ubb.events.controllers.EventControllerApiTest --console=plain` — both passed. The T69 dashboard and route-focused Cypress checks passed (15 passing) before the final CSS-only full-width adjustment; after that adjustment the dashboard spec passed again (5 passing).
- **Full suite of the affected repo:** Frontend `npx cypress run --browser edge --reporter spec` — 108 passing, 3 failing out of 111. Two failures are in `organizations-list.cy.ts` at lines 41 and 60 (expected hidden spans to be visible); these same failures are documented in completed T68. One failure occurred in `routing-and-guards.cy.ts`; the focused dashboard and route check passed earlier (15/15), but a later standalone route-spec rerun stalled after seven of ten tests and was interrupted, so that full-suite route failure is not resolved. Backend ` .\gradlew.bat test --continue --console=plain` — 327 tests, 4 failures: `OrganizationServiceStructureTest.implementationIsAnnotatedAndImplementsServiceWithoutOperations`, `OrganizationServiceStructureTest.serviceInterfaceIsEmpty`, `AnnouncementControllerStructureTest.controllerDefinesRootMappingAndServiceDependencyWithoutHandlers`, and `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`. These same four failures are documented in completed T68. The focused T69 backend tests pass.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | Plan | The existing events endpoint did not provide registration counts, so the frontend could not display the required registrations column. | The plan was extended to update the API with a safe count-only `registrationCount` field and include the backend repository. | The user chose to extend the API so the dashboard can meet the display requirement without per-event calls or exposing participant data. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use a dedicated organization-event response DTO with an aggregate `registrationCount`; participant names and email addresses are not included. Fetch the organization event list once and derive status counts and filtering in the frontend. Reuse shared table, feedback, and date components. Scope the organization workspace to the full-width Figma layout.
- **Alternatives considered:** Per-event participant/registration requests would add requests and expose more data than needed. Adding the field to the generic event response would alter unrelated create, detail, edit, and publish contracts.
- **Limitations:** The backend count is retrieved per event in the list mapping, so a large event list may require a future aggregate query. The full backend and frontend suites include failures outside the T69 focused tests; see the Test log. Metadata for requirement, workflow, and Drive/ADR remains unknown.

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
- [ ] Moved to `changes/completed/`


