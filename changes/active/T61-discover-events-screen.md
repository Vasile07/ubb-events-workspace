# T61 - Discover events screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T61
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** 
- **Use case(s):** UC-09
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T61-discover-events-screen`
- **PR link(s):** 
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/discover-events.cy.ts`; `npm run typecheck`, `npm run lint`, `npm run build`, and the frontend Cypress suite.

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 14 — Discover events screen

**Description:** Implement page `Student Screens`, frame `Discover events` (13:119): header with result count, `SearchField`, chips (Organization, Date range, "Open for registration" toggle) with "Clear filters", 3-column grid of `EventCard` and "Load more events".
Data from `GET /api/events` (non-drafts; completed and canceled included with their status); the Organization chip lists organizations from `GET /api/organizations` and filters by `organizationId`. If the API has no search/filter params do the filtering client-side and mention it in the PR. Keep filters in the URL query string.
States: loading → frame `Discover events – loading` (48:587), no results → `Discover events – no results` (48:694, EmptyState Search with "Clear filters"), error → `Discover events – error` (48:757, ErrorState + error toast).
Cards link to `/events/:eventId`.
**Depends on:** T53 — Feedback components: toast, empty state, skeleton, error state; T54 — Event card and event status logic; T57 — App shell and top navigation.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` contains the authenticated `/events` placeholder and a `/components` demo for `SearchField`, `FilterChip`, and `EventCard`; `src/shared/api/client.ts` provides `apiFetch`, and `src/shared/api/organizations.ts` provides `getOrganizations()`. `src/shared/ui/EventCard.tsx`, `FeedbackComponents.tsx`, `SearchField.tsx`, and `FilterChip.tsx` provide the reusable card, loading/error/empty, search, and filter primitives. `src/shared/lib/eventStatus.ts` provides event status mapping. There is no frontend events API module yet.
- **Related completed tasks:** `changes/completed/T53-feedback-components-toast-empty-state.md`, `T54-event-card-and-event-status.md`, and `T57-app-shell-and-top-navigation.md` are merged frontend dependencies. Backend workflow tasks `T29-discover-events` and `T30-filter-events` define the current event list endpoint and its filters.
- **Constraints / assumptions:** Frontend repo is clean on `master` and was up to date after fetch/pull. User confirmed T61 should include completed and canceled events and use T30 API filters. T29's implemented contract is still published-only, and T30 preserves published-only filtering; status is `PUBLISHED` even for past events. The user requested the backend fix as separate task [T82](../todo/T82-delete-and-discover-events.md), including capacity and spotsTaken; T61 proceeds on the assumption that canceled events will eventually be included. T30 query parameters are `search`, `organizationId`, `from`, and `to`. The canonical Figma file is accessible in session `e3494081-30d1-4e45-9b35-20f8ad22bc7f`; the main frame (`13:119`), loading (`48:587`), and no-results (`48:694`) contexts were read. The error frame (`48:757`) returned `INVALID_ARGUMENT`, and subsequent metadata access hit the Starter plan call limit; its visual context remains unverified.

**Backend follow-up:** The user requested the changes as separate task [T82](../todo/T82-delete-and-discover-events.md), including `capacity` and `spotsTaken` in discovery summaries. T61 proceeds on the assumption that canceled events will eventually be included.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the authenticated `/events` route as a focused discover-events feature in the frontend. Add a typed events API module that requests `GET /api/events` with T30's `search`, `organizationId`, `from`, and `to` parameters, loads organizations for the Organization filter, and synchronizes the filter state with `URLSearchParams`. Render the result count, search and filter controls, a responsive three-column `EventCard` grid, and event links while reusing T53/T54/T57 components and state patterns. The UI will consume T82's expected response contract, including `capacity`, `spotsTaken`, and canceled events. Map backend `PUBLISHED` summaries into the card's status model and derive completed display state using T30's ended-event rule (event end time, or start time when absent), since the backend retains `PUBLISHED` status until T22 changes it. T82 owns deletion semantics, canceled-event discovery, and the extra summary fields.

Client-side filtering will be used only as a compatibility fallback when the API does not support a requested parameter; the normal path uses T30 server-side filters. The screen will progressively reveal additional loaded results through “Load more events” because the API is not paginated. This task remains frontend-only; the backend contract change is a prerequisite/open dependency and is not implemented here.

### Steps

1. **Dependency and contract check (API/UI):** verify T53, T54, and T57 are present on frontend `master`; type the event summary including `capacity` and `spotsTaken` from T82; map `PUBLISHED`/`CANCELED` statuses and derive completed display status using T30's ended-event rule.
2. **Tests first (UI/API):** add Cypress coverage before implementation for URL-restored filters, API requests, organization loading, result count, card links, filter clearing, open-for-registration behavior, load-more reveal, loading skeletons, no-results EmptyState action, error ErrorState retry, and error toast. Capture a behavioral red result caused by the missing screen.
3. **API and query state (API/UI):** create typed event response definitions including `capacity` and `spotsTaken`, and `getEvents`/organization calls. Parse and normalize search, organization, date range, and open-registration values from the URL; update the URL without losing unrelated query parameters and refetch when criteria change. Apply the open-registration toggle using the mapped lifecycle status and capacity/count data. Handle malformed or incomplete dates consistently with the API contract.
4. **Discover screen (UI):** replace the authenticated `/events` placeholder with a page using the app shell. Render the result count, `SearchField`, Organization and Date range chips/popovers, the registration toggle, Clear filters, a responsive three-column grid of `EventCard`s linking to `/events/:eventId`, and progressive “Load more events” rendering.
5. **States and accessibility (UI):** render the named loading frame with `EventCardSkeleton`s, the Search `EmptyState` with Clear filters when no results exist, and the `ErrorState` plus error toast on request failure. Add semantic labels, keyboard operation, focus treatment, status announcements where needed, and preserve filter state across reload/navigation.
6. **Verification (UI):** run the focused Cypress spec, then `npm run typecheck`, `npm run lint`, `npm run build`, and the wider Cypress suite; record exact red/green results and any unrelated failures in the task log.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/types.ts` and `api.ts` (typed event summaries, filter model, API calls)
- `repos/ubb-events-frontend/src/features/events/DiscoverEventsPage.tsx` (screen, URL state, loading/empty/error behavior)
- `repos/ubb-events-frontend/src/App.tsx` (wire the authenticated `/events` route)
- `repos/ubb-events-frontend/src/shared/api/organizations.ts` and/or related API types (only if the existing organization response needs a typed extension)
- `repos/ubb-events-frontend/src/styles/global.css` (discover layout, filter controls, grid and responsive states)
- `repos/ubb-events-frontend/cypress/e2e/discover-events.cy.ts` (focused behavior and accessibility tests)

