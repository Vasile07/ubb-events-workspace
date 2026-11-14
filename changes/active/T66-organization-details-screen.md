# T66 - Organization details screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T66
- **Type:** workflow
- **Owner:** Karina Kentsch
- **Requirement(s):** 
- **Use case(s):** UC-04
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T66-organization-details-screen`
- **PR link(s):** 
- **Tests:** `cypress/e2e/organization-details.cy.ts`; focused `routing-and-guards.cy.ts`; `npm run typecheck`, `npm run lint`, `npm run build`; full Cypress suite.

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 19 — Organization details screen

**Description:** Implement page `Student Screens`, frame `Organization details` (14:833): gradient hero with logo tile, name, description, status badge and the membership action (same logic and mutation as task 18), "Upcoming events" as two-column `EventCard` grid (events from `GET /api/events` for this organization, non-completed) and the "About" card explaining that member lists are private.
Data from `GET /api/organizations/{organizationId}` (route `/organizations/:organizationId`). Show the 404 page for an unknown id and an empty state when there are no upcoming events. No organization code is shown anywhere.
**Depends on:** T54 — Event card and event status logic; T65 — Organizations list screen.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` routes `/organizations/:organizationId`; `src/shared/api/organizations.ts` now defines the detail query; `src/features/organizations/OrganizationDetailsPage.tsx` renders the page and states; `src/features/organizations/OrganizationsListPage.tsx` provides the membership mutation pattern; `src/shared/ui/EventCard.tsx` is reused for upcoming events; `src/shared/api/types.ts` contains the organization detail response type; `public/ui-assets/organization-*.svg` contains the exact frame assets.
- **Related completed tasks:** T54 — Event card and event status logic (frontend; merged as PR #6).
- **Constraints / assumptions:** Before implementation, the frontend repo was clean on `master`; `git pull --ff-only origin master` succeeded and reported `Already up to date`. The user confirms T65 is merged. The named Figma frame `Student Screens / Organization details` (`14:833`) was inspected; its hero, back link, side-by-side event/About columns, privacy notes, and SVG assets are reflected in the UI. Requirement(s), Workflow, and Drive doc / ADR fields are intentionally left blank per user. Membership behavior follows the existing list screen's request/status handling; the source task 18 specification was not available locally.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Build a route-backed organization details feature under `features/organizations`, keeping API calls in the shared API layer and rendering states in the page component. Fetch the organization by route id and its events with the existing events endpoint filtered by `organizationId`; exclude completed events before rendering the two-column `EventCard` grid. Reuse the membership status/request behavior from `OrganizationsListPage`, the shared feedback components, and `EventCard`, and add only the screen-specific layout and styles required by the named Figma frame. A 404 response from the organization endpoint will render the app's not-found state, while loading, request failure, and no-upcoming-events states remain explicit and accessible.

### Steps

1. **Dependency and contract check (repository/API):** verify the local frontend `master` contains merged T65 and T54 before creating the task branch; inspect the organization detail response and event DTOs used by the backend/API contract, including membership status, event lifecycle status, and the organization id query parameter. Confirm the exact Figma frame `Student Screens / Organization details` (`14:833`) for layout and responsive behavior.
2. **Tests first (API/page behavior):** add Cypress scenarios with intercepted API responses for a populated organization, pending/member/open membership states and request mutation, upcoming-event filtering, empty upcoming events, organization 404, loading, generic API failure, and accessible headings/buttons. Add a request assertion that no organization code is displayed and that event loading uses `/api/events?organizationId=<id>` (or the established equivalent query serialization). Run the tests before implementation and record a behavioral red result.
3. **API/types layer:** add typed `getOrganization(organizationId)` and reuse or extend `getEvents({ organizationId })` as needed; model the detail response without introducing organization-code presentation. Preserve `ApiError` status handling so a 404 can be distinguished from other failures.
4. **UI/page layer:** create `OrganizationDetailsPage`, read and validate `organizationId` from `useParams`, fetch organization and events with React Query, and render the gradient hero (logo tile, name, description, status badge, membership action), upcoming-events section, and private-member-list About card. Reuse the list screen's membership request mutation, 409 handling, loading state, and success/error toast copy. Filter completed events and map remaining API events to `EventCard` props; link card actions to event details.
5. **Routing and styling:** replace the placeholder `/organizations/:organizationId` route with the authenticated app shell and the new page. Add screen-specific CSS using existing tokens and responsive two-column-to-single-column behavior; use `EmptyState`, `ErrorState`, and the existing not-found presentation for the required states.
6. **Verification:** run the focused Cypress spec, then `npm run typecheck`, `npm run lint`, `npm run build`, and the wider Cypress suite. Record any pre-existing environment/API-client failures separately from T66 results.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/organizations/OrganizationDetailsPage.tsx` — route page, queries, membership action, event filtering/mapping, and states.
- `repos/ubb-events-frontend/src/shared/api/organizations.ts` — organization detail request and shared membership request usage.
- `repos/ubb-events-frontend/src/shared/api/types.ts` — detail response type(s), only as required by the API contract.
- `repos/ubb-events-frontend/src/features/events/api.ts` / `src/features/events/types.ts` — only if the organization filter or event DTO needs an additive typed change.
- `repos/ubb-events-frontend/src/App.tsx` — authenticated route replacement.
- `repos/ubb-events-frontend/src/styles/global.css` — hero, About card, upcoming-events grid, and responsive styles.
- `repos/ubb-events-frontend/cypress/e2e/organization-details.cy.ts` — end-to-end behavior and accessibility coverage.

### Tests to write

- **Acceptance criterion 1 / complete screen:** populated fixture verifies hero identity/description, status badge, membership action, About copy, two-column event cards, event navigation, and absence of organization code.
- **Acceptance criterion 1 / API interactions and rules:** intercepts verify `GET /api/organizations/{organizationId}`, organization-filtered `GET /api/events`, completed-event exclusion, and POST membership request with the existing pending/409 behavior.
- **Acceptance criterion 2 / dependencies:** implementation pre-check verifies T54 and the user-confirmed merged T65 are present on the base branch before branch creation.
- **Acceptance criterion 3 / states and accessibility:** Cypress covers loading skeletons, non-404 API error with retry, unknown-id 404 state, no-upcoming-events empty state, accessible page heading/sections/actions, and keyboard-usable membership/event actions.
- **Regression:** `npm run typecheck`, `npm run lint`, `npm run build`, and the full Cypress suite.

### Risks / open questions

- The exact organization-detail JSON shape was checked against the backend response contract and is represented by the frontend detail response type.
- The description says “same logic and mutation as task 18,” but the local task file is not available. Use the existing list-screen behavior as the provisional implementation and confirm any task-18-specific labels or state precedence from the contract/task source.
- The repository has no dedicated frontend unit-test runner; Cypress is the established browser test runner.
- The previous `/organizations/:organizationId` placeholder used the app's standard `page-*` route test marker; the implemented page preserves that marker for route-guard coverage.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/organization-details.cy.ts` — populated view, API filtering, empty events, membership request, 404, loading, retryable error, no-code, Figma asset and desktop geometry checks.
- **Result:** TDD deviation: tests were written before feature code, but no valid behavioral red result was observed before implementation. Initial Cypress attempts failed because the Cypress executable was unavailable in the restricted cache or the browser process exited with code `-2147483645`. The implementation proceeded without the required red result. Some Figma-specific assertions added after implementation first failed on the missing back link; these are post-implementation checks and do not count as red evidence. The red checklist remains unchecked.
- **Failure reason:** Cypress setup/browser execution failed before any completed assertion result, so this did not demonstrate missing planned behavior. Implementation before a valid red result is a recorded deviation from the required TDD sequence.

### Green (after implementation)

- **Result:** After implementation, `npm run typecheck`, `npm run lint`, and `npm run build` passed. The final `organization-details.cy.ts` run passed 5/5, including request filtering, membership, empty, 404, loading, retry, Figma asset dimensions, and frame geometry assertions. `routing-and-guards.cy.ts` passed 10/10 in the final full-suite run.
- **Full suite of the affected repo:** `CYPRESS_CACHE_FOLDER='C:\Users\irimi\AppData\Local\Cypress\Cache' npx cypress run --browser electron --reporter spec` ran 18 specs: 98 passed, 2 failed. Both failures are in `organizations-list.cy.ts` (lines 41 and 60), where assertions expect a hidden `organizations-page` span to be visible. That file was not changed by T66. No baseline full-suite run was taken, so these failures cannot be conclusively labeled pre-existing.

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

- **Decisions (draft for user review):** Keep the organization detail request in the shared API layer and screen queries/rendering in the page; filter completed events on the client because the existing event endpoint supplies organization events; preserve the existing membership request and pending-state behavior; reuse T54's `EventCard`, shared feedback components, and semantic style tokens.
- **Alternatives considered (draft for user review):** A new backend endpoint or an organization-specific event card was unnecessary because the task specifies existing endpoints and T54's reusable `EventCard`.
- **Limitations (draft for user review):** No valid pre-implementation behavioral red result was obtained. The full Cypress suite has two unresolved organizations-list assertion failures, and the source task 18 specification was not available locally for direct comparison.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
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


