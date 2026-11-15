# T62 - Event details screen (participant)

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T62
- **Type:** workflow
- **Owner:** Karina Kentsch
- **Requirement(s):** 
- **Use case(s):** UC-10, UC-11, UC-15, UC-18
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T62-event-details-screen-participant`
- **PR link(s):** 
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-details.cy.ts`; `npm run typecheck`; `npm run lint`; `npm run build`; full Cypress suite

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 15 — Event details screen (participant)

**Description:** Implement page `Student Screens`, frames `Event details` (13:417) and `Event details – registered` (13:538) as one route with two states.
Left: hero (status badge, organization name, title), info tiles for Date & time and Location only, "About this event" with the registration information block.
Right: registration card (places left, progress bar, "X of Y places taken", action) and the organizer card. Capacity appears only in the right card; there is no deadline and no registered-on date.
Actions:
- Register `POST /api/events/{id}/registrations`
- Cancel registration `DELETE /api/events/{id}/registrations/me` after a confirmation modal

Both update the UI optimistically and invalidate the event, my-registrations and discover queries; show success/error toasts.
Handle 409 (already registered, show the current status), full, started, completed and canceled events by disabling the button with an explanatory text.
When registered, also render the Announcements card from `GET /api/events/{id}/announcements` (hide on 403, empty state when none).
**Depends on:** T52 — Base UI components: layout and data display; T53 — Feedback components: toast, empty state, skeleton, error state; T54 — Event card and event status logic; T57 — App shell and top navigation.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-frontend/src/App.tsx` routes `/events/:eventId` to `EventDetailsPage`; `src/features/events/api.ts` and `types.ts` now include event detail, registration, and announcements calls/types; `src/features/events/DiscoverEventsPage.tsx` shows React Query, API error, toast, and event-status usage; `src/shared/ui/EventCard.tsx` and `src/shared/lib/eventStatus.ts` provide reusable event presentation/status logic; `src/shared/ui/FeedbackComponents.tsx` provides toasts, empty/error states, and skeletons; `src/shared/lib/queryClient.ts` provides query client defaults; `src/styles/global.css` contains shared UI styles. Focused Cypress coverage is in `repos/ubb-events-frontend/cypress/e2e/event-details.cy.ts`.
- **Related completed tasks:** `T52-base-ui-components-layout-and.md` (merged, frontend PR #5); `T53-feedback-components-toast-empty-state.md` (merged, frontend PR #7); `T54-event-card-and-event-status.md` (frontend PR #6); `T57-app-shell-and-top-navigation.md` (merged, frontend PR #10).
- **Constraints / assumptions:** The task’s stated T52, T53, T54, and T57 UI dependencies are merged. Frontend repo is on branch `T62-event-details-screen-participant`; the user-requested `../../.idea` ignore edit is preserved for a future commit. Backend `GET /api/events/{eventId}` returns `EventDetailsResponse` fields `id`, `title`, `description`, `organization { id, name }`, `date`, `startTime`, `endTime`, `location`, `capacity`, `registrationCount`, `registrationAvailable`, `status`, and `registrationStatus`; POST registration returns 201 with `{ id, eventId, userId }`; DELETE returns 204. Registration status is `NOT_REGISTERED`, `REGISTERED`, or `CANCELLED`; start is determined from date/time. Backend `GET /api/events/{eventId}/announcements` returns a list of `AnnouncementResponse { id, eventId, title, content }`, ordered by ID; access is allowed to registered participants and active organization members, otherwise 403. The Figma plugin is not connected, so frames `Student Screens` `13:417` and `13:538` have not been visually inspected.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the participant event-details route as a feature-level page backed by typed event-detail, registration, and announcement API functions. Keep server state in React Query, derive the registration action state from the event status and current registration, and use optimistic cache updates with rollback on mutation failure. Reuse the existing shell, status badge, progress bar, toast, modal/dialog, skeleton, empty, and error primitives. The two Figma frames become one responsive page whose registered state is selected from query data; no capacity or registration metadata will be duplicated in the left content column.

The event detail, registration, and announcement response contracts were confirmed from backend code. Event and registration calls use their backend DTOs; announcements render title and content from the backend response.

### Steps

1. **API contract verification (frontend/API):** Confirm and use the backend event detail, registration, and announcement DTOs recorded in Context; preserve the registered-participant and active-member access behavior, including 403 handling.
2. **Tests first (frontend/UI and API):** Add focused Cypress coverage with stubbed API responses for loading, successful event rendering, registered state, register/cancel actions, optimistic updates, rollback/toasts, 409 current-status handling, full/started/completed/canceled disabled states, announcement 403 hiding, and announcement empty state. Include semantic labels, confirmation dialog behavior, and keyboard accessibility.
3. **Feature API and query model (frontend/API):** Add typed functions for `GET /api/events/{id}`, `POST /api/events/{id}/registrations`, `DELETE /api/events/{id}/registrations/me`, and `GET /api/events/{id}/announcements`, mapping `ApiError` status 409 and 403 without exposing unsafe details. Define stable query keys for the event, my registrations, discover events, and announcements.
4. **Event details UI (frontend/UI):** Replace the authenticated `/events/:eventId` placeholder with an `EventDetailsPage` inside the existing `AppShell`. Render the hero, status/organization/title, date and location tiles, About section and registration-information block, right-side registration card with places/progress/action, and organizer card using the named Figma frames `13:417` and `13:538`. Keep capacity only in the right card and omit deadline/registered-on fields.
5. **Registration actions and rules (frontend/UI/state):** Implement register and cancel mutations with optimistic cache updates, rollback on errors, success/error toasts, and invalidation of event, my-registrations, and discover queries. Require a confirmation modal before cancellation. Disable the action with explanatory text for already registered, full, started, completed, and canceled states; on 409, refetch/show the current registration status.
6. **Announcements and feedback (frontend/UI):** Fetch announcements only for the registered state. Hide the card on 403, show the shared empty state when the response is empty, and use shared loading skeleton and error states for event/announcement failures. Ensure accessible headings, progress semantics, dialog focus/labels, and responsive layout.
7. **Verification (frontend):** Run the focused Cypress spec first, then `npm run typecheck`, `npm run lint`, `npm run build`, and the full Cypress suite. Record a behavioral red run before implementation and green results afterward.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/EventDetailsPage.tsx` (new participant page and registration/announcement UI).
- `../../repos/ubb-events-frontend/src/features/events/api.ts` and `src/features/events/types.ts` (confirmed event-detail, registration, announcement contracts and API calls).
- `../../repos/ubb-events-frontend/src/App.tsx` (route integration, preserving auth guard and shell).
- `../../repos/ubb-events-frontend/src/styles/global.css` (event-details layout, cards, modal, responsive rules).
- `repos/ubb-events-frontend/cypress/e2e/event-details.cy.ts` (focused behavior and accessibility coverage).
- Existing shared UI files only if a required dialog or event-details primitive is genuinely absent; extend the shared barrel when needed.

