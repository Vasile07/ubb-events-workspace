# T53 - Feedback components: toast, empty state, skeleton, error state

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T53
- **Type:** artifact
- **Owner:** Brudea Armand
- **Requirement(s):** Source task 6 — Feedback components: toast, empty state, skeleton, error state.
- **Use case(s):** N/A — the source task defines a reusable UI artifact and does not identify a standalone user scenario.
- **Workflow:** N/A — shared frontend feedback components; no API/service workflow is specified.
- **Drive doc / ADR:** [Project requirements, use cases, architecture, and API documentation](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban) (general project document; no T53-specific section reference is recorded).
- **Repo(s):** frontend
- **Branch:** `T53-feedback-components-toast-empty-state`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/7 (created, reviewed, and merged by user)
- **Tests:** `cypress/e2e/feedback-components.cy.ts`; `npm run typecheck`; `npm run lint`; `npm run build`; `npm run cypress:run`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 6 — Feedback components: toast, empty state, skeleton, error state

**Description:** Implement the components that cover non-happy-path states.
- `Toast` (Success and Error, icon circle, title, message, close; Design System node 46:30) with a `ToastProvider` and `useToast()` that stacks toasts top-right, auto-dismisses after ~5 s and is announced via `aria-live`.
- `EmptyState` (icon variants Search, Calendar, Ticket, Users; title, description, optional primary action; node 46:68).
- `Skeleton` primitives (rounded `bg/muted` blocks) plus ready-made `EventCardSkeleton` and `TableSkeleton` (see frames `Discover events – loading` 48:587 and `Org – Events (loading)` 49:1175).
- `ErrorState` (danger icon, title, description, "Try again" button, error code caption; see frame `Discover events – error` 48:757) and a React `ErrorBoundary` that renders it.

Done when each component has a demo. Every screen task below must implement its own loading, empty and error states with these components.
**Depends on:** T51 — Base UI components: actions and form fields; T52 — Base UI components: layout and data display.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/shared/ui/LayoutComponents.tsx` provides `Icon`, `Button` is exported through `src/shared/ui/index.ts`, and `src/App.tsx` hosts the `/components` showcase; `src/styles/global.css` contains semantic status, surface, muted text and border tokens. `package.json` currently defines `cypress:run`, `typecheck`, `lint`, and `build` scripts.
- **Related completed tasks:** `changes/completed/T51-base-ui-components-actions-and.md` and `changes/completed/T52-base-ui-components-layout-and.md`; both prerequisites are recorded as merged into frontend `master`.
- **Constraints / assumptions:** Frontend implementation is on `T53-feedback-components-toast-empty-state`, created from verified up-to-date `master` at `6820013`. T51 and T52 are merged prerequisites. Figma session `d72eb6f8-e488-49b6-ad3d-ca7955202bdb` verified the named design contexts/screenshots: Toast `46:30`, EmptyState `46:68`, EventCardSkeleton `48:639`, TableSkeleton `49:1301`, and ErrorState `48:808`. The user selected `artifact` for this React UI deliverable. `context/project-context.md` says frontend test commands will be established in a future task, although current `package.json` defines `cypress:run`, `typecheck`, `lint`, and `build` scripts.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the feedback primitives as typed, presentational React components in `src/shared/ui`, reusing the existing `Icon`, `Button`, T49 semantic tokens, and T52 layout/data-display styles. Keep transient toast state in a context provider with a small public `useToast` API; keep `EmptyState`, skeletons, and `ErrorState` stateless except for their callback props. Add a review-only `/components` showcase section that demonstrates every variant and exercises the `ErrorBoundary`. Use Cypress against that showcase for behavior and accessibility because the frontend currently exposes Cypress as its browser-level test runner. This is an artifact/UI task; no backend or deployment configuration is required.

### Steps

1. **Dependency and design verification (UI):** confirm T51 and T52 are present on frontend `master`; inspect the named Design System nodes/frames for toast, empty, loading and error states and record any material sizing or copy decisions in the task notes.
2. **Tests first (UI):** extend the `/components` Cypress coverage before implementation. Assert all Toast, EmptyState, Skeleton, EventCardSkeleton, TableSkeleton and ErrorState variants; toast stacking, close and approximately five-second dismissal; `aria-live`; empty-state action; skeleton structure; ErrorBoundary fallback, retry callback and code caption; keyboard focus and accessible names. Capture a valid behavioral red result.
3. **Feedback components (UI):** add typed components under `src/shared/ui` for `ToastProvider`/`useToast`, `Toast`, `EmptyState`, `Skeleton`, `EventCardSkeleton`, `TableSkeleton`, `ErrorState`, and `ErrorBoundary`. Use native buttons, stable IDs where relationships are needed, and the existing `Icon`/`Button` APIs. Ensure timers are cleaned up on unmount and provider consumers fail clearly when used outside a provider.
4. **Styling and accessibility (UI):** add token-backed styles for top-right stacked toasts, success/error icon circles, muted rounded skeleton blocks, empty-state icon variants, danger error state and responsive behavior. Apply `aria-live` to the toast region, meaningful labels for icons, visible focus treatment, and correct button semantics without introducing a new dependency.
5. **Showcase integration (UI):** add the feedback examples to the existing `/components` route with controls that open success/error toasts, render each empty icon variant, show both ready-made skeletons, and trigger/recover from an error boundary. Keep existing T51/T52 demos intact.
6. **Verification (UI):** run the focused Cypress spec, then the currently available `npm run typecheck`, `npm run lint`, `npm run build`, and wider Cypress suite. Record exact commands and distinguish any pre-existing environment failures from T53 behavior.

### Files to create / modify

- `repos/ubb-events-frontend/src/shared/ui/FeedbackComponents.tsx` (or focused files for provider, states and boundary)
- `repos/ubb-events-frontend/src/shared/ui/index.ts`
- `repos/ubb-events-frontend/src/App.tsx` or a dedicated showcase component for the `/components` route
- `repos/ubb-events-frontend/src/styles/global.css` and/or focused feedback styles
- `repos/ubb-events-frontend/cypress/e2e/feedback-components.cy.ts`

### Tests to write

- **Acceptance criterion 1:** showcase Cypress coverage verifies every described component, prop variant and interaction, including toast auto-dismiss/stacking/close, all four EmptyState icons, both ready-made skeletons, ErrorState retry/code, and ErrorBoundary fallback.
- **Acceptance criterion 2:** a repository check records T51 and T52 as merged prerequisites before implementation; implementation must stop if either dependency is absent from the base branch.
- **Acceptance criterion 3:** accessibility/state tests verify `aria-live`, keyboard-accessible close/retry/primary-action buttons, visible focus, icon labels or decorative hiding, stable error relationships, skeleton loading semantics, and error-boundary recovery. Include timer cleanup, multiple toasts, empty descriptions, and child errors as edge cases.

### Risks / open questions

- The task names Figma nodes but does not include node IDs; before implementation, use the named page/frame metadata and the existing Figma workflow to identify the exact nodes. If a named frame is absent or ambiguous, ask the user rather than guessing.
- `context/project-context.md` says frontend test commands will be established in a future task; current `package.json` nevertheless exposes Cypress/typecheck/lint/build scripts. The plan uses those existing scripts for verification and will document that they are repository-local commands, not a project-wide convention.
- React error boundaries cannot catch errors thrown in event handlers; the demo should trigger a render-time child error and expose a retry path. The boundary API should document this limitation in the task notes.
- Toast timing tests can be flaky with real time; use Cypress clock control for deterministic dismissal while retaining the production five-second default.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/feedback-components.cy.ts` — four Cypress tests covering component rendering, toast lifecycle, empty/error actions, and ErrorBoundary recovery.
- **Result:** `npm run cypress:run -- --spec cypress/e2e/feedback-components.cy.ts --reporter spec` — 0 passing, 4 failing. `renders the feedback component variants and skeleton semantics`, `announces, stacks, closes, and auto-dismisses toasts`, `supports empty-state actions and error-state retry with keyboard access`, and `catches render errors and recovers through the error boundary retry action` failed. The sandbox could not find its Cypress executable; the same command was rerun with approved escalation, Cypress started successfully, and all four assertions executed.
- **Failure reason:** The tests could not find the planned T53 showcase and controls (`feedback-demo`, `toast-success-trigger`, `empty-state-action`, and `error-boundary-trigger`) on the existing `/components` route. Cypress and the assertions ran successfully; the missing behavior is the planned T53 functionality.

