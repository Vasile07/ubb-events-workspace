# T70 - Event form component

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T70
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** N/A
- **Use case(s):** UC-06, UC-07
- **Workflow:** Organization event creation and editing
- **Drive doc / ADR:** N/A
- **Repo(s):** frontend
- **Branch:** `T70-event-form-component`
- **PR link(s):** [Frontend PR #23](https://github.com/Vasile07/ubb-events-frontend/pull/23)
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-form.cy.ts`; frontend `typecheck`, `lint`, and `build`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 23 — Event form component

**Description:** Build the reusable `EventForm` from page `Organization Screens`, frames `Org – Create event` (15:202) and `Org – Edit event` (49:740): sections "Event details" (title, description textarea, location), "Schedule & capacity" (start date, start time, end date, end time, capacity, registration information) and, in create mode, "Tasks" (rows of description + assignee select + remove button + "Add another task").
Right column in create mode: Event lifecycle card and "Before you publish" checklist; in edit mode: the information card and current status card.
Validation: required title, description, location, start and end; end after start (Error state on End time with "End must be after the start"); capacity ≥ 1; in edit mode capacity cannot be below the active registrations (show the backend message as shown in the edit frame).
The assignee select lists members from `GET /api/organizations/{organizationId}/members`.
Backend errors: the form accepts `fieldErrors` with nested paths (e.g. `tasks[1].description`, `tasks[2].assigneeId`) and shows each message on the exact task row, plus a short error summary at the top of the form listing everything that must be corrected. Form values must never be lost after a failed submit.
Props: `mode`, `initialValues`, `onSubmit`, `submitting`, `fieldErrors`.
**Depends on:** T51 — Base UI components: actions and form fields; T52 — Base UI components: layout and data display; T68 — Organization workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/shared/ui/TextField.tsx` provides labeled input and textarea controls with helper/error messaging; `Select.tsx`, `Button.tsx`, and `LayoutComponents.tsx` provide the select, actions, icons, banners, and layout primitives. `src/shared/api/client.ts`, `src/shared/api/organizations.ts`, and `src/shared/api/types.ts` provide API/error patterns and organization member-related types; `src/features/events/api.ts` and `types.ts` provide event API/type patterns. `src/styles/global.css` contains the design tokens and shared styles. `src/App.tsx` and `src/features/organizations/` contain the T68 organization workspace structure.
- **Related completed tasks:** T51 (base UI actions and form fields), T52 (layout and data display), and T68 (organization workspace shell), all reported merged by the user and represented in the completed task files.
- **Constraints / assumptions:** This task is frontend-only and depends on T51, T52, and T68 being merged before implementation. The user confirmed both repositories are up to date with current `master`; the frontend worktree was clean and switched to `master`. Follow existing shared controls, API client, React Query, Cypress, and CSS token conventions. Use the named Organization Screens Figma frames in the task description as the visual targets.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement a controlled, reusable `EventForm` feature component that owns draft form state and client-side validation while exposing the task's public API (`mode`, `initialValues`, `onSubmit`, `submitting`, and `fieldErrors`). Keep API access outside the presentational form: add a typed organization-members request helper/query for assignee options, and let create/edit screens supply organization id, initial values, submit handling, and backend errors. Render the two named Figma variants (`Org – Create event`, `15:202`, and `Org – Edit event`, `49:740`) with shared sections and mode-specific side cards. Reuse T51/T52 controls and preserve draft values across failed submissions while mapping nested backend paths to row-level messages and a summary.

### Steps

1. **UI/API contract:** Define event-form value, task-row, mode, and field-error types. Add typed `GET /api/organizations/{organizationId}/members` access using existing API client and React Query conventions, including loading, empty, and error states for assignee options.
2. **Tests first:** Add Cypress coverage for a stable event-form review route or existing organization event route before implementation. Cover create/edit rendering, initial values, task add/remove, member options, required-field validation, chronological validation, capacity rules, nested backend errors, error summary, submit/loading behavior, and preservation of entered values after failed submit. Include semantic labels, descriptions, keyboard access, and disabled states.
3. **Form implementation:** Build `EventForm` with controlled draft state initialized from `initialValues`; render event details, schedule/capacity, registration information, and create-only task rows. Use stable field names/IDs and exact paths (`tasks[index].description`, `tasks[index].assigneeId`) so backend errors attach to each row. Keep add/remove changes local and preserve remaining values.
4. **Validation and submit behavior:** Validate required title, description, location, start and end, `end > start` with the exact End time message, capacity `>= 1`, and edit-mode capacity against active registrations. Show inline errors and a concise top summary listing corrections. Prevent submission while invalid or `submitting`; pass the complete draft to `onSubmit`; retain the draft when `fieldErrors` or submission failure is returned.
5. **Mode-specific layout and states:** Match the create frame with lifecycle/checklist cards and tasks, and the edit frame with information/current-status cards. Add accessible submitting state and responsive/token-backed styles. Keep business rules in surrounding workflow/API layers; the form validates and presents supplied values/errors.
6. **Verification:** Run the focused Cypress spec, then `npm run typecheck`, `npm run lint`, `npm run build`, and the full Cypress suite. Record behavioral red evidence before implementation and green results afterward, including unrelated pre-existing failures.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/EventForm.tsx` — reusable form and mode-specific layout.
- `repos/ubb-events-frontend/src/features/events/eventFormTypes.ts` and/or `eventFormValidation.ts` — public types and pure validation helpers.
- `repos/ubb-events-frontend/src/shared/api/organizations.ts` and `src/shared/api/types.ts` — organization members endpoint and response type.
- `repos/ubb-events-frontend/src/features/organizations/` or event create/edit route modules — wire member query, initial values, submit and backend error props without moving business rules into the form.
- `repos/ubb-events-frontend/src/App.tsx` only if a review route is needed for the Cypress surface.
- `repos/ubb-events-frontend/src/styles/global.css` — form grid, task rows, side cards, validation and responsive styles.
- `repos/ubb-events-frontend/cypress/e2e/event-form.cy.ts` — focused behavioral and accessibility coverage.

### Tests to write

- **Acceptance criterion 1 / description behavior:** Cypress verifies both modes, every section and mode-specific card, task add/remove, member endpoint/options, exact submit payload, and the public props contract.
- **Acceptance criterion 1 / rules and errors:** tests cover required fields, end-before-start exact message, capacity lower bound, edit active-registration capacity message, nested task field errors, summary contents, and preservation after failed submit.
- **Acceptance criterion 2 / dependencies:** the focused run exercises the merged T51/T52 controls and T68 organization shell; record the confirmed dependency state in the test log.
- **Acceptance criterion 3 / states and accessibility:** tests cover member loading, empty and failed lookup, submitting/disabled controls, `aria-invalid` and described-by relationships, keyboard operation, and visible error summary.
- **TDD red/green evidence:** write the spec before `EventForm`, run it to show behavioral failures caused by missing form behavior, then implement and rerun focused and full frontend checks.

### Risks / open questions

- The task specifies the members endpoint but not its exact response shape. Reuse the existing frontend/backend contract if present; otherwise confirm fields before implementation rather than inventing them.
- Create/edit screens may not yet exist on `master`; if so, use a minimal review harness for the reusable component and keep route integration within T68. Do not implement T71/T73 workflow behavior in T70.
- The edit capacity error must display the backend-provided message. Accept it through `fieldErrors` and do not hard-code a registration count.
- Figma details are visual targets; existing shared components and design tokens remain the source of truth for behavior and accessibility.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/event-form.cy.ts` (2 tests).
- **Result:** `$env:CYPRESS_CACHE_FOLDER='C:\\Users\\Teofan\\AppData\\Local\\Cypress\\Cache'; npx cypress run --spec cypress/e2e/event-form.cy.ts --reporter spec` — 0 passing, 2 failing after configuring `VITE_API_URL=http://backend.test` for Vite.
- **Failure reason:** Both tests executed against the app and timed out because `[data-testid="event-form"]` and `[data-testid="add-task"]` did not exist. This was a behavioral failure caused by the missing planned EventForm functionality. The initial run without the Cypress cache override was blocked by the sandbox-local Cypress binary path and was not used as red evidence.

### Green (after implementation)

- **Result:** Focused Cypress run with the expanded spec passed 7/7. `npm run typecheck`, `npm run lint`, and `npm run build` all passed.
- **Full suite of the affected repo:** With Vite running at `http://127.0.0.1:5173`, `npx cypress run --reporter spec` completed 23 specs: 132 tests, 130 passing, 2 failing. Both failures are in the pre-existing `organizations-list.cy.ts` visibility assertions; the T70 spec passed 7/7.

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

- **Decisions:** The form owns controlled draft state and pure client validation while accepting backend `fieldErrors`; supplied backend field errors take precedence over local messages. Organization member lookup is isolated in the API helper and React Query call, with explicit loading, empty, and failure states. A small `/event-form-demo` route provides a stable Cypress review surface without implementing T71/T73 workflow behavior.
- **Alternatives considered:** Reusing the existing organization event routes was avoided because create/edit screens are placeholders; a dedicated review route keeps T70 focused on the reusable component. The side cards use semantic lists and status text rather than introducing new business state.
- **Limitations:** The member contract is represented by the existing frontend-compatible fields (`id`, `fullName` or first/last name). The wider suite retains two pre-existing `organizations-list.cy.ts` visibility failures; focused T70 coverage is complete and green.

  **TDD deviation:** Nested backend-error, member-state, and edit-card assertions were added during post-implementation review to close coverage gaps. They were run as supplemental checks and passed; the original pre-implementation red run remains the valid red evidence for the core form behavior.

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


