# T72 - Event workspace shell

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T72
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** Source task 25 — Event workspace shell
- **Use case(s):** N/A
- **Workflow:** N/A
- **Drive doc / ADR:** [Project requirements, use cases, workflows, API specification, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi)
- **Repo(s):** frontend
- **Branch:** `T72-event-workspace-shell`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/21
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-workspace-shell.cy.ts`; frontend `typecheck`, `lint`, and `build` scripts

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 25 — Event workspace shell

**Description:** Build the event workspace header and tab bar shared by the frames `Org – Event: Tasks` (15:412), `Registrations` (15:589), `Organizer chat` (15:822), `Announcements` (16:431) and `Follow-up` (16:591): back link to events, title with status badge, meta row (date and time range, location, number of places) and the underline tabs Tasks / Registrations / Organizer chat / Announcements / Follow-up that map to the nested routes.
Data from `GET /api/organizations/{organizationId}/events/{eventId}` (verify the event belongs to the organization, otherwise 404).
The actions slot is filled by task 26. For canceled events show a read-only banner; for completed events keep Follow-up enabled.
**Depends on:** T52 — Base UI components: layout and data display; T68 — Organization workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` defines the organization event routes; `/org/:organizationId/events` is a working page, while the task, registration, chat, announcement, and follow-up routes currently render placeholders through shared member/manager wrappers. `src/features/organizations/OrganizationWorkspace.tsx` provides the existing organization shell to compose with. `src/features/organizations/OrganizationEventsPage.tsx` shows the event status, date, location, and registration-count patterns. `src/features/events/api.ts` and `src/features/events/types.ts` contain the current organization event API/types; `src/shared/api/client.ts` and `src/shared/api/types.ts` define API behavior and shared response types. `src/shared/ui/Badge.tsx` and `src/shared/ui/FeedbackComponents.tsx` provide status and feedback components; `src/styles/global.css` contains the shell styles. Existing Cypress patterns are in `cypress/e2e/organization-workspace-shell.cy.ts` and `cypress/e2e/organization-events-dashboard.cy.ts`.
- **Related completed tasks:** T52 — Base UI components: layout and data display; T68 — Organization workspace shell. Both task files are in `changes/completed/`; T68 records successful merge of its frontend and backend PRs.
- **Constraints / assumptions:** T72 is frontend-only and uses the existing organization-scoped event-detail API, which returns `id`, `organizationId`, `categoryId`, `title`, `description`, `date`, `startTime`, `endTime`, `location`, `capacity`, and `status`; mismatched organization ownership returns 404. Preserve T68's shared shell, auth guards, and React Router route structure. Show a canceled-event read-only banner and keep Follow-up navigable for completed events. Frontend run/test commands are marked as pending in `context/project-context.md`; use the repository scripts and workspace frontend instructions for verification. The frontend was clean on `master` tracking `origin/master`; the user confirmed `git pull` returned “Already up to date.”

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Create a route-scoped `EventWorkspace` layout for the five organization event child routes. It will parse `organizationId` and `eventId`, fetch the organization-scoped event detail through the shared API client, and render one shared header plus nested-route tabs around each child page. The shell will derive status presentation from the event response, keep the actions area as an explicit slot for task 26, and preserve the existing T56/T68 authentication and organization guards. A canceled event will render a read-only notice and a completed event will retain a usable Follow-up link.

Keep event loading, not-found handling, and display formatting in the frontend feature layer; do not duplicate business authorization in the UI. The backend contract is treated as authoritative for organization ownership: a 404 from `/api/organizations/{organizationId}/events/{eventId}` is shown as not found. The existing flat React Router map will be wrapped with the shared layout rather than introducing an outlet refactor unless the current route composition requires it.

### Steps

