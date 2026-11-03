# T51 - Base UI components: actions and form fields

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T51
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T51-base-ui-components-actions-and`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/4
- **Tests:** `cypress/e2e/base-ui-components.cy.ts`; `npm run typecheck`; `npm run lint`; `npm run build`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 4 — Base UI components: actions and form fields

**Description:** Implement the reusable components from page `Design System`, section `Components` (2:2):
- `Button` (variants Primary, Secondary, Ghost, Danger; sizes Md 44px and Sm 34px; node 2:20) with `loading` and `disabled` states and optional leading icon
- `Badge` (tones Neutral, Info, Success, Warning, Danger, Brand with the leading dot; node 2:40)
- `TextField` (states Default, Focus, Error; label, helper/error text; node 2:57) plus a `TextArea` variant with the same look and configurable height
- `Select` styled like TextField with a chevron
- `SearchField` (pill, search icon; node 2:59)
- `FilterChip` (Default/Active with chevron; node 2:72)

All must forward refs, support keyboard focus (focus ring uses `border/focus`), `aria-invalid` and `aria-describedby`. Add a demo page (or Storybook) showing every variant.
**Depends on:** T49 — Design tokens and global styles.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` currently contains the design foundations page; `src/styles/global.css` defines palette, semantic color, spacing, radius, and typography tokens. `package.json` provides `typecheck`, `lint`, `build`, and Cypress scripts.
- **Related completed tasks:** `changes/completed/T49-design-tokens-and-global-styles.md` establishes the design tokens T51 depends on. T49 is merged in frontend `master` via merge commit `db11590` (PR #2), containing T49 commit `4cc6488`; only the task checklist metadata is stale.
- **Constraints / assumptions:** Frontend repository is clean on `master` and its current commit matches `origin/master` (`d6980d7`); the previous fetch failed, so this equality is based on existing local refs. Figma session `a7f11753-8f82-4968-a522-78a4ad957b20` inspected Design System / Components (`2:2`) and contexts/screenshots for Button (`2:20`), Badge (`2:40`), Text field (`2:57`), Search field (`2:59`), and Filter chip (`2:72`). Reference sizes include Button 44/34px, TextField control 48px, SearchField 52px, and FilterChip 42px; TextField uses 12px radius, SearchField and FilterChip use pill radius. The section has no explicit Select or TextArea node; follow the written spec that extends TextField styling. The Figma contexts provide SVG dot, search, and chevron assets; keep any used assets local and do not retain temporary Figma URLs. T49 is merged in frontend `master` via `db11590` (PR #2), despite its task checklist not recording the PR or merge. `context/project-context.md` says frontend run/test commands will be established later, though the current frontend `package.json` defines the scripts listed above.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Build the controls as small, typed React components in `shared/ui`, using the semantic tokens from T49 and native form/button behavior wherever possible. Match the inspected Figma sizing: Button 44px/34px (Md/Sm), TextField controls 48px high with 12px radius, SearchField 52px high with a pill radius, and FilterChip 42px high with a pill radius; Badge includes its leading dot. Keep state and interaction logic local to the presentation layer: variants and sizes are explicit props, labels and messages generate stable relationships for assistive technology, and all ref-bearing controls use `forwardRef`. Add a dedicated demo route that renders every documented variant and state so the implementation can be reviewed visually and exercised through Cypress. Select and TextArea reuse the TextField visual treatment because no separate nodes exist in the inspected component section. Confirm that T49 has been merged before implementation.

### Steps

1. **UI/design verification:** confirm the T49 prerequisite is merged; use Figma session `a7f11753-8f82-4968-a522-78a4ad957b20` and the inspected nodes to verify dimensions, spacing, colors, icons, and state styling. Preserve the localizable SVG dot/search/chevron assets locally and do not leave temporary Figma URLs in source. Record any material differences from the written specification before coding.
2. **UI tests first:** add Cypress coverage for the demo route and accessibility contract. Cover each Button variant/size plus loading and disabled behavior, Badge tones and dot, TextField/TextArea default/error relationships, Select and SearchField semantics, FilterChip default/active behavior, keyboard focus rings, forwarded focus, `aria-invalid`, and `aria-describedby`.
3. **UI component implementation:** create typed components under `repos/ubb-events-frontend/src/shared/ui/` for `Button`, `Badge`, `TextField`, `TextArea`, `Select`, `SearchField`, and `FilterChip`. Use `forwardRef`, native props, generated IDs where needed, explicit loading/disabled handling, and token-based CSS classes. Ensure focus styles use `--border-focus` and error/helper content is associated with its control. Implement Select and TextArea as TextField-style extensions, not as separate Figma-derived variants.
4. **UI styling:** add scoped component styles (in the existing global stylesheet or a component stylesheet consistent with the repo) for Button 44/34px, TextField 48px/12px radius, SearchField 52px pill, FilterChip 42px pill, Badge leading dot, variants, tones, chevrons, leading icons, focus rings, disabled/loading states, and configurable TextArea height. Use the inspected SVG assets locally, avoid hard-coded colors, and reuse T49 tokens.
5. **UI demo:** add a route/page reachable from the existing app that shows every component variant and state with representative labels, helper/error text, icons, options, and interactive controls. Keep it as review/demo plumbing only; no backend behavior is needed.
6. **Verification:** run the focused Cypress spec, then `npm run typecheck`, `npm run lint`, `npm run build`, and the wider Cypress suite. Visually compare the demo to the Figma reference and fix only in-scope mismatches.

### Files to create / modify

- `repos/ubb-events-frontend/src/shared/ui/Button.tsx`
- `repos/ubb-events-frontend/src/shared/ui/Badge.tsx`
- `repos/ubb-events-frontend/src/shared/ui/TextField.tsx` (including `TextArea`)
- `repos/ubb-events-frontend/src/shared/ui/Select.tsx`
- `repos/ubb-events-frontend/src/shared/ui/SearchField.tsx`
- `repos/ubb-events-frontend/src/shared/ui/FilterChip.tsx`
- `repos/ubb-events-frontend/src/shared/ui/index.ts` (shared exports, if needed by the existing structure)
- `repos/ubb-events-frontend/src/App.tsx` or a dedicated demo page/route file for the component showcase
- `repos/ubb-events-frontend/src/styles/global.css` and/or component stylesheet(s) for token-based component styles
- `repos/ubb-events-frontend/cypress/e2e/base-ui-components.cy.ts`

### Tests to write

- **Acceptance criterion 1:** demo-route Cypress checks find every named component and assert Button variants/sizes, Badge tones, field/select/search/chip rendering, loading/disabled/error states, and the configured TextArea height; focused checks also verify no required variant is omitted.
- **Acceptance criterion 2:** a pre-implementation repository/task check records T49's merged status; implementation must stop before code changes if its PR is not merged.
- **Acceptance criterion 3:** Cypress keyboard and accessibility checks tab through controls and assert focus visibility, forwarded focus on the underlying input/select/textarea, `aria-invalid` values, and `aria-describedby` links to helper/error text. Include disabled/loading interaction checks and error-state messaging.
- **Edge cases:** empty helper/error text, disabled controls, loading Button without duplicate activation, an invalid field with both helper and error content, custom TextArea height, and FilterChip state changes.

### Risks / open questions

- T49 is merged in frontend `master` via `db11590` (PR #2), containing commit `4cc6488`; its task file still has stale PR/Gate 3 checklist metadata that should be corrected for traceability, but it does not block T51.
- Figma contexts are available for the named component nodes, but Select and TextArea have no separate nodes in the Components section; they should follow the written instruction to extend TextField styling.
- The repository has Cypress but no component-testing library; the plan therefore uses the demo route as the integration surface. If a component test runner is introduced later, the same behavior cases can be moved there without changing the public component API.
- The Figma contexts expose localizable SVG dot/search/chevron assets. Materialize approved local assets through the Figma workflow and verify that no temporary Figma URL remains in source. The task does not prescribe an icon package; ask before adding a new dependency if the existing project has no suitable option.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/base-ui-components.cy.ts` — variant coverage, accessibility/focus relationships, disabled/loading behavior, TextArea height, and FilterChip state.
- **Result:** `npx cypress run --spec cypress/e2e/base-ui-components.cy.ts --reporter spec` ran against the Vite app. The reported failures were `renders all documented variants and states` and `exposes keyboard focus and accessible field relationships`; the run also produced a failed screenshot for the FilterChip/TextArea test, but its test title was not present in the captured console output.
- **Failure reason:** The first reported failure could not find the expected component demo selectors under `/components` (the existing route still rendered Foundations), and the second therefore could not find the planned field controls. This was a behavioral failure caused by the missing T51 functionality; Cypress itself and the assertions compiled and executed.
- **Supplemental red loop:** After adding review assertions, `npx cypress run --spec cypress/e2e/base-ui-components.cy.ts --reporter spec` produced 1 passing and 2 failing tests before the fixes. `exposes keyboard focus and accessible field relationships` failed because the combined helper/error field's `aria-describedby` contained only one message ID; `supports the configured text area height and filter state` failed because the active FilterChip had no `aria-pressed` attribute. These were behavioral failures demonstrating the missing planned accessibility relationships and state exposure, not test setup failures.

### Green (after implementation)

- **Result:** Focused `npx cypress run --spec cypress/e2e/base-ui-components.cy.ts --reporter spec` passed all 3 tests after the review fixes. `npm run typecheck`, `npm run lint`, and `npm run build` passed.
- **Full suite of the affected repo:** `npx cypress run --reporter spec` ran the wider suite; T51's 3 tests and the existing Foundations/home coverage passed. One existing API-client assertion failed because the configured API URL was `https://ubb-events-backend.onrender.com` while that test expected `http://backend.test`; this is an existing environment/configuration mismatch outside T51's scope.

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

- **Decisions:** Use typed `forwardRef` React controls in `src/shared/ui`, native form elements for keyboard and browser semantics, semantic T49 tokens, and a `/components` demo route as the Cypress integration surface. Select and TextArea reuse TextField styling because the Components section has no separate Figma nodes.
- **Alternatives considered:** A component-test runner or a new icon dependency was not introduced; the existing Cypress setup and local SVG assets keep the task within the current dependency set.
- **Limitations:** The full Cypress suite has one pre-existing API-client environment mismatch: the configured Render URL differs from the test's `backend.test` expectation. Figma's temporary SVG URLs were not retained; the inspected dot, search, and chevron assets are served from `public/ui-assets`.

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


