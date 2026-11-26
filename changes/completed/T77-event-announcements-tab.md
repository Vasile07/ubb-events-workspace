# T77 - Event announcements tab

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T77
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** ORGZ-04 — Announcement forum.
- **Use case(s):** UC-15
- **Workflow:** Project task-list workflows 45 (Publish event announcements) and 46 (View event announcements).
- **Drive doc / ADR:** [Project requirements, use cases, workflows, API specification, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi) — ORGZ-04 / UC-15; local task references: `docs/task-list.md` items 45–46 and `docs/use-cases.md` UC-15.
- **Repo(s):** frontend
- **Branch:** `T77-event-announcements-tab`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/38
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-announcements.cy.ts`; `npm run typecheck`; `npm run lint`; `npm run build`; full Cypress suite.

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 30 — Event announcements tab

**Description:** Implement page `Organization Screens`, frame `Org – Event: Announcements` (16:431): list of announcements (avatar, author with role, time, title, content, newest first) and the "New announcement" composer card (title, content, Publish).
Create with `POST .../events/{eventId}/announcements` (members and managers), read with `GET /api/events/{eventId}/announcements`.
Validate required title and content, refresh the list and clear the form after publishing, show a toast, hide the composer for canceled events and show an empty state when there are none. Reuse the `AnnouncementItem` component built for the participant view in task 15.
**Depends on:** T62 — Event details screen (participant); T72 — Event workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules (as refined in the approved plan and Human Corrections).
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` already routes the announcements tab through a placeholder inside the T72 shell; `src/features/events/EventWorkspace.tsx` exposes event status and marks canceled/completed terminal; `src/features/events/api.ts` and `types.ts` already provide participant announcement GET and `AnnouncementResponse` (id, eventId, title, content); `EventDetailsPage.tsx` renders those fields; no reusable `AnnouncementItem` component exists yet. The backend GET service currently orders by announcement ID ascending (`AnnouncementRepository.findAllByEventIdOrderByIdAsc`), and IDs use database identity generation. Existing API, query, form, toast, empty/error, and Cypress patterns are in `EventOrganizerChatPage.tsx`, `EventTasksPage.tsx`, `shared/api/client.ts`, and `cypress/e2e/`. - **Related completed tasks:** `changes/completed/T62-event-details-screen-participant.md` and `T72-event-workspace-shell.md`; both record Gate 3 merged (T72 frontend PR #21). - **Constraints / assumptions:** Frontend repo is clean on `master`, has no unpushed commits, and is up to date with `origin/master`. Implement only the frontend workflow described using the existing `AnnouncementResponse` fields (id, eventId, title, content). Follow the organizer chat pattern for canceled events: keep the composer visible but disabled and show a status message. The T77 acceptance criteria require dependency merge, loading/empty/error/validation/accessibility behavior, and API interactions to be testable.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Build the organization announcements view as a vertical frontend slice inside the existing T72 event workspace. Add the organization-scoped publish mutation, reuse the existing announcement GET helper and `AnnouncementResponse`, extract a shared `AnnouncementItem` for the participant and organization views using only id, eventId, title, and content, and add a page that derives canceled/read-only state from the workspace event query. Keep validation and UI state in the page, use the existing React Query, API error, toast, loading, empty, and accessibility patterns, and sort the announcement list by descending ID so newest items render first, and invalidate the query after a successful publish.

### Steps

1. **API/types:** use the existing `AnnouncementResponse` (id, eventId, title, content) and GET helper; add the organization-scoped `POST /api/organizations/{organizationId}/events/{eventId}/announcements` helper, and preserve safe `ApiError` field-level handling.
2. **Shared UI:** extract or create the reusable `AnnouncementItem` component and styles, rendering title and content from `AnnouncementResponse`. Update the participant announcement card to use it without changing its registration/403 behavior.
3. **Organization page:** replace the App route placeholder with an `EventAnnouncementsPage` that loads the organization event and announcements, shows loading/error/retry and empty states, sorts announcements by descending ID (the current backend returns ascending IDs), keeps the composer visible but disables its fields and publish action for canceled events, with a status message following the organizer chat pattern; validates required title/content before submit. On success clear the form, invalidate the list, and show a toast; map API validation and general errors to existing field/alert patterns.
4. **Styling/accessibility:** match the named Figma frame using existing design tokens and components; provide labels, keyboard-submit behavior, status/alert announcements, disabled/loading controls, and stable test IDs for the page, list, item, composer, and states.
5. **Verification:** write Cypress coverage before implementation for loading, empty, newest-first rendering by descending ID, validation, successful POST plus refresh/reset/toast, API failure, canceled read-only behavior, and the participant component regression; then run the focused spec and the affected frontend checks.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/events/EventAnnouncementsPage.tsx` (new organization tab page).
- `repos/ubb-events-frontend/src/features/events/AnnouncementItem.tsx` (new shared presentation, or the existing participant component if found during implementation).
- `repos/ubb-events-frontend/src/features/events/api.ts`, `src/features/events/types.ts` (announcement request/response helpers and types).
- `repos/ubb-events-frontend/src/features/events/EventDetailsPage.tsx`, `src/App.tsx` (reuse component and wire the route).
- `repos/ubb-events-frontend/src/styles/global.css` (scoped announcements/composer styles).
- `repos/ubb-events-frontend/cypress/e2e/event-announcements.cy.ts` (new focused workflow coverage).

### Tests to write

- Organization route renders loading, API error with retry, empty state, and announcement list with newest-first ordering.
- Required title/content validation prevents POST and exposes accessible field errors; canceled events disable the composer and publish action.
- Successful POST sends the organization/event path and body, clears both fields, invalidates/refetches the list, and shows a success toast; failed POST displays field or general error without losing the draft.
- Participant event details continue rendering announcements through the shared item and keep the existing 403-hidden behavior.
- Add/update Cypress fixtures to cover the existing `AnnouncementResponse` shape, then run the focused spec plus `npm run typecheck`, `npm run lint`, and `npm run build` as applicable.

### Risks / open questions

- The response has no timestamp, but the current backend returns announcements by ascending ID and IDs are database-generated identity values. The frontend can sort by descending ID without a backend change; this assumes higher IDs represent later announcements.

## Test log
<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/event-announcements.cy.ts` — renders newest announcements first and publishes a new announcement; validates required fields and disables the composer for canceled events.
- **Result:** Command `$env:CYPRESS_CACHE_FOLDER='C:\Users\Armand\AppData\Local\Cypress\Cache'; npm run cypress:run -- --spec cypress/e2e/event-announcements.cy.ts` produced 0 passing, 2 failing.
- **Failure reason:** The route rendered the existing Event announcements placeholder, so `[data-testid=event-announcements-page]` and the Publish announcement button were absent. This is the expected behavioral failure for the missing page implementation.

### Green (after implementation)

- **Result:** Initial focused command `$env:CYPRESS_CACHE_FOLDER='C:\Users\Armand\AppData\Local\Cypress\Cache'; npx cypress run --spec cypress/e2e/event-announcements.cy.ts --reporter spec` passed: 2 tests, 2 passing. Post-implementation test expansion added loading, empty, retry, POST payload, and failed-POST draft-retention coverage; the same command then passed 5 tests, 5 passing.
- **Full suite of the affected repo:** `npm run typecheck` passed; `npm run lint` passed; `npm run build` passed. The final focused T77 spec passed 5/5. The final full Cypress run completed 35 specs with 205 tests: 202 passing and 3 failing (one existing failure in `event-actions.cy.ts` and two existing visibility failures in `organizations-list.cy.ts`). The T77 spec passed 5/5 and `routing-and-guards.cy.ts` passed 10/10.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | Do not use fields like avatar, author role or time. Just use AnnouncementResponse. Use the existing chat pattern. | The plan now uses only `id`, `eventId`, `title`, and `content`, and keeps the canceled-event composer visible but disabled with a status message. | Applied the requested plan corrections. |

## Design decisions, alternatives, limitations
<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use the existing `AnnouncementResponse` fields only; sort by descending generated ID because the backend response has no timestamp; keep the canceled-event composer visible but disabled following the organizer chat pattern; reuse a shared `AnnouncementItem` in participant and organization views.
- **Visual verification:** Compared the implementation against Figma `Organization Screens` frame `Org – Event: Announcements` (16:431): the page uses the existing workspace shell with a two-column announcement feed/composer layout, light surface background, white rounded cards, and existing typography/color tokens. Avatar/author/time content from the reference was intentionally omitted per the approved scope.
- **Alternatives considered:** Changing the backend response to add timestamps was outside the approved frontend scope. Hiding the canceled composer conflicted with the user-approved chat pattern.
- **Limitations:** ID ordering is a proxy for recency. Full-suite Cypress has three unrelated failures: one in `event-actions.cy.ts` and two in `organizations-list.cy.ts`; the focused T77 tests and frontend typecheck, lint, and build pass.

- **Implementation verification deviation:** The expanded loading, empty, retry, payload, and failed-POST tests were added after implementation, so they are recorded as post-implementation verification rather than pre-implementation red evidence. Existing `event-details.cy.ts` covers participant announcements rendering and hiding on a 403, including the shared `AnnouncementItem` behavior.

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
- [x] **Gate 3:** merged by user — Frontend PR #38
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`















