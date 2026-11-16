# T73 - Event actions: edit, publish and delete

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T73
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** Source task 26 — Event actions: edit, publish and delete
- **Use case(s):** UC-07, UC-08, UC-17
- **Workflow:** N/A (no separate workflow reference was supplied)
- **Drive doc / ADR:** N/A (no task-specific link was supplied)
- **Repo(s):** frontend
- **Branch:** `T73-event-actions-edit-publish-and`
- **PR link(s):** [Frontend PR #25](https://github.com/Vasile07/ubb-events-frontend/pull/25)
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-actions.cy.ts`; frontend typecheck, lint, build, and Cypress regression suite

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 26 — Event actions: edit, publish and delete

**Description:** Add the manager actions to the event workspace header.
- Edit event → route `/org/:organizationId/events/:eventId/edit` reusing `EventForm` in edit mode (frame `Org – Edit event` 49:740) with `PUT .../events/{eventId}`; disable it for completed/canceled events.
- Publish (DRAFT only): primary button "Publish event" next to Edit and Delete as in frame `Org – Event: Tasks (draft)` (49:897) with the draft warning banner; clicking opens the confirmation modal from frame `Org – Publish event confirmation` (49:1055) and calls `PATCH .../events/{eventId}/publish`; show backend validation messages (missing end date etc.) in the modal.
- Delete event: Danger button opening the modal from frame `Org – Delete event confirmation` (16:1061) that shows the number of affected participants and calls `DELETE .../events/{eventId}` (soft delete; the backend refuses ended events, show that message).

Invalidate event, events list and discover queries and show toasts. Hide all of these for plain members.
**Depends on:** T70 — Event form component; T72 — Event workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/features/events/EventWorkspace.tsx` provides the manager-action slot and event status/data; `src/features/events/api.ts` has the organization-scoped detail call; `src/features/events/types.ts` defines `OrganizationEvent`; `src/App.tsx` defines the manager-only edit route and event workspace routes; `src/shared/api/client.ts` and `src/shared/api/types.ts` define API error handling; shared UI includes buttons, modals, and toasts. On `T70-event-form-component`, `src/features/events/EventForm.tsx` supports create/edit modes, but edit-mode initialization, persistence, and mapping need to be checked against its types and the API contract during planning.
- **Related completed tasks:** T70 — Event form component and T71 — Create event flow are merged to master; T72 — Event workspace shell (`changes/completed/T72-event-workspace-shell.md`) is also merged to master.
- **Constraints / assumptions:** Frontend-only workflow. The T73 branch was created from `origin/T70-event-form-component` and fast-forwarded to current `master` after T70 and T71 merged, preserving the requested ancestry. The backend `UpdateEventRequest` has one `date` plus start/end times; it has no separate end date or registration-information field, so the edit form disables its end-date input and sends only the confirmed DTO fields. The workspace root has pre-existing T70/T71 task-file moves; they were left untouched. Frontend run/test commands remain unspecified in `context/project-context.md`; repository scripts were used.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Extend the T72 `EventWorkspace` header with a manager-only action group and keep the existing member workspace unchanged. Reuse T70's `EventForm` for the edit route, adding only the data mapping and save orchestration needed by this task. Put lifecycle requests in the events API module and coordinate them with React Query mutations so event detail, organization event lists, and discover results are invalidated after success. Use the existing shared `Modal`, `Button`, and toast primitives for confirmation, accessible focus handling, validation/error feedback, and completion messages.

The UI will treat backend responses as authoritative for lifecycle rules: only drafts expose Publish, completed/canceled events cannot be edited, and delete/publish failures are shown in their originating modal. Plain members continue to receive the T72 shell without the manager action controls. The task branch will be created from `T70-event-form-component` after Gate 1, and implementation starts only once T70 is merged; if the merge changes the branch base, update the T73 branch while preserving the T70 implementation.

### Steps

1. **Dependency and contract check:** after Gate 1, verify T70 is merged and create `T73-event-actions-edit-publish-and` from `T70-event-form-component` as requested. Confirm the actual edit response/request DTO, publish validation error shape, delete response participant count, and query keys used by T72/T69 before writing implementation code; record any contract mismatch as an open question rather than inventing fields.
2. **Tests first (UI/API):** add Cypress tests and minimal API/type stubs for manager and member roles. Capture behavioral red results for edit navigation and populated edit form, PUT save/error handling, draft-only publish confirmation and validation feedback, delete confirmation with affected participant count and ended-event rejection, successful redirects/toasts, query invalidation, and hidden controls for members and completed/canceled states.
3. **API layer:** add typed `PUT`, publish `PATCH`, and soft-delete `DELETE` calls under `src/features/events/api.ts`, plus request/response/error types required by the confirmed backend contract. Preserve `ApiError` messages and structured validation details so modal feedback can render missing-field and business-rule messages safely.
4. **Edit workflow:** replace the manager edit placeholder with a page that loads the organization event, maps its values into T70's `EventForm` edit shape, submits the specified `PUT /api/organizations/{organizationId}/events/{eventId}`, renders field and form-level backend errors, disables editing for completed/canceled events, and returns to the workspace with a success toast after save.
5. **Workspace actions:** fill T72's actions slot with manager-only Edit, Publish, and Delete controls. Render Publish only for `DRAFT`; open the specified confirmation modal and show the draft warning plus backend validation messages. Open the delete modal with affected participant count, execute soft delete, show ended-event refusal in the modal, and navigate back to the organization events page after success. Keep controls keyboard accessible and focus-safe through the shared modal.
6. **Cache and feedback integration:** invalidate the event detail, organization events list, and discover query keys after each successful mutation (and refetch detail when a conflict or validation response requires current state). Show success/error toasts, preserve loading/disabled states, and ensure member routes never render manager controls.
7. **Verification:** run the focused Cypress spec first for the red/green evidence, then the available frontend typecheck, lint, build, and wider Cypress suite. Record exact commands, outcomes, and any unrelated baseline failures in the task test log.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/api.ts` — edit, publish, and soft-delete API calls and typed payloads/results.
- `repos/ubb-events-frontend/src/features/events/types.ts` and T70 event-form types/validation files — confirmed edit request/response and backend error mappings only where required.
- `repos/ubb-events-frontend/src/features/events/EventWorkspace.tsx` — manager action controls, lifecycle gating, confirmation modals, mutation feedback, and invalidation hooks.
- `repos/ubb-events-frontend/src/features/events/EventEditPage.tsx` (or the established edit-page location) — event loading, T70 form mapping, PUT submission, and disabled terminal-state handling.
- `repos/ubb-events-frontend/src/App.tsx` — replace the edit placeholder and preserve T72 member/manager route guards.
- `repos/ubb-events-frontend/src/shared/api/types.ts` — shared error/detail types only if the existing client types cannot represent the confirmed response.
- `repos/ubb-events-frontend/src/styles/global.css` — action group, warning banner, and modal/form feedback styling when existing styles do not cover the named frames.
- `repos/ubb-events-frontend/cypress/e2e/event-actions.cy.ts` — focused manager/member, lifecycle, API, modal, error, accessibility, and navigation coverage.

### Tests to write

- **Acceptance criterion 1:** as an organization manager, verify Edit routes to `/org/:organizationId/events/:eventId/edit`, loads the event into T70's edit form, sends the exact PUT payload, returns to the workspace, and displays a success toast; verify completed/canceled events disable or hide Edit according to the description.
- **Acceptance criterion 1:** for a draft, verify the warning banner, primary Publish button, confirmation modal, exact `PATCH .../publish` request, success navigation/toast, and backend validation messages (including missing end date) rendered in the modal without closing it prematurely.
- **Acceptance criterion 1:** verify Delete is a danger action, its modal displays the backend participant count, exact `DELETE .../events/{eventId}` request is made, success returns to the organization events page, and an ended-event refusal remains visible with an error toast or modal message.
- **Acceptance criterion 1:** verify event detail, organization events, and discover queries are invalidated/refetched after successful edit, publish, and delete mutations.
- **Acceptance criterion 2:** run the action flows on a branch containing T70 and T72; assert the T70 form and T72 workspace remain functional and that the task is not implemented against an unmerged dependency base.
- **Acceptance criterion 3:** verify loading and disabled states during each mutation, form-level and field-level API errors, retryable detail failures, modal Escape/close and focus return, semantic button names, and that plain members see no Edit/Publish/Delete controls.

### Risks / open questions

- The backend contract is confirmed: `UpdateEventRequest` contains one `date`, start/end times, and no registration-information field; the edit page preserves the loaded category ID and maps the single backend date into the form date fields.
- The existing T72 actions slot is an empty `<div>`, so wiring controls may require a small prop or context change. Keep that change local to the event workspace and preserve existing route guards.
- Query keys are confirmed from the current hooks: `organization-event`, `organization-events`, and `events/discover`; mutations invalidate those exact keys.
- T73 was created from `T70-event-form-component` and synchronized with merged master before implementation.
- Frontend test commands are not specified in `context/project-context.md`; use the scripts declared by `repos/ubb-events-frontend/package.json` and record unavailable checks.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/event-actions.cy.ts` (initial manager publish and delete workflows).
- **Result:** `npx cypress run --browser electron --spec cypress/e2e/event-actions.cy.ts --reporter spec` — 0 passing, 2 failing. Both scenarios timed out because `[data-testid=event-action-edit]` / `[data-testid=event-action-delete]` were absent before implementation.
- **Failure reason:** This was behavioral red evidence for the missing manager action controls; the app loaded successfully with `VITE_API_URL` configured. Additional edit/member/terminal scenarios were added after implementation and are recorded as post-implementation coverage.

### Green (after implementation)

- **Result:** Focused `event-actions.cy.ts` — 8 passing, 0 failing, covering the inspected draft warning, publish/delete confirmation copy and icons, disabled edit end-date and registration-information fields, pending PUT submit state, lifecycle success/error paths, and manager/member gating. `npm run typecheck`, `npm run lint`, and `npm run build` — passed after the final UI and CSS changes. The routing regression spec passed all 10 tests.
- **Full suite of the affected repo:** The final full Cypress run completed 26 specs, 145 tests; 143 passed and 2 failed. Both failures are the existing `organizations-list.cy.ts` hidden-span visibility assertions; all T73, event-form, and routing tests passed.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | implementation | The edit form exposed an end date that the backend DTO cannot persist. | End date is visibly and semantically disabled in edit mode; create mode remains unchanged. | The user selected disabling end date in edit mode. |
| 2 | implementation | Registration information had the same edit-mode contract mismatch. | Registration information is visibly and semantically disabled in edit mode. | The user selected disabling registration information in edit mode. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use the existing T72 workspace as the action host, the backend's `ApiError` field errors in publish feedback, and React Query invalidation for detail, organization events, and discover keys. Reuse T70 `EventForm` for the edit page and preserve manager route guards.
- **Alternatives considered:** A separate action toolbar route was unnecessary; keeping actions in the shared workspace header preserves the T72 frame and member view. The backend delete endpoint returns `204`, so the existing event registration count is shown in the confirmation before the request.
- **Limitations:** The backend edit request has no registration-information or separate end-date field, so the edit page disables both corresponding edit inputs and sends only the confirmed DTO fields. Post-implementation focused coverage was added after the required red run. Active event-detail and organization-list refetches are asserted; discover invalidation is implemented with the exact `events/discover` query key, but no discover query is mounted during these mutation flows so Cypress cannot observe an HTTP refetch for that inactive cache. Figma frames `49:740`, `49:897`, `49:1055`, and `16:1061` were inspected in the connected session; implementation uses the existing UI primitives and available tokens, with no remote design assets added.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user (2026-11-16)
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push (2026-11-16)
- [x] Committed and pushed — frontend `8a2df3b` to `T73-event-actions-edit-publish-and`
- [x] PR created — [Frontend PR #25](https://github.com/Vasile07/ubb-events-frontend/pull/25)
- [x] PR reviewed — approved by user
- [x] **Gate 3:** merged by user — Frontend PR #25
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`


