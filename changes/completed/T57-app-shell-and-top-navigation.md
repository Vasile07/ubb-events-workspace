# T57 - App shell and top navigation

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T57
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** N/A
- **Use case(s):** N/A
- **Workflow:** App shell and top navigation.
- **Drive doc / ADR:** [Google Drive project documentation](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban) (task-specific section not identified).
- **Repo(s):** frontend
- **Branch:** `T57-app-shell-and-top-navigation`
- **PR link(s):** [Frontend PR #10](https://github.com/Vasile07/ubb-events-frontend/pull/10) — reviewed and merged.
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/app-shell-and-navigation.cy.ts`; frontend typecheck, lint, build, and full Cypress suite

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 10 — App shell and top navigation

**Description:** Build the authenticated layout with the `Navbar` from page `Design System` (3:180): logo, links Discover / My registrations / My tasks / Organizations / Workspace, notification bell and the user chip (avatar initials + `fullName`) linking to `/profile`.
The active link follows the current route (the `None` variant is used on the profile page).
The Workspace link must appear only when the user has at least one membership: with one organization it goes to `/org/:organizationId/events`, with several it opens a small picker.
The bell can stay non-functional (no notifications in scope) but must render.
Content is centered with 80px horizontal padding on desktop.
**Depends on:** T52 — Base UI components: layout and data display; T55 — Authentication context provider; T56 — Routing and route guards.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` defines the authenticated routes, `RequireAuth`, and placeholder content; `src/features/auth/AuthProvider.tsx` exposes `user`, `status`, and memberships; `src/features/auth/types.ts` defines `fullName` and membership `organizationId`/`role` with optional `organizationName`; `src/shared/ui/LayoutComponents.tsx` and `src/shared/ui/index.ts` contain reusable `Avatar`, `Icon`, and layout primitives; `src/styles/global.css` contains shared tokens and UI styles; `cypress/e2e/routing-and-guards.cy.ts` covers the current route guards.
- **Related completed tasks:** `changes/completed/T52-base-ui-components-layout-and.md` (merged frontend PR #5) and `changes/completed/T55-authentication-context-provider.md` (auth provider/context). T56 route and guard implementation is now on frontend `master` (merge commit `d6b5742`); no completed T56 task file was found.
- **Constraints / assumptions:** Frontend repo is clean on up-to-date `master`; task targets frontend only with branch `T57-app-shell-and-top-navigation`. Authenticated routes currently render placeholders. Memberships provide organization IDs and roles; organization names are optional and the T55 notes defer their data source. Navbar must use the specifically named Figma `Design System` frame `3:180`; do not infer a substitute frame.
## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add a presentational authenticated app shell around the existing guarded routes. The shell will own the responsive top navigation, route-aware active state, membership-aware Workspace affordance, and centered page content while consuming `useAuth()` for user and membership data. Reuse the T52 `Avatar` and `Icon` primitives and React Router links/navigation rather than duplicating route or authentication rules. Keep the notification bell visual only, and keep the shell outside login/register and other public routes.

The Workspace behavior will be explicit: hide it for zero memberships, link directly to the sole membership's events route, and open a compact keyboard-accessible picker for multiple memberships. The picker will use `organizationName` when available and display `Organization {organizationId}` when it is absent. This is a display fallback only; it does not change the membership data contract.

### Steps

1. **UI tests first (frontend):** Add Cypress coverage for the authenticated shell before implementation. Stub the authenticated profile and visit representative guarded routes to verify the logo, all required links, bell, profile chip/link, desktop content padding, and shell presence only on authenticated pages.
2. **Shell and navigation UI (frontend/UI):** Create the app-shell/navbar components using semantic `header`, `nav`, links, buttons, and a labeled notification control. Reuse `Avatar`, `Icon`, and existing tokens/styles; expose the user initials and `fullName` from `useAuth()` and link the chip to `/profile`.
3. **Route state (frontend/UI):** Determine active navigation from the current location, including nested event and organization routes, and apply the required `None`/inactive variant on `/profile`. Keep navigation through React Router so guards and redirects remain centralized in `App.tsx`.
4. **Workspace membership behavior (frontend/UI):** Read memberships from `useAuth()`. Hide Workspace with no memberships; use the membership ID for the single-membership `/org/:organizationId/events` link; for several memberships render a small menu/popover with one link per organization, keyboard focus handling, Escape/blur closure, and accessible names. Do not add API calls or alter membership business rules.
5. **Route integration and responsive styling (frontend/UI):** Wrap authenticated route content with the shell while leaving public/authentication routes and existing guard components intact. Add token-backed navbar and content-container styles matching Figma `Design System` frame `3:180`, including 80px horizontal desktop padding and a responsive smaller-padding rule.
6. **Verification:** Run the focused Cypress spec and then the affected frontend typecheck, lint, build, and full Cypress suite. Record the pre-implementation behavioral red result and post-implementation green results in this task file.

### Files to create / modify

- `repos/ubb-events-frontend/src/shared/ui/AppShell.tsx` (or equivalent shell/navbar component and Workspace picker).
- `repos/ubb-events-frontend/src/App.tsx` (wrap authenticated route elements with the shell without changing guard policy).
- `repos/ubb-events-frontend/src/styles/global.css` (navbar, picker, active states, centered content and responsive padding).
- `repos/ubb-events-frontend/cypress/e2e/app-shell-and-navigation.cy.ts` (focused behavior and accessibility coverage).
- `repos/ubb-events-frontend/src/shared/ui/index.ts` only if the new component is exported from the shared UI barrel.

### Tests to write

- **Acceptance criterion 1 / description behavior:** Cypress verifies authenticated rendering of logo, Discover, My registrations, My tasks, Organizations, Workspace when eligible, bell, and profile chip with initials/full name and `/profile` destination.
- **Route behavior:** Tests cover active state for `/events`, nested `/events/:eventId`, `/my-registrations`, `/my-tasks`, `/organizations` and nested organization pages, `/org/:organizationId/events`, plus the profile `None` variant.
- **Workspace rules:** Tests cover zero memberships (hidden), one membership (direct events link), and multiple memberships (picker with each membership link, close/reopen, keyboard and Escape behavior). Include a membership with `organizationName` and one without it; assert the missing-name membership displays `Organization {organizationId}`.
- **Acceptance criterion 2 / dependencies:** Verify the test run is based on merged T52, T55, and T56 code on frontend `master`; do not implement until those prerequisites remain available.
- **Acceptance criterion 3 / accessibility and layout:** Assert semantic navigation/link names, accessible bell label, profile link, picker focus/keyboard behavior, active-state semantics, and the desktop content container's 80px horizontal padding (with a responsive viewport check).

### Risks / open questions

- Some memberships may not include `organizationName`; per the user-approved plan, display `Organization {organizationId}` as the deterministic fallback.
- The task references the Figma `Design System` frame `3:180`; that exact frame was read and compared during implementation, and the navbar spacing, typography, pill states, logo, bell, and profile treatment were aligned to it.
- Existing placeholder pages are guarded independently. The shell integration must avoid changing redirect/authorization behavior or wrapping login/register routes.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/app-shell-and-navigation.cy.ts` (3 tests), written before implementation.
- **Result:** `npx cypress run --browser electron --spec cypress/e2e/app-shell-and-navigation.cy.ts --reporter spec` with `VITE_API_URL=http://localhost:8080` — 0 passing, 3 failing.
- **Failure reason:** Each test timed out because the planned `[data-testid=app-shell]` / Workspace navigation elements did not exist before the shell implementation. The initial runner lacked a Cypress binary and a VITE API URL; after installing Cypress and starting Vite with the configured URL, the behavioral red run was captured.

### Green (after implementation)

- **Result:** Focused `app-shell-and-navigation.cy.ts` passed: 3 passing, 0 failing.
- **Full suite of the affected repo:** Final `npx cypress run --browser electron --reporter dot` after the Figma-informed visual corrections completed across all 9 frontend specs with 41 passing and 0 failing tests. The focused shell spec remained 3/3 passing. `npm run typecheck`, `npm run lint`, and `npm run build` passed after the visual corrections.

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

- **Decisions:** The authenticated shell is composed around each existing guarded placeholder route, preserving T56 guard ownership. Workspace is hidden for zero memberships, a direct link for one membership, and a keyboard-accessible menu for several; missing organization names use the user-approved `Organization {organizationId}` fallback. Navigation active state is derived from the current pathname; profile uses the `None` variant marker.
- **Alternatives considered:** A global shell around the entire router was not used because public login/register routes must remain outside the authenticated layout. No new API call was added because membership data already comes from T55's auth context.
- **Limitations:** The canonical Figma frame was readable through the design-to-code connector, but its generated bell SVG is a temporary asset; the implementation reuses the existing T52 `Icon` primitive instead of storing a temporary URL. No static image asset was required for this text-and-icon navbar.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed (`fc096e3`)
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
