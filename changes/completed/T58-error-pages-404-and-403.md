# T58 - Error pages: 404 and 403

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T58
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T58-error-pages-404-and-403`
- **PR link(s):** Frontend: https://github.com/Vasile07/ubb-events-frontend/pull/26
- **Tests:** `cypress/e2e/error-pages.cy.ts` (new); `cypress/e2e/routing-and-guards.cy.ts` (catch-all expectation updated); regression: `organization-workspace-shell.cy.ts`, `event-workspace-shell.cy.ts` unchanged

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 11 — Error pages: 404 and 403

**Description:** Implement Figma page `Student Screens`, frames `404 – Page not found` (48:876) and `403 – No access` (48:914): large faded code, title, description, decorative circles and two buttons (Back to discover, Go back / My organizations).
Use them as the router's catch-all 404 and as the target of `RequireOrgManager`/`RequireOrgMember` failures and API 403 responses on page-level queries.
Keep the navbar visible when the user is authenticated.
**Depends on:** T53 — Feedback components: toast, empty state, skeleton, error state; T56 — Routing and route guards; T57 — App shell and top navigation.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code** (frontend `master` at `f576b64`):
  - `src/App.tsx`:
    - the catch-all `<Route path="*">` currently does `<Navigate to="/events">`;
    - `RequireOrgMember` and `RequireOrgManager` render an inline `ForbiddenPage` (`data-testid="page-forbidden"`, plain "403 — Access denied") **without** `AppShell`, so the navbar disappears;
    - `authPage`, `memberPage`, `managerPage` and `eventPage` wrap the placeholders.
  - Page-level queries under `/org/:organizationId/...`:
    - `src/features/organizations/OrganizationWorkspace.tsx` (T68) loads the organization and maps an error to "Organization not found" (404) or "Could not load organization";
    - `src/features/events/EventWorkspace.tsx` (T72) loads the event the same way ("Event not found" / "Could not load event").

    Neither handles a 403. Every `/org/...` page renders inside these shells.
  - `src/shared/ui`: `Button` (Primary/Secondary, Md) and `AppShell` (navbar, T57).
  - `src/features/auth/types.ts`: `AuthMembership` has `organizationId`, `role` and an optional `organizationName`.
  - Existing tests:
    - `cypress/e2e/routing-and-guards.cy.ts` (T56) expects `/does-not-exist` → `/events`, which T58 changes. It also checks `page-forbidden` for guard failures.
- **Related completed tasks:**
  - T56 Routing and route guards;
  - T57 App shell;
  - T53 Feedback components;
  - T68 and T72, the workspace shells, owned by colleagues.
- **Figma** (session `5f98b591-…`, 2 reads: `404 – Page not found` 48:876 and `403 – No access` 48:914). Both frames share one layout:
  - the navbar, with no active link;
  - the page background `--bg-page`;
  - three decorative circles in `#D0E1F2`: 120px at (1040,130), 260px at (80,160), 320px at (1160,520). They are the same three SVGs in both frames, already downloaded;
  - a centred message with a 20px gap:
    - the code "404" / "403": 140/140 ExtraBold, `--accent-light` (#6E9ECC), letter-spacing −4px;
    - the title (H1 32/40);
    - the description: 16/24 `--text-secondary`, max 520px, centred;
    - two Md buttons with a 12px gap.

  The texts:
  - **404:**
    - title: "This page doesn’t exist";
    - description: "The page you are looking for was moved, deleted or never existed. Check the link or head back to the events.";
    - buttons: Primary "Back to discover", Secondary "Go back".
  - **403:**
    - title: "You don’t have access to this page";
    - description: "This area is only available to managers of the organization. Ask a manager of Societatea Hermes if you need access.";
    - buttons: Primary "Back to discover", Secondary "My organizations".
- **Constraints / assumptions:** tests stub `fetch`, as in the other specs.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Create `src/features/errors/ErrorPages.tsx`. It has one shared `ErrorPageLayout` (decor, code, title, description, actions) and two pages, `NotFoundPage` and `ForbiddenPage`. Use them:
- as the catch-all 404;
- in `RequireOrgMember` and `RequireOrgManager`, wrapped in `AppShell` so the navbar stays visible;
- when the workspace shells' page-level query returns a 403.

Decisions (Gate 1: user approved all six as proposed, "all as proposed gate 1 approved"):
1. **The 403 description depends on why access was refused.** The Figma text is the manager case:
   - **Manager-only route** (`RequireOrgManager`, user is a member but not a manager): "This area is only available to managers of the organization. Ask a manager of {organization name} if you need access." The name comes from the membership's `organizationName` when available, otherwise "the organization".
   - **Member route, not a member** (`RequireOrgMember`, or an API 403): "This area is only available to members of the organization. Ask to join it from the Organizations page if you need access." This is a new sentence following the Figma style.
2. **Buttons:**
   - "Back to discover" → `/events`.
   - "Go back" (404) → browser back (`navigate(-1)`), or `/events` when there is no previous in-app page.
   - "My organizations" (403) → `/organizations`.
3. **404 catch-all:**
   - an authenticated user sees the 404 inside `AppShell`, so the navbar is visible;
   - an anonymous user sees the same page without the navbar, and "Back to discover" then leads to login through the existing guard.

   Today's redirect to `/events` is replaced. The T56 test that expects that redirect is updated to expect the 404 page (a necessary regression update).
4. **API 403 on page-level queries:** `OrganizationWorkspace` and `EventWorkspace`, the T68/T72 shells that wrap every `/org/...` page, render `ForbiddenPage` (member variant) when their query fails with 403. Their existing 404 and error states stay unchanged. No other page has a page-level query that can return 403. Discover, My tasks, the profile and the public event and organization pages use endpoints that need authentication only.
5. **Decor:** the three Figma circle SVGs go to `public/ui-assets/error-decor-{120,260,320}.svg`. They are positioned absolutely as in the frame (scaled from the 1440px width), decorative (`aria-hidden`), and hidden below 900px so they don't cover the text.
6. **Test ids:** keep `page-forbidden` (used by T56 tests); add `page-not-found`. Accessibility: the code is decorative (`aria-hidden`), the title is the page `h1`, and the region keeps `role="alert"` for 403 as today.

### Steps

1. Create the frontend branch `T58-error-pages-404-and-403` from up-to-date `master` (after `git status`).
2. Write the tests first, `cypress/e2e/error-pages.cy.ts`, and update the T56 catch-all expectation. Minimum stubs: `NotFoundPage` and `ForbiddenPage` exported from `ErrorPages.tsx`, still rendering today's plain content.
3. Run them and record a behavioral red result.
4. Implement:
   - `ErrorPages.tsx` (layout plus the two pages and the variants);
   - the decor assets;
   - `App.tsx`: the catch-all; the guards wrapped in `AppShell`; the old inline `ForbiddenPage` removed;
   - the 403 branch in `OrganizationWorkspace.tsx` and `EventWorkspace.tsx`;
   - page-scoped CSS (`.error-page…`) with the T49 tokens.
5. Run to green, then the full Cypress suite, typecheck, lint and build. Do a visual check against both frames.

### Files to create / modify

- New: `src/features/errors/ErrorPages.tsx`, `public/ui-assets/error-decor-{120,260,320}.svg`, `cypress/e2e/error-pages.cy.ts`.
- Modify:
  - `src/App.tsx`;
  - `src/features/organizations/OrganizationWorkspace.tsx` and `src/features/events/EventWorkspace.tsx` (the 403 branch only);
  - `src/styles/global.css`;
  - `cypress/e2e/routing-and-guards.cy.ts` (the catch-all expectation only).

### Tests to write

- `error-pages.cy.ts`:
  - an unknown route shows the 404 (code, title, description, both buttons) with the navbar when authenticated;
  - an unknown route shows the 404 without the navbar when anonymous;
  - "Back to discover" → `/events`;
  - "Go back" returns to the previous page, or falls back to `/events` when opened directly;
  - a manager-only route for a member shows the 403 manager text with the organization name and the navbar;
  - a member route for a non-member shows the 403 member text with the navbar;
  - "My organizations" → `/organizations`;
  - an API 403 from the organization workspace query shows the 403 page;
  - an API 403 from the event workspace query shows the 403 page;
  - the decorative circles are hidden from assistive technology.
- Regression: the full suite, with the updated T56 catch-all test; the other `page-forbidden` checks stay as they are.

### Risks / open questions

- Decisions 1–6 above need confirmation. Decision 1 (the member-variant sentence) adds text that isn't in Figma.
- Decision 4 touches two colleagues' files (T68/T72): only a 403 branch is added, and their existing states and tests are kept.
- Decision 3 changes T56 behaviour (no more silent redirect for unknown URLs), so one T56 test is updated.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:**
  - `cypress/e2e/error-pages.cy.ts` (new), 11 tests:
    - `shows the 404 page with the navbar for an unknown route when authenticated`
    - `shows the 404 page without the navbar for an unknown route when anonymous`
    - `goes to discover from the 404 page`
    - `goes back to the previous page, or to discover when opened directly`
    - `shows the manager 403 with the organization name and the navbar on a manager-only route`
    - `falls back to 'the organization' when the organization name is unknown`
    - `shows the member 403 with the navbar on a member route for a non-member`
    - `goes to the organizations list from the 403 page`
    - `shows the 403 page when the organization workspace query is forbidden by the API`
    - `shows the 403 page when the event workspace query is forbidden by the API`
    - `keeps the decorative circles hidden from assistive technology`
  - `cypress/e2e/routing-and-guards.cy.ts` (T56, updated by decision 3): `renders public placeholders and redirects the default and unknown paths` now expects `/does-not-exist` to stay on the URL and show `page-not-found`.
  - Stubs used so the tests compile:
    - `src/features/errors/ErrorPages.tsx` exports `NotFoundPage` (renders nothing) and `ForbiddenPage`, which is the old plain "403 — Access denied" content moved out of `App.tsx` and still used by the guards;
    - the catch-all still redirects to `/events`;
    - the decor SVGs are in `public/ui-assets/`.
- **Command:** `npx cypress run --spec "cypress/e2e/error-pages.cy.ts,cypress/e2e/routing-and-guards.cy.ts" --browser electron --reporter spec` (Vite on `127.0.0.1:5173`, warmed up first; `npm run typecheck` exit 0).
- **Result:**
  - `error-pages.cy.ts`: **0 passing, 11 failing.**
    - Unknown routes are redirected: the pathname is `'/events'` instead of `'/this/does/not/exist'` (authenticated), and `'/login'` instead of `'/nowhere'` (anonymous).
    - "Back to discover", "Go back" and "My organizations" buttons are not found, nor are `page-not-found`, `error-code` or `.error-page-decor`.
    - The 403 text is "403 — Access denied…" instead of the manager or member sentence.
    - The API-403 cases show no `page-forbidden`.
  - `routing-and-guards.cy.ts`: **9 passing, 1 failing.** The updated catch-all test can't find `page-not-found`. The other 9 T56 tests (including every `page-forbidden` guard check) still pass.
- **Failure reason:** behavioral. The Figma 404 and 403 pages, the catch-all 404, the navbar around guard failures, the manager and member 403 texts, the button navigation and the API-403 handling in the workspace shells are all missing. There were no compilation errors, and the stubs, routes and session restore worked.

### Green (after implementation)

- **Result:** the same command gives **11 passing, 0 failing** (`error-pages.cy.ts`) and **10 passing, 0 failing** (`routing-and-guards.cy.ts`).
- **Full suite of the affected repo:** `npx cypress run --browser electron --reporter spec` gives **146 of 148 passing, 25 of 26 specs.**
  - The only failures are the 2 tests in `organizations-list.cy.ts` (T65): `renders cards, membership actions, and filters by name` and `handles loading and no-match states`.
  - They assert that a `hidden` marker span is visible, so they **already fail on `origin/master`**: this was checked in a clean checkout on 2026-11-14. T58 doesn't touch T65's code.
  - All other specs pass, including `organization-workspace-shell`, `event-workspace-shell`, `create-event-flow` and the remaining T56 guard tests.
- `npm run typecheck`, `npm run lint` and `npm run build` all exit 0.
- **Visual check:**
  - Method: a throwaway Cypress spec (deleted afterwards) took screenshots at 1440×900: the authenticated 404, the manager 403 for a member of "Societatea Hermes", the anonymous 404 and a 375px mobile 404. These were compared with the screenshots from the two Figma reads.
  - Both pages match the frames:
    - the navbar is visible for an authenticated user;
    - the faded 140px code is in `--color-blue-600`;
    - the title, the centred description and the Primary/Secondary buttons match;
    - the three circles sit in the frame positions.
  - On mobile the circles are hidden and the code shrinks.
  - Vite was stopped afterwards.

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

<!-- DRAFT written by the implementer. Verify and correct before using it as a study note. -->
- **Decisions:**
  - `src/features/errors/ErrorPages.tsx` has one shared `ErrorPageLayout` (three decorative circles, code, `h1` title, description, actions) and two pages:
    - `NotFoundPage` (`page-not-found`);
    - `ForbiddenPage` (`page-forbidden`, `role="alert"`, with the `manager` and `member` variants via `forbiddenDescription`).

    The old inline `ForbiddenPage` in `App.tsx` was removed.
  - **Catch-all:** `<Route path="*">` renders `CatchAllNotFound`, which shows the 404 inside `AppShell` when authenticated and without it when anonymous. It replaces the old silent redirect to `/events`, and the T56 test was updated to match (Gate 1 decision 3).
  - **Guards:**
    - `RequireOrgMember` and `RequireOrgManager` render the 403 inside `AppShell`, so the navbar stays visible;
    - `RequireOrgManager` uses the manager text when the user is a member but not a manager, naming the organization from `membership.organizationName` and falling back to "the organization";
    - non-members and invalid ids get the member text.
  - **API 403:** `OrganizationWorkspace` and `EventWorkspace` render `ForbiddenPage` (member) when their page-level query fails with 403. Their 404, error and retry states are unchanged.
  - **Buttons:**
    - "Back to discover" → `/events`;
    - "Go back" → `navigate(-1)`, or `/events` when the router location key is `"default"` (no previous in-app entry);
    - "My organizations" → `/organizations`.
  - **Styling:**
    - page-scoped `.error-page…` CSS with the T49 tokens. The faded code uses `--color-blue-600` (#6E9ECC), because the Figma variable `--accent-light` doesn't exist in the code tokens;
    - the three Figma circle SVGs are in `public/ui-assets/error-decor-{120,260,320}.svg`, are `aria-hidden` with empty `alt`, and are hidden below 900px;
    - the numeric code is `aria-hidden`, so screen readers read the title.
- **Alternatives considered:**
  - Keeping the redirect for unknown URLs: rejected, because the task asks for a 404 page.
  - One generic 403 text: rejected at Gate 1, because the reason is shown instead.
  - Handling 403 in every page's own query: unnecessary, because every `/org/...` page renders inside the two workspace shells, whose query fails first.
  - Adding an `--accent-light` token: not needed, because the existing palette token has the same value.
- **Limitations:**
  - The backend profile (`MembershipResponse`) has no organization name, so in practice the manager text usually says "the organization". The name appears only if a future profile response includes `organizationName`.
  - An API 403 from the event query renders inside the organization workspace layout, so the sidebar stays visible, because the organization query succeeded.
  - API 404s keep the existing "not found" states of the workspace shells. The full-page 404 is used only for unknown routes, as the task describes.
  - **To report to the team:** the 2 tests in `organizations-list.cy.ts` (T65) already fail on `master`.

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
- [x] Moved to `changes/completed/`