1. **API/types:** Add a typed `getOrganizationEvent(organizationId, eventId)` call and an event-detail type (reusing or extending `OrganizationEvent` only where fields match) for `GET /api/organizations/{organizationId}/events/{eventId}`. Preserve `apiFetch` error/status behavior so an ownership mismatch remains a 404. Confirm the response's date/time, location, capacity/places, and lifecycle-status fields before implementation; do not invent a second endpoint.
2. **Tests first:** Add Cypress coverage and the minimum typed/component stubs needed to compile. Stub the exact organization-scoped detail URL and assert a behavioral red result for the missing shell/header/tabs before implementing the layout. Include route, status, error, and accessibility cases listed below.
3. **UI shell:** Implement `EventWorkspace` (or an equivalent event feature component) that validates numeric route parameters, loads the event detail with a stable query key, renders loading and retryable error states, and displays back-to-events navigation, title, status badge, and the meta row for date/time range, location, and number of places. Use semantic landmarks and accessible names; keep the actions slot empty/ready for task 26.
4. **Navigation integration:** Add the shared shell around the Tasks, Registrations, Organizer chat, Announcements, and Follow-up route elements while retaining existing member/manager guards and child content. Tabs must use the exact nested paths, expose active state, keyboard focus, and visible focus styles. Ensure completed events keep Follow-up enabled and canceled events expose a read-only banner without enabling organizer actions.
5. **Styling and verification:** Add scoped layout styles using existing design tokens/base components and the named Figma frames as the visual target. Verify responsive overflow and the required header/tab spacing, then run focused Cypress plus the repository's available typecheck/lint/build checks and the wider frontend suite; record exact red/green evidence in the task file.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/api.ts` — organization-scoped event-detail request.
- `repos/ubb-events-frontend/src/features/events/types.ts` — detail response type/field refinements, if required by the confirmed API contract.
- `repos/ubb-events-frontend/src/features/events/EventWorkspace.tsx` (or the existing organizations feature location if that is the established convention) — data loading, header, status/banner, meta row, actions slot, and tabs.
- `repos/ubb-events-frontend/src/App.tsx` — compose the shared event workspace around the five nested organization event routes without changing their guard policy.
- `repos/ubb-events-frontend/src/styles/global.css` — event workspace header, metadata, tabs, banner, loading/error, and responsive styles.
- `repos/ubb-events-frontend/cypress/e2e/event-workspace-shell.cy.ts` — focused behavior, API, state, routing, and accessibility coverage.

### Tests to write

- **Acceptance criterion 1 (description/UI/API):** stub `GET /api/organizations/3/events/42` and verify the back link, event title, lifecycle badge, formatted date and time range, location, places/capacity, and the five exact tab destinations on each representative child route.
- **Acceptance criterion 1 (status rules):** verify a canceled response renders a read-only banner and does not expose an enabled organizer action; verify a completed response keeps Follow-up enabled and links to `/org/3/events/42/follow-up`; verify the actions slot is present for task 26 without adding task-26 behavior.
- **Acceptance criterion 2 (dependencies):** visit the shell through the existing authenticated organization routes and assert T68's workspace/app shell and existing member/manager guards remain in effect; non-organization routes remain unchanged.
- **Acceptance criterion 3 (states/accessibility):** verify loading status, invalid route parameters, 404/not-found, other API error with retry, semantic navigation/tab landmarks, active tab state, keyboard navigation/focus, and a usable narrow viewport/overflow behavior.
- **API contract/ownership:** assert the exact organization-scoped URL is requested and a 404 response is rendered as not found, documenting that cross-organization event access is rejected by the backend contract.
- Run the focused Cypress spec before implementation to capture a behavioral red result, then rerun it green and execute the affected frontend typecheck/lint/build and full Cypress suite where available.

### Risks / open questions

