# T49 - Design tokens and global styles

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T49
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T49-design-tokens-and-global-styles`
- **PR link(s):** 
- **Tests:** `cypress/e2e/foundations.cy.ts`, `cypress/e2e/home.cy.ts`; npm `typecheck`, `lint`, `build`, and Prettier checks

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 2 — Design tokens and global styles

**Description:** Implement the design tokens so no component hardcodes colors, sizes or fonts.
Figma: page `Design System`, frame `Foundations` (4:40) and the variable collections `Primitives`, `Semantic`, `Spacing & Radius` (read them through `get_variable_defs`).
Include:
- the blue palette (#223A5E, #355982, #4D79A8, #6E9ECC, #9FC0E3, #D0E1F2, #F3F8FD)
- semantic tokens (bg/page, bg/surface, bg/brand, text/primary|secondary|muted|inverse, border/default|strong|focus, accent, status success/warning/danger fg+bg)
- spacing (4–48)
- radii (8, 12, 16, 24, pill)
- shadows `Shadow/Card` and `Shadow/Raised`
- font Plus Jakarta Sans (400–800) with the 10 text styles (Display/Hero, Heading H1–H3, Body Large/Default/Strong, Caption Default/Strong, Label/Button).

Add a global reset and set the page background to `bg/page`.
Done when a demo page side by side with the Foundations frame is visually identical.

## Acceptance criteria


- The Figma primitive palette and semantic colors are defined and consumed through tokens: blue 900/800/700/600/400/200/50, bg/page, bg/surface, bg/brand, text/primary/secondary/muted/inverse, border/default/strong/focus, accent, and status success/warning/danger foreground/background values.
- The spacing scale (4–48), radii (8, 12, 16, 24, pill), `Shadow/Card`, and `Shadow/Raised` are defined as reusable tokens, and the 10 Plus Jakarta Sans text styles are implemented: Display/Hero, Heading H1–H3, Body Large/Default/Strong, Caption Default/Strong, and Label/Button.
- A global reset is applied, the page background resolves to `bg/page`, and a Foundations demo route renders the token, typography, status, spacing, radius, and elevation examples with accessible labels and a desktop presentation visually matching Figma Design System frame `4:40`; focused automated checks and the frontend verification scripts pass.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` is a placeholder home route with no feature-specific styling; `src/main.tsx` mounts it under React Router and TanStack Query; the repository currently has no stylesheet under `src/`. `index.html` and `package.json` define the Vite app shell and `dev`, `build`, `lint`, and `typecheck` scripts.
- **Related completed tasks:** `changes/completed/T48-project-setup.md` established the React + TypeScript + Vite frontend, npm scripts, Cypress configuration, and `.gitignore` rules for `node_modules/`, `dist/`, and `*.tsbuildinfo`.
- **Constraints / assumptions:** `context/project-context.md` specifies React and the frontend layering conventions; use the existing frontend repo and its established npm commands. The frontend is clean on `master` and synchronized with `origin/master`. Figma session `581a16e3-822f-4743-accb-d043d46ca6eb` inspected Design System page `0:1` and Foundations frame `4:40` (1440 × 1768). The frame/context exposes the palette, status colors, typography, and `Shadow/Card`/`Shadow/Raised`; design variables include Plus Jakarta Sans styles and semantic color tokens. The implementation covers the 10 text styles present in the Figma typography frame: Display/Hero, Heading H1–H3, Body/Large, Body/Default, Body/Strong, Caption/Default, Caption/Strong, and Label/Button. Figma names spacing 4–48 and radii 8, 12, 16, 24, pill in the task description.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Create one global stylesheet for the frontend foundation layer and import it from the app entry point. Define the Figma primitives and semantic aliases as CSS custom properties, then expose the documented spacing, radius, elevation, and Plus Jakarta Sans type styles through reusable classes. Keep the placeholder route as a small Foundations demo so the implementation can be compared directly with Figma frame `4:40`; do not add feature business logic or API behavior. The CSS-token approach fits the existing Vite shell and lets later components consume semantic names without hardcoded visual values. A component-library abstraction or a utility framework would add dependencies and obscure the mapping to the named Figma variables, so it is out of scope.

### Steps

1. **UI / test setup:** Add Cypress checks for the Foundations route and computed styles before implementing the stylesheet, including a deliberate failure for the missing `bg/page` background and token-backed demo content. Record the behavioral red result according to the repository TDD workflow.
2. **UI / tokens:** Create the global stylesheet with the seven blue primitives, status colors, semantic background/text/border/accent aliases, spacing scale from 4 through 48, radii (8, 12, 16, 24, pill), and the two Figma shadow definitions. Keep token names stable and use semantic aliases in rendered styles.
3. **UI / typography:** Add the Plus Jakarta Sans font loading strategy and the 10 named text styles with the Figma size, weight, line-height, and letter-spacing values: Display/Hero; Heading H1–H3; Body Large/Default/Strong; Caption Default/Strong; Label/Button.
4. **UI / global reset:** Import the stylesheet from `src/main.tsx`, apply a predictable box-sizing/margin reset, set the document/body/root sizing, and set the page background to `bg/page`.
5. **UI / demo route:** Update the placeholder home route into a Foundations demo containing representative palette swatches, status colors, spacing/radius/elevation samples, and typography samples arranged to match the Foundations frame at desktop width. Ensure labels and contrast remain accessible.
6. **Verification:** Run the focused Cypress checks, then `npm run typecheck`, `npm run lint`, `npm run build`, and the wider Cypress suite. Compare the demo side by side with Figma frame `4:40`; adjust only in-scope styling mismatches and record the final evidence.

### Files to create / modify

