# T65 - Organizations list screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T65
- **Type:** workflow
- **Owner:** Karina Kentsch
- **Requirement(s):** 
- **Use case(s):** UC-04
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T65-organizations-list-screen`
- **PR link(s):** 
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/organizations-list.cy.ts`; frontend typecheck, lint, build, and Cypress suites

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 18 — Organizations list screen

**Description:** Implement page `Student Screens`, frame `Organizations` (14:699): header, search field and a 3-column grid of organization cards (icon tile, status badge, name, description, action).
Data from `GET /api/organizations` including the user's status.
Actions:
- MEMBER → "Open organization" (to `/org/:organizationId/events`)
- PENDING → disabled "Waiting for approval"
- NONE → "Request membership" calling `POST /api/organizations/{organizationId}/membership-requests` (no body), then switch the card to Pending without a reload and show a toast; handle an already-pending 409 gracefully.

The search filters by organization name only, on the client. Equal-height cards with the action pinned to the bottom. Loading, empty ("No organizations match") and error states.
**Depends on:** T53 — Feedback components: toast, empty state, skeleton, error state; T57 — App shell and top navigation.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/shared/api/organizations.ts` currently exposes only `GET /api/organizations`; `src/shared/api/types.ts` defines `OrganizationResponse` with `organizationId`, `name`, nullable `description`, membership `status`, and `role`. `src/shared/ui/FeedbackComponents.tsx` provides toast and feedback primitives; `src/shared/ui/AppShell.tsx` provides the shell; `src/App.tsx` currently has showcase/fallback routes. 
- **Related completed tasks:** `changes/completed/T53-feedback-components-toast-empty-state.md` (merged), `changes/completed/T57-app-shell-and-top-navigation.md` (merged). 
- **Constraints / assumptions:** The frontend repo is clean on up-to-date `master` at `b868dbb` (T61 merge). Frontend verification commands are established in completed T57: `npm run typecheck`, `npm run lint`, `npm run build`, and Cypress (`npm run cypress:run`; confirm package scripts before implementation). 

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the Organizations route as a focused frontend feature that fetches the typed organization list once, derives the visible cards from a local name-only search value, and keeps membership-request state local to the page. Reuse the existing API client, React Query, shared `SearchField`, `Badge`, `Button`, toast, empty, error, and skeleton primitives, and the authenticated `AppShell` already used by the route. Before styling, inspect the exact Figma `Student Screens` / `Organizations` frame `14:699` through the Figma workflow and use it as the visual target.

The API layer will expose the membership-request `POST` beside the existing organizations query. The UI will distinguish `MEMBER`, `PENDING`, and `NONE` actions, optimistically switch a successful request to pending without refetching, and treat a 409 as an already-pending result with the same pending card state and a user-facing toast. Other failures remain errors and leave the request action available.

### Steps

1. **UI/API tests first (frontend):** Add Cypress coverage for the `/organizations` route with intercepted `GET /api/organizations` and membership-request responses. Cover the rendered header/search/grid/card fields, all three membership states, client-side name filtering, loading skeletons, no-match empty state, GET error/retry state, successful request mutation, 409 handling, and the member navigation target. Run the new spec before implementation and record a behavioral red result.
2. **API layer (frontend):** Extend `src/shared/api/organizations.ts` with a typed `requestOrganizationMembership(organizationId)` function that calls `POST /api/organizations/{organizationId}/membership-requests` with no body. Reuse `ApiError.status` for conflict handling; do not put business rules in the component beyond mapping the API result to UI state.
3. **Organizations UI (frontend):** Create a page-level feature component and card presentation for the named Figma frame. Use `useQuery` for the organization list, a controlled `SearchField` for a case-insensitive name-only filter, and shared feedback primitives for loading, empty (`No organizations match`), and error/retry states. Render an accessible three-column responsive grid with equal-height cards, pinned bottom actions, icon tile, status badge, name, nullable-description fallback, and the required action labels/links.
4. **Membership interaction (frontend/UI):** Keep per-organization request state in local component state or a mutation keyed by organization ID. On `NONE`, disable the request control while submitting, call the API, switch that card to `PENDING` without a reload, and show a success toast. On a 409, switch/show pending and show an informational toast; on another failure, show an error toast and restore the action. `MEMBER` links to `/org/:organizationId/events`; `PENDING` is disabled and labeled `Waiting for approval`.
5. **Route and styling integration (frontend/UI):** Replace the `/organizations` placeholder with the page inside `RequireAuth` and `AppShell`, preserving existing guards and public routes. Add only the route-specific styles and test IDs needed for the Figma layout, desktop three-column grid, equal card heights, pinned actions, and responsive behavior.
6. **Verification:** Run the focused Cypress spec, then `npm run typecheck`, `npm run lint`, `npm run build`, and the full `npm run cypress:run`. Record exact red and green evidence in the task file and visually compare the implementation against Figma frame `14:699`.

### Files to create / modify

- `repos/ubb-events-frontend/src/shared/api/organizations.ts` — add the membership-request POST function.
- `repos/ubb-events-frontend/src/features/organizations/OrganizationsListPage.tsx` — fetch, search, feedback states, request interaction, and card rendering.
- `repos/ubb-events-frontend/src/features/organizations/OrganizationCard.tsx` (if keeping card markup separate improves readability) — organization card presentation and action states.
- `repos/ubb-events-frontend/src/App.tsx` — replace the `/organizations` placeholder with the authenticated Organizations page.
- `repos/ubb-events-frontend/src/styles/global.css` — route-specific header, grid, card, icon-tile, and responsive styles.
- `repos/ubb-events-frontend/cypress/e2e/organizations-list.cy.ts` — acceptance and edge-case coverage written before implementation.

### Tests to write

- **Acceptance criterion 1 / page and API behavior:** Cypress verifies the Organizations page calls `GET /api/organizations`, renders the header, search field, three-column card grid, icon tile, status, name, description, and correct action for `MEMBER`, `PENDING`, and `NONE`; the member action navigates to `/org/{organizationId}/events`.
- **Acceptance criterion 1 / mutation behavior:** Cypress verifies `NONE` sends a bodyless `POST` to `/api/organizations/{organizationId}/membership-requests`, changes only that card to pending without a GET reload, and displays a success toast; a 409 produces the pending state and a graceful informational toast; non-409 failures display the error feedback and keep retry available.
- **Acceptance criterion 1 / search and layout:** Cypress verifies filtering is case-insensitive and uses organization name only, descriptions do not affect matches, and card actions remain at the bottom of equal-height cards at the desktop viewport.
- **Acceptance criterion 2 / dependencies:** The focused run uses merged T53 feedback primitives and T57 shell on frontend `master`; route rendering remains behind the existing auth guard.
- **Acceptance criterion 3 / states and accessibility:** Cypress verifies loading skeletons, the exact `No organizations match` empty message, GET error state with retry, semantic search labeling, accessible card/action names, disabled pending controls, and responsive grid behavior.

### Risks / open questions

- The backend response shape is currently represented by `OrganizationResponse`; confirm the live backend contract still uses numeric `organizationId`, nullable `description`, and `status` values `MEMBER | PENDING | NONE` before implementation.
- The description says “icon tile” but provides no organization-icon field or asset. Use the existing design-system icon treatment or a deterministic initials/icon fallback after inspecting Figma frame `14:699`; do not invent per-organization imagery.
- The exact success and conflict toast copy is not specified. Use concise user-facing copy consistent with T53 and record the chosen wording in the implementation’s design decisions.
- Cypress must run with the frontend server available at the configured base URL; if the backend is unavailable, intercept all API calls in the focused spec and record any environment limitation separately.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `repos/ubb-events-frontend/cypress/e2e/organizations-list.cy.ts` (3 tests).
- **Result:** `$env:CYPRESS_CACHE_FOLDER='C:\Users\irimi\AppData\Local\Cypress\Cache'; npm run cypress:run -- --browser electron --spec cypress/e2e/organizations-list.cy.ts --reporter spec` with the local Vite server running; the first test failed and produced a Cypress screenshot because the route rendered the existing placeholder.
- **Failure reason:** The planned `[data-testid=organizations-page]` and organization card elements were absent before implementation. This was a behavioral failure caused by the missing Organizations screen, not a compilation or setup failure. Cypress required its local binary to be installed and run with the configured cache path.

### Green (after implementation)

- **Result:** Focused `npm run cypress:run -- --browser electron --spec cypress/e2e/organizations-list.cy.ts --reporter spec` passed: 3 passing, 0 failing. `npm run typecheck`, `npm run lint`, and `npm run build` passed.
- **Full suite of the affected repo:** Initial full Cypress run reached 54 passing and 1 failing: the pre-existing route guard assertion expected a `data-testid^=page-` marker on `/organizations`. The marker was preserved, and the affected routing assertion passed on rerun. A final full-suite run used `$env:CYPRESS_CACHE_FOLDER='C:\Users\irimi\AppData\Local\Cypress\Cache'` with `--reporter json`; the environment truncated the aggregate output before completion, so no final total is claimed here.

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

- **Decisions:** Added a typed bodyless membership-request API function and a React Query-backed Organizations page. Search is local and name-only. Each card owns its request state, switches to pending on success or 409, and uses existing toast, feedback, and button primitives. The exact Figma frame uses a shared building icon in the brand-soft 56px tile, so the implementation uses the existing local `figma-building.svg` asset rather than an initial fallback. Preserved the existing `page-organizations` route marker for compatibility with route guard coverage.
- **Alternatives considered:** Kept the card in the page module instead of adding a separate card file because the interaction state is small and page-specific. Used existing shared design primitives rather than adding a new component dependency.
- **Limitations:** The exact Figma frame `14:699` was inspected through the canonical file session. Its building icon tile, 480px search field below the page header, 32px card gaps, 344px cards, and token-backed colors informed the final styling. The execution environment truncated the final aggregate output of the post-fix full Cypress run; focused Organizations coverage and the affected routing assertion passed.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`