- **Resolved before implementation:** Backend `EventResponse` uses `title`, `date`, `startTime`, `endTime`, `location`, `capacity`, and `status`; organization mismatch returns 404. The UI labels total capacity as places because the specified response has no remaining-capacity field.
- The existing child routes are flat elements and mostly placeholders. Keep their current paths and guards; if a child page later needs an outlet, use the smallest route change that preserves those guards.
- Figma design-context requests used the named node IDs, but the Figma MCP Starter-plan call limit prevented inspection of all five frames. The Registrations frame context was available; detailed visual comparison across every active-tab frame remains unverified.
- `context/project-context.md` says frontend run/test commands are future work. Use the scripts already declared in `repos/ubb-events-frontend/package.json` and record any unavailable checks rather than guessing commands.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/event-workspace-shell.cy.ts` (three focused Cypress scenarios written before implementation).
- **Result:** `npx cypress run --browser electron --spec cypress/e2e/event-workspace-shell.cy.ts --reporter spec` — 0 passing, 3 failing. Failures: `renders the shared header and nested tabs` could not find `[data-testid=event-workspace]`; `shows canceled read-only state and keeps completed follow-up enabled` could not find `[data-testid=event-workspace-canceled]`; `renders loading and not-found states` could not find `[data-testid=event-workspace-not-found]`.
- **Failure reason:** After installing Cypress 16.2.0 and running it outside the sandbox so it could use the local binary, all three tests reached the app. Each failed because the corresponding planned event workspace element is absent before implementation. This is behavioral red evidence for the missing workspace shell and states, not a compilation or test setup failure.

### Green (after implementation)

- **Result:** `npx cypress run --browser electron --spec cypress/e2e/event-workspace-shell.cy.ts --reporter spec` — 5 passing, 0 failing. Covers the shared header and metadata, organization-scoped API URL, all five route destinations, active/focus state, narrow-screen tab overflow, canceled banner, completed Follow-up route, loading, retry after server error, scoped 404, and invalid event ID. `npm run typecheck`, `npm run lint`, and `npm run build` all passed.
- **Conflict-resolution verification:** After fetching and merging `origin/master` to resolve PR #21 conflicts, `npx cypress run --browser electron --spec "cypress/e2e/event-workspace-shell.cy.ts,cypress/e2e/event-details.cy.ts" --reporter spec` passed 14/14 tests; `npm run typecheck`, `npm run lint`, and `npm run build` also passed. The conflict-resolution merge was committed as `4cd7bdf` and pushed.
- **Full suite of the affected repo:** `npx cypress run --browser electron --reporter spec` — 21 specs, 116 tests: 106 passing, 10 failing. Failures are in unchanged specs: `organization-details.cy.ts` (one request-path assertion), `organization-events-dashboard.cy.ts` (five auth membership errors), `organization-workspace-shell.cy.ts` (two organization stub expectations), and `organizations-list.cy.ts` (two hidden-span visibility assertions). These specs do not touch T72's changed source files; their status on the pre-T72 base was not verified.
- **TDD deviation:** The initial pre-implementation red run covered the three core shell/status/not-found scenarios. Additional metadata, navigation/accessibility, loading, retry, and invalid-ID assertions were added after implementation during test expansion; their passing results are not pre-implementation red evidence. The Cypress fixture was also adjusted to normalize URL paths and provide the manager role required by the existing Follow-up guard.

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

- **Decisions:** Fetch event details with the organization-scoped endpoint and let its 404 represent an event outside that organization. Display the API's total `capacity` as the number of places. Compose one `EventWorkspace` wrapper around the five existing flat routes, retain their existing organization guards, and use links with `aria-current` for route navigation. Keep the actions slot empty for task 26.
- **Alternatives considered:** A nested router outlet could share the layout, but wrapping the current flat route elements was a smaller change. Remaining places could be calculated from registrations, but that data is not present in the specified detail response.
- **Limitations:** The task-26 actions slot is a placeholder. Figma MCP rate limits prevented detailed inspection of all five named frames, so active-tab visual variants were not fully compared. The full Cypress suite had 10 failures in unchanged organization specs; their base-branch status is unknown.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — frontend `b4e57a6`, conflict-resolution merge `4cd7bdf`
- [x] PR created — [Frontend PR #21](https://github.com/Vasile07/ubb-events-frontend/pull/21)
- [x] PR reviewed — approved by user
- [x] **Gate 3:** merged by user — Frontend PR #21
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`