- `repos/ubb-events-frontend/src/styles/global.css`: primitives, semantic aliases, typography classes, shadows, reset, and document-level styling.
- `repos/ubb-events-frontend/src/main.tsx`: import the global stylesheet before rendering the app.
- `repos/ubb-events-frontend/src/App.tsx`: Foundations demo route and accessible sample markup.
- `repos/ubb-events-frontend/cypress/e2e/foundations.cy.ts`: token, reset, page-background, typography, and representative demo assertions.
- `repos/ubb-events-frontend/index.html` (only if needed for the chosen Plus Jakarta Sans loading method): font preload/import wiring without secrets or external application behavior.

### Tests to write

- **Description / primitive and semantic tokens:** Cypress assertions that the Foundations demo exposes the seven blue values, required status values, semantic aliases, spacing values, radii, and both shadows through computed styles or CSS custom-property values.
- **Description / typography:** Cypress assertions for each of the 10 style classes, including family, size, weight, line-height, and letter-spacing; include a fallback-font case if the web font is unavailable.
- **Description / reset and page background:** Cypress assertions that body margins are reset, box sizing is inherited, the root fills the viewport, and the page background resolves to `#F3F8FD` (`bg/page`).
- **Description / visual demo:** a Cypress smoke/accessibility check that the demo route renders all labeled sections and remains keyboard-readable; visual comparison against Figma frame `4:40` is a manual review gate recorded with the task.
- **Acceptance criterion about prerequisites:** verify the demo is built on the T48 frontend baseline and the existing npm scripts remain usable; no backend or API tests apply to this UI-only task.

### Risks / open questions

- No open questions remain after the user confirmed the specific acceptance criteria and the 10-style typography scope.
- Plus Jakarta Sans may require a network font import during development. Prefer a committed local asset if the repository provides one; otherwise use the documented font stack and record network availability as a visual verification limitation.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/foundations.cy.ts` (`renders the Foundations demo sections`, `applies the page background and reset`, `exposes the palette and typography tokens`).
- **Result:** `npx cypress run --spec cypress/e2e/foundations.cy.ts --browser electron --config baseUrl=http://127.0.0.1:5174` ran with Vite on port 5174: 0 passing, 3 failing. The demo heading and `[data-token=blue-900]` element were missing, and the body margin was `8px` instead of `0px`.
- **Failure reason:** The failures demonstrate the planned Foundations demo, global reset, page background, and token-backed content are not implemented yet. Cypress executed the tests in Electron against the running app; this was a behavioral red result.
- **Correction-cycle red (post-implementation):** Supplemental visual assertions were added for status computed colors, exact Figma typography samples, the status note, shadow subtitle, spacing specimens, and desktop structure. The pre-fix command `npx cypress run --spec cypress/e2e/foundations.cy.ts --browser electron --config baseUrl=http://127.0.0.1:5174` passed 5 tests and failed the new visual test because `.status-color` computed to `rgba(0, 0, 0, 0)` instead of `rgb(43, 122, 87)`. This was separate from the original pre-implementation red evidence.
- **Geometry-correction red (post-implementation):** Added assertions for zero heading/description margins and the Figma geometry at 1440×1768. Before the fix, `npx cypress run --spec cypress/e2e/foundations.cy.ts --browser electron --config baseUrl=http://127.0.0.1:5174` failed on the margin assertion (`21.44px 0px` instead of `0px`), demonstrating the existing layout still diverged from frame `4:40`.
- **Display/Hero red (post-implementation):** Added a desktop regression asserting the exact 44px sample remains one line. Before the fix, the browser reported `white-space: normal`, so the assertion failed before any UI change; the computed family remained the configured Plus Jakarta Sans stack and the test measured the rendered sample geometry.

### Green (after implementation)

- **Result:** The geometry correction removed default text margins, matched the 886×592 type card and target section bounds, set target typography row gaps/heights, and moved shadow subtitles inside each card. The Display/Hero correction adds `white-space: nowrap` without changing its specified 44px size. The final Cypress run passed 9/9 tests across `foundations.cy.ts` and `home.cy.ts`. `npm run typecheck`, `npm run lint`, `npm run build`, and the Prettier check also passed.
- **Full suite of the affected repo:** The frontend Cypress suite passed 9/9; typecheck, lint, production build, and formatting checks passed. A side-by-side visual comparison was performed at `http://localhost:4173/` with a 1440×1768 viewport against Figma Design System → Foundations frame `4:40`; the section, type-card, and one-line Display/Hero geometry now match the inspected target.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The task type was `deployment`. | Changed the type to `workflow`. | The user requested the change. |
| 2 | plan | “yeah, the generic acceptance cryteria should be replaced with specific ones” | Replaced the three generic acceptance criteria with criteria naming the token groups and values, 10 text styles, global reset/page background, and Foundations visual verification. | The user requested specific, traceable criteria. |
| 3 | plan | “yeah, Body/Medium was removed so no more Body/Medium” | Removed `Body/Medium` from the context, plan, tests, and open questions; the implementation scope now covers the 10 styles present in Figma. | The user confirmed the design scope. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Draft for user review: CSS custom properties mirror the Figma primitive and semantic token names, with reusable typography classes and a global reset. The existing placeholder route now renders the Foundations demo so the token system can be checked visually and through Cypress. Plus Jakarta Sans is loaded through a Google Fonts import with a system fallback.
- **Alternatives considered:** Draft for user review: a utility framework or component-library abstraction was excluded because it would add dependencies and obscure the direct mapping from Figma variables to CSS tokens. Feature behavior and API integration remain outside this design-foundation task.
- **Limitations:** Draft for user review: the font import depends on network availability; the fallback stack preserves layout and readable typography when unavailable. Cypress was run with the Electron browser, which reports a deprecation warning but passed all tests. The Figma frame does not contain a spacing row, so the spacing specimen follows the frame-matched sections.

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