### Tests to write

- **Acceptance criterion 1 / page behavior:** Cypress verifies both Figma states, all required content, correct capacity placement, status badge, organizer card, and route rendering under the authenticated shell.
- **API actions and optimistic state:** Tests stub successful register and cancel calls, assert immediate UI changes, toast messages, and invalidation/refetch of event, my-registrations, and discover queries; cancel requires confirmation.
- **Business/status rules:** Tests cover 409 already-registered with current status, full, started, completed, canceled, and already-registered action disabling with explanatory text.
- **Announcements:** Registered users trigger the specified announcements GET; 403 hides the card, an empty response shows the shared empty state, and a nonempty response renders announcement titles and content from the confirmed DTO.
- **Acceptance criterion 2 / dependencies:** Test setup runs against the merged T52, T53, T54, and T57 frontend code.
- **Acceptance criterion 3 / feedback/accessibility:** Tests cover event loading/error, announcement loading/error behavior, skeleton/empty states, semantic progress and headings, dialog labels/focus/keyboard dismissal, and responsive layout.

### Risks / open questions

- Backend event, registration, and announcement contracts are confirmed from backend code. Announcement GET allows registered participants and active organization members; the UI hides the card on 403 as specified.
- The task references Figma frames `13:417` and `13:538`; use them as the visual target once the Figma plugin is connected.
- Backend does not expose a separate started status; the screen derives started/completed states from event date, start time, and end time.
- Reuse the shared `Modal` primitive for focus management, Escape handling, and focus restoration.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-details.cy.ts` (3 tests), written before implementation.
- **Result:** `$env:CYPRESS_CACHE_FOLDER='C:\Users\Karina Kentsch\AppData\Local\Cypress\Cache'; npx cypress run --spec cypress/e2e/event-details.cy.ts --browser electron --reporter spec --config baseUrl=http://192.168.0.122:5173` produced 0 passing and 3 failing. The assertions for `[data-testid=event-details-page]`, `[data-testid=registration-card]`, and `[data-testid=register-event]` timed out on the placeholder route.
- **Failure reason:** The failures were behavioral: the event-details page, registration card, and registration action were absent before implementation.

### Green (after implementation)

- **Result:** Final focused command `$env:CYPRESS_CACHE_FOLDER='C:\Users\Karina Kentsch\AppData\Local\Cypress\Cache'; npx cypress run --spec cypress/e2e/event-details.cy.ts --browser electron --reporter spec --config baseUrl=http://192.168.0.122:5173` passed 9/9, including announcement DTO title/content rendering. Additional checks for registration/cancellation, 409 refetch, progress semantics, terminal states, announcement 403, and announcement DTO rendering were added after implementation; they are post-implementation regression checks, not pre-implementation red evidence.
- **Full suite of the affected repo:** `npm run typecheck`, `npm run lint`, and `npm run build` passed. Final full command `$env:CYPRESS_CACHE_FOLDER='C:\Users\Karina Kentsch\AppData\Local\Cypress\Cache'; npx cypress run --browser electron --reporter dot --config baseUrl=http://192.168.0.122:5173` passed all 61 tests across 12 specs.

- **TDD deviation:** The initial 3 tests produced the required behavioral red before implementation. Additional tests for registration/cancellation, 409 refetch, progress semantics, terminal states, and announcement states were added afterward; these are regression checks, not pre-implementation red evidence. After the backend DTO was confirmed, a post-implementation check exposed that announcement title/content were not rendered; the frontend was updated and the final focused run passed 9/9. A first expanded run also failed because its selector clicked the dialog heading instead of its button; the selector was corrected.

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

- **Decisions:** Use one authenticated route with query-driven registered/unregistered states. Keep event, registration, and announcement API types aligned with backend DTOs, derive started/completed from event date/time, use optimistic React Query updates with rollback, and reuse shared ProgressBar and Modal primitives. Render announcement title/content from the confirmed DTO and hide the card on 403.
- **Alternatives considered:** A second route for the registered screen was rejected because the task defines one route with two states. A custom progress element and dialog were replaced with shared primitives for accessibility and focus behavior.
- **Limitations:** The Figma plugin was unavailable, so visual comparison with frames `13:417` and `13:538` remains pending.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [ ] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``