### Tests to write

- **Acceptance criterion 1:** Cypress verifies the complete discover screen: result count, search, Organization and Date range filters, open-registration toggle, Clear filters, three-column cards, event detail destinations, and Load more behavior. API stubs assert `search`, `organizationId`, `from`, and `to` are sent and that completed/canceled statuses render through `EventCard`.
- **Acceptance criterion 2:** a pre-implementation repository/dependency check records merged T53, T54, and T57; the focused test must run against the expected backend response contract from the follow-up bug task.
- **Acceptance criterion 3:** Cypress covers loading skeletons, no-results EmptyState and clear action, request failure ErrorState and toast, keyboard labels/focus, URL persistence across reload, invalid/partial date criteria, and organization-loading failure handling.

### Risks / open questions

- The current backend T29/T30 implementation returns published events only and omits canceled events; the separate T82 bug-fix task must change deletion and discovery semantics before canceled cards can appear.
- The existing T29 summary response omits `capacity` and `spotsTaken`; the user added these fields to T82 so T54 `EventCard` and the open-registration filter can use real values without defaults.
- T30 has no pagination, so Load more can only reveal more items already returned. If product requires server pagination, that is a separate API change.
- The canonical Figma file and main/loading/no-results contexts are accessible. The error frame (`48:757`) could not be read (`INVALID_ARGUMENT`), and metadata access then reached the Starter plan call limit. Follow the T53 ErrorState component and task description unless that frame becomes accessible before implementation.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/discover-events.cy.ts` — `loads event cards, statuses, organization filters, and detail destinations`; `restores query filters from the URL and lets users clear them`; `sends a partial date range and keeps events usable if organizations fail to load`; `shows loading and no-results feedback states with a clear-filters action`; `shows the error state and toast, then retries successfully`.
- **Result:** `npm run cypress:run -- --spec cypress/e2e/discover-events.cy.ts --browser electron --reporter spec` — 0 passing, 5 failing.
- **Failure reason:** Behavioral red. The authenticated `/events` route renders the placeholder rather than the planned discover page. The discover page, result controls, cards, loading/empty/error states are absent, and no discovery API request is made. The initial test harness used an unmatched network intercept; it was corrected to stub `window.fetch`, and this final run showed the authenticated placeholder with the profile stub working. An earlier run was also blocked by Cypress's sandbox cache path; Cypress was installed and the valid red run was executed outside the sandbox.

### Green (after implementation)

- **Result:** `npm run cypress:run -- --spec cypress/e2e/discover-events.cy.ts --browser electron --reporter spec` — 5 passing. `npm run typecheck`, `npm run lint`, and `npm run build` all passed on the final implementation. `git diff --check` passed.
- **Full suite of the affected repo:** `npm run cypress:run -- --browser electron --reporter spec` — 50 passing across 11 specs.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “It seems that some changes to the backend are needed for deleting an event and discover event.” | Created a separate backend bug-fix task for draft deletion, canceling published events, and including canceled events in discovery; kept T61 frontend-only and assumed canceled events will eventually be included. | The user requested that backend work be tracked separately while T61 continues. |
| 2 | plan | T54 `EventCard` requires `capacity` and `spotsTaken`, but the current discovery summary omits them. | Added `capacity` and `spotsTaken` (registration count) to T82's planned discovery summary response. | The user chose to extend T82 so T61 can render cards and filter open events using API data. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Keep the discover page under the existing authenticated app shell. Treat URL query parameters as the persistent source for search, organization, date range, and open-only filters; send supported criteria to the T30 API. Map backend `PUBLISHED` into the card lifecycle, then display it as completed after its end time (or start time when no end time exists), matching T30. Use T82's planned `capacity` and `spotsTaken` values for registration availability. Reveal loaded results six at a time because the endpoint has no pagination.
- **Alternatives considered:** Filter all criteria in the browser, which would duplicate T30's server behavior and be less efficient; add server pagination, which is outside the current API contract; add backend changes to T61, which the user assigned to separate task T82.
- **Limitations:** Canceled events and capacity/count fields depend on T82 changing the backend contract; the current backend does not return them yet. “Load more” reveals already-loaded results and does not fetch another page. The Figma error-state frame was not accessible due to an invalid node response and Starter plan call limit, so the implementation follows the task description and existing T53 `ErrorState` pattern. The Electron runner emitted a deprecation warning, but the suite completed successfully.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`


