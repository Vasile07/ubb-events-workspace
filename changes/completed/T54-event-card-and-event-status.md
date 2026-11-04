# T54 - Event card and event status logic

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T54
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** UC-09, UC-10
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T54-event-card-and-event-status`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/6
- **Tests:** `cypress/e2e/event-card.cy.ts`; `npm run typecheck`; `npm run lint`; `npm run build`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 7 — Event card and event status logic

**Description:** Build `EventCard` from page `Design System`, component `Event card` (3:301): gradient cover with date chip and status badge, organization name row, title, location row, capacity progress bar and "X of Y spots taken", and the "View details" action.
Create the pure function `getEventBadge(event, registrationStatus)` returning label and tone: Open (Success), Few spots left (Warning, when ≤10% or ≤5 places remain), Full (Danger), Registered (Info), Completed (Info), Canceled (Danger).
Cover precedence rules in unit tests (e.g. Canceled beats Full; Registered beats Open).
Format dates as "Sat, 14 Nov · 18:00" with a shared `formatEventDate` helper.
**Depends on:** T51 — Base UI components: actions and form fields; T52 — Base UI components: layout and data display.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/shared/ui/Badge.tsx` provides tone-based badges; `LayoutComponents.tsx` provides `ProgressBar`, `DateBlock`, and `Icon`; `src/shared/ui/index.ts` exports shared UI components; `src/styles/global.css` defines shared tokens and component styles; `src/App.tsx` contains the `/components` showcase. `package.json` defines typecheck, lint, build, and Cypress scripts.
- **Related completed tasks:** T51 (base UI actions/form fields) and T52 (layout/data display) are in `changes/completed/`; T52 records its merge after T51 and confirms use of T51 shared UI components. Their shared component and showcase patterns apply here.
- **Constraints / assumptions:** Frontend repo was clean on `master`; fetched `origin/master` successfully and fast-forward pull reported already up to date. Local `HEAD` and `origin/master` both resolve to `68200139e0c5dd56983027d07dd9fd18d59da60c`. Task specifies the Event card component on the Design System page (node `3:301`), the status-label/tone precedence behavior, and date format example. Treat this as frontend-only; no API interaction is specified.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the event card as a reusable presentation component in `shared/ui`, with the status decision and date formatting kept as pure, independently testable helpers. Reuse the existing `Badge`, `ProgressBar`, `DateBlock`, `Icon`, `Button`, and design tokens from T51/T52, while keeping event-card rendering free of API calls or business-side effects. Define the smallest typed event view model needed by the card and make the precedence rules explicit in `getEventBadge` so later screens can consume the same behavior.

### Steps

1. **UI/design verification:** inspect the Design System `Event card` component (`3:301`) and record the required layout, spacing, gradient cover, date chip, badge placement, typography, progress treatment, and action affordance. Confirm the existing T51/T52 components and token names that can be reused.
2. **Tests first (pure helpers):** add unit-test coverage for `getEventBadge` across Open, Few spots left, Full, Registered, Completed, and Canceled, including the threshold boundaries (10% and 5 places) and precedence cases such as Canceled over Full and Registered over Open. Add unit tests for `formatEventDate`, including the specified `Sat, 14 Nov · 18:00` output and a stable timezone/locale setup.
3. **Tests first (component):** add Cypress coverage for the showcase/card fixture, asserting the gradient cover and date chip, organization/title/location content, status badge label/tone, capacity progress and “X of Y spots taken” copy, accessible action name, and responsive/empty-safe rendering for optional image or end-time data. Run the tests before implementation and record the valid behavioral red result.
4. **UI implementation:** create the typed `EventCard`, `getEventBadge`, and shared `formatEventDate` modules under `repos/ubb-events-frontend/src/shared/ui/`; export them from the shared index. Map helper output to the existing `Badge` tones, reuse `ProgressBar`/`DateBlock`/`Icon` where their APIs fit, and keep status and threshold calculations in the pure helper rather than JSX.
5. **Styling and showcase:** add only the token-backed EventCard styles needed for the named Figma component, and extend `/components` with representative cards covering the key status and capacity states so the visual result and helper output can be reviewed.
6. **Verification:** run the focused helper/component tests, then `npm run typecheck`, `npm run lint`, `npm run build`, and the wider Cypress suite; compare the showcase against the Figma target and record any pre-existing failures separately.

### Files to create / modify

- `repos/ubb-events-frontend/src/shared/ui/EventCard.tsx` — card rendering and typed card props.
- `repos/ubb-events-frontend/src/shared/lib/eventStatus.ts` (or the existing shared-lib location) — `getEventBadge` and its event/registration status types.
- `repos/ubb-events-frontend/src/shared/lib/formatEventDate.ts` (or the existing shared-lib location) — shared date/time formatter.
- `repos/ubb-events-frontend/src/shared/ui/index.ts` — public exports.
- `repos/ubb-events-frontend/src/App.tsx` — `/components` EventCard showcase fixtures.
- `repos/ubb-events-frontend/src/styles/global.css` — EventCard styles using existing semantic tokens.
- `repos/ubb-events-frontend/cypress/e2e/event-card.cy.ts` — component/showcase behavior and accessibility coverage.
- `repos/ubb-events-frontend/src/shared/lib/*.test.*` or the repository’s established unit-test location — pure helper tests, pending confirmation of the available unit-test runner.