### Green (after implementation)

- **Result:** Final post-review run `npm run cypress:run -- --spec cypress/e2e/feedback-components.cy.ts --reporter spec` passed 4/4. The Cypress checks assert that each EmptyState uses its local Figma SVG and that an error Toast uses the circled-exclamation SVG. `npm run typecheck`, `npm run lint`, and `npm run build` passed. The first post-implementation Cypress run was 3/4 because the toast count selector also matched trigger and close buttons; it was narrowed to `.ui-toast` inside the live region, after which focused runs passed.
- **Full suite of the affected repo:** With the Vite server started after setting `VITE_API_URL=http://backend.test` (PowerShell: `$env:VITE_API_URL='http://backend.test'; npm run dev`), the final post-review `npm run cypress:run -- --reporter spec` passed all 6 specs, 27/27 tests. This config matches the existing API-client Cypress fixture. The first full-suite attempt had no `VITE_API_URL` in the Vite process and stopped during `api-client.cy.ts`; it was rerun with the test fixture URL and passed. Browser review identified white EmptyState glyphs and a triangular error Toast glyph; these were replaced with local exact SVG assets from the verified Figma contexts, then reviewer visual inspection confirmed the corrected icon treatments in the `/components` preview at the available viewport width.
- **Commit and push:** Commit `0fa5d84` (`T53: add feedback UI components`) was created on `T53-feedback-components-toast-empty-state` and pushed to `origin/T53-feedback-components-toast-empty-state`.
- **PR:** [#7](https://github.com/Vasile07/ubb-events-frontend/pull/7) was created, reviewed, and merged by the user.

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

- **Decisions:** Implemented typed presentational components in `src/shared/ui/FeedbackComponents.tsx`, exported through the shared UI index, and added them to the `/components` showcase. `ToastProvider` owns toast state and timers, `useToast` exposes `showToast`, `dismissToast`, and success/error helpers, and each toast dismisses after 5 seconds unless closed sooner. Toast, empty, error, and skeleton styles reuse the existing design tokens and `Button`; EmptyState and error Toast icons use local SVG assets exported from the Figma design context, while other matching icons reuse the existing `Icon`. `ErrorBoundary` renders the shared `ErrorState` and retries by clearing its failed state.
- **Alternatives considered:** Kept the related primitives in one `FeedbackComponents.tsx` module instead of splitting each into a separate file; reused the existing Cypress runner and shared icon/button components rather than adding dependencies.
- **Limitations:** React error boundaries catch render and lifecycle errors, not errors from event handlers. Visual review of the `/components` preview confirmed the corrected empty-state icons and error Toast match the Figma icon treatments; the preview was inspected in the available in-app viewport rather than at the full reference-frame width. Full Cypress verification needs `VITE_API_URL=http://backend.test` supplied to the Vite process so the existing API-client test fixture can run.

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
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`


