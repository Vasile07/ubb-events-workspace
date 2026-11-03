# T52 - Base UI components: layout and data display

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T52
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T52-base-ui-components-layout-and`
- **PR link(s):** [Frontend PR #5](https://github.com/Vasile07/ubb-events-frontend/pull/5) — merged.
- **Tests:** `cypress/e2e/base-ui-components-layout.cy.ts` (4 tests); T51 regression `cypress/e2e/base-ui-components.cy.ts` (3 tests); full Cypress suite; frontend `typecheck`, `lint`, and `build`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 5 — Base UI components: layout and data display

**Description:** Implement the shared building blocks used by many screens:
- `AuthLayout` (split layout used by Login and Create account: left brand panel with gradient, headline, three feature rows and decorative circles; right form panel; frames `Login` 13:10 and `Create account` 13:62). Both auth screens use it, so they stay independent from each other
- `Tabs` in two styles (pill/segmented as in frame `My registrations` 14:355, and underline as in the event workspace frame 15:412)
- `Table` (header row, rows, bottom dividers, fixed and fill columns, as in `Org – Events` 15:10)
- `Modal` (centered card, `Shadow/Raised`, dark backdrop as in `Org – Delete event confirmation` 16:1061, focus trap, Esc to close)
- `InfoBanner` (info and warning tones)
- `Avatar` (initials from first and last name, sizes 36/48/80)
- `ProgressBar` (capacity)
- `DateBlock` (month + day tile)
- `InfoTile` (icon box + label + value, as in `Event details` 13:417)
- `PageHeader` (title, subtitle, right slot)
- `Icon` wrapper (calendar, pin, clock, users, building, bell, check, x, plus, send, lock, trash, edit, info, alert, chevrons, arrow-left, search, ticket, megaphone, tasks, shield, logout).

**Depends on:** T49 — Design tokens and global styles; T51 — Base UI components: actions and form fields.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] T49 is available on `master`, T51's reviewed implementation is the base for T52 development, and T52 is rebased onto `master` after T51 merges and before T52's PR is created.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx`, `src/main.tsx`, `src/styles/global.css`, `src/shared/ui/` (no shared UI files are present on `master` yet); `package.json` provides `dev`, `build`, `lint`, `typecheck`, and Cypress scripts.
- **Related completed tasks:** `changes/completed/T49-design-tokens-and-global-styles.md` (merged into frontend `master`); `changes/completed/T51-base-ui-components-actions-and.md` (merged into frontend `master` as PR #4).
- **Constraints / assumptions:** T52 targets the frontend repository and depends on T49 and T51. T51 merged to frontend `master` at `dd567ee`; T52 was rebased onto that updated `master` before PR creation. T52 merged to frontend `master` in PR #5 (merge commit `6820013`).

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Build the shared data-display and layout primitives in `src/shared/ui` on top of the T49 tokens and the reviewed T51 action/form components. Keep components presentational and composable: props control labels, slots, variants, sizes and states; no API calls or screen-specific business rules belong in this task. Add a `/components` showcase route as the stable Cypress integration surface and accessibility review target. Create the T52 branch/worktree from the T51 branch so the implementation can compile against T51, then rebase T52 onto `master` after T51 merges and resolve any conflicts before opening T52's PR.

### Steps

1. **Branch/repository setup:** After Gate 1, verify the frontend worktree is clean, create `T52-base-ui-components-layout-and` from `T51-base-ui-components-actions-and`, and keep T51's reviewed implementation as the dependency base. Before PR creation, rebase the T52 branch onto updated `master` after T51 has merged; record any conflict resolution in the task evidence.
2. **UI tests first:** Add Cypress coverage for the `/components` showcase before implementing the new components. Cover rendering and public props for every listed component, both Tabs styles, Table column behavior, Modal close/focus behavior, Avatar initials and sizes, ProgressBar values, DateBlock, InfoTile, PageHeader, InfoBanner tones, and the complete Icon name set. Add keyboard and accessibility assertions for focus trap, Escape, dialog semantics, tab semantics, table headers, labels, and status text; capture a behavioral red result.
3. **UI primitives:** Implement `AuthLayout`, `Tabs`, `Table`, `Modal`, `InfoBanner`, `Avatar`, `ProgressBar`, `DateBlock`, `InfoTile`, `PageHeader`, and the typed `Icon` wrapper in `src/shared/ui`, reusing T49 semantic tokens and T51 controls/icons where appropriate. Keep Modal focus trapping and Escape handling self-contained, and expose loading, empty, warning/info, disabled, and capacity boundary states through props or slots where the component requires them.
4. **UI styling and accessibility:** Add token-backed styles in the existing global/component stylesheet layer for the Figma layouts and responsive behavior. Use semantic HTML and ARIA roles/relationships, visible focus treatment, keyboard navigation, initials fallback, deterministic progress clamping, and icon labels for decorative versus meaningful icons. Avoid introducing a new dependency unless the existing T51 implementation cannot provide the required icon assets.
5. **UI demo:** Add a review-only `/components` route that renders each component and its documented variants/states with representative content. Keep the existing Foundations route intact and do not add backend/API behavior.
6. **Verification and dependency handoff:** Run the focused Cypress spec, then `npm run typecheck`, `npm run lint`, `npm run build`, and the wider Cypress suite. Compare the showcase to the named Figma frames (`Login` 13:10, `Create account` 13:62, `My registrations` 14:355, event workspace 15:412, `Org – Events` 15:10, delete confirmation 16:1061, `Event details` 13:417). After T51 merges, rebase and rerun the affected checks before T52's PR.

### Files to create / modify

- `repos/ubb-events-frontend/src/shared/ui/AuthLayout.tsx`
- `repos/ubb-events-frontend/src/shared/ui/Tabs.tsx`
- `repos/ubb-events-frontend/src/shared/ui/Table.tsx`
- `repos/ubb-events-frontend/src/shared/ui/Modal.tsx`
- `repos/ubb-events-frontend/src/shared/ui/InfoBanner.tsx`
- `repos/ubb-events-frontend/src/shared/ui/Avatar.tsx`
- `repos/ubb-events-frontend/src/shared/ui/ProgressBar.tsx`
- `repos/ubb-events-frontend/src/shared/ui/DateBlock.tsx`
- `repos/ubb-events-frontend/src/shared/ui/InfoTile.tsx`
- `repos/ubb-events-frontend/src/shared/ui/PageHeader.tsx`
- `repos/ubb-events-frontend/src/shared/ui/Icon.tsx`
- `repos/ubb-events-frontend/src/shared/ui/index.ts`
- `repos/ubb-events-frontend/src/App.tsx` or a dedicated components showcase route/page
- `repos/ubb-events-frontend/src/styles/global.css` and/or component styles for token-backed layout and states
- `repos/ubb-events-frontend/cypress/e2e/base-ui-components-layout.cy.ts`

### Tests to write

- **Description / component coverage:** Cypress checks on `/components` find and exercise AuthLayout, both Tabs variants, Table headers/rows/fixed/fill columns, Modal, InfoBanner tones, Avatar sizes/initials, ProgressBar, DateBlock, InfoTile, PageHeader slots, and every Icon name listed in the task.
- **Description / behavior and states:** assertions cover active tab changes, empty table content, progress values at 0/full/over-capacity boundaries, InfoBanner info/warning styling, missing-name avatar fallback, DateBlock month/day output, modal open/close, Escape, backdrop behavior, and right-slot rendering.
- **Acceptance criterion 2 / dependency strategy:** repository evidence verifies T49 is on `master`, T52 was based on T51, and the final pre-PR check verifies the T52 branch is rebased onto the updated `master` after T51 merges.
- **Acceptance criterion 3 / accessibility:** keyboard tests verify tablist/tab semantics and arrow or activation behavior, dialog role/name/focus return and focus trap, table header associations, visible focus, meaningful icon labels, and no duplicate interactive controls. The showcase includes loading, empty, warning/info, disabled, and validation/error examples where applicable.
- **Regression:** run the existing Foundations/home and T51 component specs plus typecheck, lint, build, and the wider Cypress suite.

### Risks / open questions

- T51 is not merged yet, so the implementation branch must remain based on the reviewed T51 branch and be rebased once T51 lands; conflicts in shared UI exports or global styles are expected to be resolved before the PR.
- The task names icon symbols but does not prescribe an icon package or asset source. Prefer the reviewed T51 icon mechanism; adding a dependency requires user review if it is not already available.
- The Modal focus trap and Tabs keyboard interaction need browser-level coverage because the current repository has Cypress but no component test runner.
- The Figma references identify visual targets; exact copy for the showcase can remain representative as long as the component API and states are exercised.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/base-ui-components-layout.cy.ts` — planned component/variant coverage and keyboard/modal behavior.
- **Result:** `$env:CYPRESS_CACHE_FOLDER='C:\\Users\\irimi\\AppData\\Local\\Cypress\\Cache'; npx cypress run --spec cypress/e2e/base-ui-components-layout.cy.ts --reporter spec --config baseUrl=http://127.0.0.1:5173` — 0 passing, 2 failing. `renders every planned component and variant` could not find `[data-testid="auth-layout-demo"]`; `supports keyboard tabs and modal escape behavior` could not find `[data-testid="tabs-pill"] [role="tab"]`.
- **Failure reason:** The Cypress tests compiled and executed against the existing `/components` route, which does not yet render the T52 layout/data-display components. These are behavioral failures caused by the missing planned functionality, not setup failures.

### Green (after implementation)

- **Result:** Final focused command `$env:CYPRESS_CACHE_FOLDER='C:\\Users\\irimi\\AppData\\Local\\Cypress\\Cache'; npx cypress run --spec cypress/e2e/base-ui-components-layout.cy.ts --reporter spec --config baseUrl=http://127.0.0.1:5173` passed 4/4. After rebasing onto T51-merged `master`, T52 and T51 specs passed 7/7 together. `npm run typecheck`, `npm run lint`, and `npm run build` passed after retrying a transient Windows output-directory cleanup error.
- **Commit and push:** Commit `91f7779` (`T52: add layout and data display components`) was rebased onto `master` at `dd567ee` and pushed to `origin/T52-base-ui-components-layout-and`.
- **Full suite of the affected repo:** `$env:CYPRESS_CACHE_FOLDER='C:\\Users\\irimi\\AppData\\Local\\Cypress\\Cache'; npx cypress run --reporter spec --config baseUrl=http://127.0.0.1:5173` ran all 5 specs: 22/23 passed. The existing API-client test `calls GET /api/organizations through the configured base URL with a Bearer token` failed because the configured URL is `https://ubb-events-backend.onrender.com/api/organizations` while it expects `http://backend.test/api/organizations`; this is outside T52.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The workflow's dependency timing would make us wait for T51 to merge. | Plan T52 from the T51 branch, then rebase T52 from `master` before creating its PR. | User said T51 is implemented but not merged and wants to continue using a branch from that branch. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Implemented typed React primitives in `src/shared/ui/LayoutComponents.tsx`, reused T49 tokens and T51 controls, and kept `/components` as the Cypress showcase. Modal traps focus, closes on Escape/backdrop, and restores focus; progress values are clamped to capacity. Figma SVGs are local assets for icons present in the named frames; remaining requested icons use distinct inline SVG paths because no standalone icon set is present in the referenced design library.
- **Alternatives considered:** Kept T51's showcase in place to preserve its regression surface. Added no icon dependency; the named frames did not supply a full set, and the existing plan favored avoiding a new dependency.
- **Limitations:** The full suite retains the pre-existing API-client URL mismatch described in the Test log. The merged branch includes the rebased T51 dependency.

> **TDD deviation:** After the initial implementation, review added delayed checks for boundary/empty states, icon coverage, Figma auth layout content, and arrow-key tab behavior. The supplemental run produced 2 passing and 2 failing tests (tabs did not move focus and the auth headline selector was absent); these were not pre-implementation red evidence. After the fixes, the final focused run passed 4/4. The original valid pre-implementation red run remains the evidence for the red status item.

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
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`