### Tests to write

- **Acceptance criterion 1 / card behavior:** helper tests cover every badge label/tone, threshold boundary, and precedence rule; Cypress verifies all EventCard content, progress text, View details action, and status presentation.
- **Acceptance criterion 2 / dependencies:** a pre-implementation check records that T51 and T52 are merged on frontend `master` before implementation begins.
- **Acceptance criterion 3 / state and accessibility:** Cypress checks accessible naming and keyboard activation for View details, progressbar semantics, badge text visibility, safe rendering at zero/full capacity, and cards with missing optional end time or cover image. Loading/empty/error states are not described for this presentational component; if the user intends them here, add explicit fixtures and acceptance assertions during Gate 1.
- **Regression:** typecheck, lint, build, and the complete Cypress suite.

### Risks / open questions

- The task does not define the exact event prop shape or enum spellings. Proposed default: a view model with event lifecycle status, start date/time, title, organization name, location, capacity, and registration count, plus `registrationStatus`; confirm names against the API DTOs before implementation.
- The task does not specify whether `registrationStatus` values include `CANCELLED` or whether a canceled event always wins over a registered user. The written precedence examples establish Canceled over Full and Registered over Open, but the complete precedence order should be confirmed at Gate 1.
- The repository currently exposes Cypress but no obvious unit-test runner. Confirm whether pure helpers should use a newly configured lightweight runner or be exercised through Cypress component/page tests; do not add a dependency without review.
- “Gradient cover” does not state whether a cover image is part of the data contract. Use a token-backed gradient fallback unless the Figma/API review identifies a required image prop.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/event-card.cy.ts` — both EventCard showcase scenarios.
- **Result:** `$env:CYPRESS_CACHE_FOLDER='C:\\Users\\irimi\\AppData\\Local\\Cypress\\Cache'; npx cypress run --browser chrome --spec cypress/e2e/event-card.cy.ts --reporter spec --config baseUrl=http://127.0.0.1:5174` produced 0 passing and 2 failing. Both tests timed out because `[data-testid="event-card-open"]` and `[data-testid="event-card-full"]` were absent.
- **Failure reason:** The tests compiled and ran against the existing `/components` page; the failures demonstrate the planned EventCard fixtures and behavior were missing.

### Green (after implementation)

- **Result:** Focused Cypress run with Chrome passed 2/2. `npm run typecheck`, `npm run lint`, and `npm run build` passed.
- **Supplemental post-implementation check:** Added `Completed` and exact 10% remaining-capacity fixtures/assertions (the boundary card has 90 of 100 spots taken). The focused Chrome Cypress run passed 2/2 again. These checks were added after implementation and are not pre-implementation red evidence.
- **Visual correction verification:** Updated the card to the inspected Figma target (360px width, calendar icon in the date chip, decorative cover shapes, 20px body padding, and text-plus-chevron CTA). After the changes, focused Chrome Cypress passed 2/2, `npm run typecheck` passed, `npm run lint` passed, and `npm run build` passed.
- **Exact asset verification:** Replaced CSS circle and text-chevron approximations with localized Figma SVGs `event-card-decor.svg`, `event-card-decor-2.svg`, and `event-card-chevron.svg`. Focused Chrome Cypress passed 2/2 after the asset update; typecheck, lint, and build also passed.
- **Final geometry verification:** Matched the inspected cover gradient (`156.95deg`, `#4D79A8` to `#223A5E`), decor positions, 6px capacity bar, 8px footer gap, shared seats/CTA footer row, and 14px body spacing. Focused Chrome Cypress passed 2/2; `npm run typecheck`, `npm run lint`, and `npm run build` passed.
- **Full suite of the affected repo:** Full Cypress execution reached the existing API-client URL mismatch (`api-client.cy.ts` expects `http://backend.test/api/organizations` while configured API base URL is the deployed backend URL); EventCard and other UI specs were executed before the runner stopped. This failure is outside T54.
- **Commit / push:** Commit 9541942 (T54: add event card and status logic) was created on T54-event-card-and-event-status. The first push attempt was rejected with HTTP 403; the user-requested retry succeeded and the branch now tracks origin/T54-event-card-and-event-status. GitHub CLI still reports the configured Vasile07 token is invalid, so no PR was created.

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

- **Decisions:** Draft — EventCard uses a typed frontend view model, token-backed gradient fallback, existing Badge/ProgressBar/Icon/Button primitives, local calendar asset, localized Figma decor and chevron SVGs, and UTC formatting for deterministic display. Status precedence is Canceled, Registered, Completed, Full, Few spots left, then Open. Pure helpers live in `src/shared/lib`; the repository has no unit-test runner, so helper behavior is covered through Cypress fixtures.
- **Alternatives considered:** Draft — A new unit-test dependency was not added because the user chose Cypress-only coverage and the existing frontend exposes Cypress as its test runner. The card follows the inspected Figma target with a 360px width, 20px body padding, text-and-chevron CTA, and optional cover image with a gradient fallback.
- **Limitations:** Draft — The full Cypress suite retains the pre-existing API-client base URL mismatch. Loading and error states are not part of this presentational component’s description. The unit-test requirement is recorded as a Cypress-only deviation.

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
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`











