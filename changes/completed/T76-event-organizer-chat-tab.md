# T76 - Event organizer chat tab

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T76
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** Source task 29 — Event organizer chat tab
- **Use case(s):** UC-14
- **Workflow:** workflow
- **Drive doc / ADR:** [Project requirements, use cases, workflows, API specification, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi)
- **Repo(s):** frontend
- **Branch:** `T76-event-organizer-chat-tab`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/36
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-organizer-chat.cy.ts`; `repos/ubb-events-frontend/cypress/e2e/routing-and-guards.cy.ts`; `npm run typecheck`; `npm run lint`; `npm run build`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 29 — Event organizer chat tab

**Description:** Implement page `Organization Screens`, frame `Org – Event: Organizer chat` (15:822): chat card with header ("Organizer conversation", lock note "Not visible to participants"), message list (other users on the left with avatar, name and time; the current user's messages on the right in the brand color) and a composer with Send.
Data from `GET` and `POST .../events/{eventId}/messages`.
Poll every ~5 s (`refetchInterval`) and keep the scroll at the bottom unless the user scrolled up; trim and validate text (not empty, backend max length with a counter near the limit); Enter sends, Shift+Enter adds a line.
Disable the composer for canceled events and show an empty state when there are no messages. Keep the data layer isolated so polling can be swapped for WebSocket later.
**Depends on:** T72 — Event workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` currently renders the chat route as a placeholder; tasks and registrations already compose the `EventWorkspace` shell. `src/features/events/EventWorkspace.tsx` loads event status and displays the canceled read-only state. `src/features/events/api.ts` and `src/shared/api/client.ts` provide the API client pattern; `src/features/auth/AuthProvider.tsx` exposes the current user's name and id. `src/shared/api/organizations.ts` already exposes the organization-members endpoint; its response includes member IDs and names, which can resolve other message senders and supply initials for the Figma avatar circles. Backend `MessageController`, `MessageResponse`, and `SendMessageRequest` define `GET`/`POST /api/organizations/{organizationId}/events/{eventId}/messages`; responses contain `id`, `senderId`, `content`, and `sentAt`, and content is nonblank with a 1000-character maximum.
- **Related completed tasks:** T72 — Event workspace shell (frontend dependency, merged as PR #21); T40 — Init Message entity; T43 — Real-time chat delivery (optional; WebSocket delivery exists, but T76 specifies polling). Their task records are in `changes/completed/`.
- **Constraints / assumptions:** Keep this task frontend-only as declared. Use the exact Figma target `Organization Screens` → `Org – Event: Organizer chat` (`15:822`); it shows member initials in avatar circles and the lock note “Not visible to participants.” The existing chat API is organization-scoped and a canceled event rejects POST with 409; frontend should disable composing based on the event status. Existing frontend scripts include `npm run typecheck`, `npm run lint`, `npm run build`, and `npm run cypress:run`. After the user's pull, the frontend repo is clean on `master` tracking `origin/master`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the chat as a route-level React feature inside the existing T72 `EventWorkspace` shell. Add a small API adapter and message/member types so the page owns query and mutation wiring while the visual component remains focused on rendering the Figma frame 15:822. Use React Query polling at approximately five seconds, with a mutation that invalidates the message query after a successful POST; keep this transport boundary independent of the rendering so a later WebSocket adapter can replace polling.

Render the event header/tabs through T72, then add a private conversation card with accessible message semantics, sender initials resolved from the organization-members query, and current-user messages aligned to the right. Use the event status already loaded by `EventWorkspace` to disable the composer for canceled events. Treat server validation/conflict responses as user-visible form errors, while preserving loading, empty, retry, and keyboard/focus states.

### Steps

1. **API/types:** Add typed `getEventMessages(organizationId, eventId)` and `sendEventMessage(organizationId, eventId, content)` functions for the existing organization-scoped GET/POST endpoints. Add message types for `id`, `senderId`, `content`, and nullable `sentAt`, reusing the existing organization-member response and current-user auth types for sender display. Keep the API adapter free of polling and UI concerns.
2. **Tests first:** Add focused Cypress coverage and the minimum page/API stubs needed to compile. Run the focused spec before the implementation and record a behavioral red result for the absent chat page, then expand the implementation only after the red evidence is captured.
3. **Route/UI integration:** Replace the chat placeholder in `src/App.tsx` with the guarded `EventWorkspace` plus a new `EventOrganizerChatPage`. Preserve the existing member guard and nested `/org/:organizationId/events/:eventId/chat` path. Render the Figma card header (“Organizer conversation”, lock icon/note), message list, empty state, loading/error states, and composer with accessible labels and disabled/read-only state for canceled events.
4. **Message data and presentation:** Query messages with a stable event-scoped key and `refetchInterval` of about 5 seconds; query organization members to map sender IDs to display names and initials, falling back safely when a sender cannot be resolved. Format `sentAt` into the displayed time and align bubbles by whether `senderId` matches the authenticated user. Keep message ordering from the API and avoid exposing participant-facing content beyond the private organizer card.
5. **Composer behavior:** Trim input for validation and submission, reject empty text, enforce the backend 1000-character maximum with a near-limit counter, send on Enter, preserve a newline on Shift+Enter, clear only after a successful POST, and show field/server errors without losing the draft. Disable Send and the editor while sending, when canceled, and when the request is invalid. Use a textarea so multiline messages remain possible.
6. **Scroll and accessibility:** Keep the message viewport at the bottom on initial load and when new polling results arrive only if the user was already near the bottom; preserve the user’s position after they scroll up. Use semantic headings, list/group labels, live status for loading/errors, keyboard-focusable controls, and responsive overflow behavior consistent with the existing frontend styles.
7. **Verification:** Run the focused Cypress chat spec, then `npm run typecheck`, `npm run lint`, `npm run build`, and the full available Cypress suite from `repos/ubb-events-frontend`; record exact red/green commands and any unrelated baseline failures in the task log.

### Files to create / modify

- `repos/ubb-events-frontend/src/App.tsx` (replace the chat placeholder with the guarded event workspace route).
- `repos/ubb-events-frontend/src/features/events/EventOrganizerChatPage.tsx` (new chat card, message list, composer, polling/scroll/accessibility behavior).
- `repos/ubb-events-frontend/src/features/events/api.ts` and `src/features/events/types.ts` (message request/response API functions and types, or a narrowly scoped chat API module if that matches the existing feature conventions).
- `repos/ubb-events-frontend/src/styles/global.css` (chat card, bubbles, composer, responsive and state styling using existing tokens).
- `repos/ubb-events-frontend/cypress/e2e/event-organizer-chat.cy.ts` (focused API/UI behavior coverage).

### Tests to write

- **Acceptance criterion 1 — behavior/API/design:** Cypress stubs the exact organization-scoped GET and POST message URLs, verifies the T72 event shell plus the private conversation header/lock note, left/right sender alignment, member initials/name/time, Send behavior, successful draft clearing, and polling refetch configuration. It also verifies the named chat route replaces the placeholder and matches the requested empty and populated Figma states.
- **Acceptance criterion 2 — dependency:** Before implementation, verify the completed T72 task record documents its merged frontend PR and that the local checkout contains the T72 `EventWorkspace` shell. Cypress then exercises that shell as regression coverage by confirming the five tabs remain present with Organizer chat active and event status data controls the chat page; the Cypress assertion is not treated as proof of merge.
- **Acceptance criterion 3 — states/accessibility:** Cover loading, empty, GET error with retry, POST validation error, POST canceled-event conflict, canceled-event disabled composer, 1000-character counter/limit, Enter versus Shift+Enter, disabled sending state, keyboard focus/labels, responsive message overflow, and scroll retention when the user is away from the bottom. Assert that a new polling result scrolls only when the viewport was already near the bottom.
- **Edge cases:** invalid route IDs, unresolved sender IDs (safe initials fallback), null `sentAt`, whitespace-only drafts, exactly 1000 characters, over-limit input, and a server response with no messages.

### Risks / open questions

- The backend message response has sender IDs but no embedded names or avatar URLs. The existing organization-members endpoint supplies names and IDs, so the plan uses it for initials and display names. This access is confirmed: the endpoint is guarded by `@RequireOrgPermission`, whose default role is `MEMBER`, so members and managers can fetch it.
- The API returns `sentAt` as an offset timestamp, but the Figma sample shows local clock text. Use the browser locale/timezone consistently and document this if visual snapshots require a fixed timezone.
- The description says “near the limit” without a threshold. Use a visible counter when the draft reaches a reasonable near-limit threshold (suggested: 900/1000) and enforce the exact backend maximum; confirm during plan review if a different threshold is required.
- Existing frontend test commands are documented as future work in `context/project-context.md`; use the scripts present in `package.json` and record unavailable checks rather than inventing commands.
- T43 provides optional WebSocket delivery, but this task remains polling-only as specified; the API/query boundary should make a later transport swap possible without changing the chat presentation.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `repos/ubb-events-frontend/cypress/e2e/event-organizer-chat.cy.ts` (two focused scenarios).
- **Result:** `npx cypress run --browser electron --spec cypress/e2e/event-organizer-chat.cy.ts --reporter spec` — 0 passing, 2 failing. Both tests reached the app and failed because `[data-testid=event-organizer-chat]` and the `No messages yet` state were absent from the placeholder route.
- **Failure reason:** This is a behavioral red result for the missing organizer chat page, not a compilation or test setup failure. Cypress 16.2.0 was installed and Vite was running on the configured port 5173.

### Green (after implementation)

- **Result:** Focused `npx cypress run --browser electron --spec cypress/e2e/event-organizer-chat.cy.ts --reporter spec` — 5 passing, 0 failing. A route compatibility check, `npx cypress run --browser electron --spec "cypress/e2e/event-organizer-chat.cy.ts,cypress/e2e/routing-and-guards.cy.ts" --reporter dot`, passed 15/15 after restoring the existing `page-org-event-chat` test marker. `npm run typecheck`, `npm run lint`, and `npm run build` passed.
- **Full suite of the affected repo:** Latest `npx cypress run --browser electron --reporter dot` completed all 33 specs (195 tests): 192 passed, 3 failed. The `my-registrations-screen.cy.ts` spec now passes 6/6 after its active-event fixture date was moved into the future. Remaining failures: `event-actions.cy.ts` — `deletes successfully, navigates to the organization events list, and refetches it`; and two hidden-marker visibility assertions in `organizations-list.cy.ts`. The route-map expectation for the old chat placeholder marker was fixed by restoring `page-org-event-chat`; focused chat and route-guard specs pass 15/15. The remaining failures do not touch T76’s feature code; their base-branch status was not established.
- **TDD deviation:** The initial pre-implementation red run covered two core scenarios and confirmed the placeholder lacked the chat page and empty state. Three additional scenarios for retry/POST behavior, failed-draft and keyboard behavior, polling and scroll retention were added after feature implementation. Their passing results verify current behavior but are not pre-implementation red evidence.

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

- **Decisions:** The implementation uses a transport adapter in `features/events/api.ts` and React Query polling at 5 seconds, keeping the page independent from the optional WebSocket delivery. Sender names and initials are resolved from the existing organization-members query; current-user alignment uses the authenticated user id. The composer trims before POST, enforces the backend 1000-character limit, submits on Enter, preserves Shift+Enter newlines, and retains the draft on errors. Scroll position stays at the bottom for new messages unless the user is reading earlier messages. The separate `my-registrations-screen.cy.ts` fixture date was moved into the future so it still exercises an active registration under the current clock; no production implementation for that screen changed.
- **Alternatives considered:** A WebSocket client was not introduced because the task explicitly specifies polling and the backend already exposes the REST message contract. Message responses do not include sender names, so fetching organization members was used instead of adding a backend change outside this frontend task.
- **Limitations:** The latest full Cypress run completed 33 specs with 192/195 tests passing. Three failures remain in unrelated code: one event-actions refetch assertion and two organizations-list hidden-marker visibility assertions. Their status on the base branch was not established. The pre-implementation red run covered two core scenarios; three broader scenarios were added after implementation and are recorded as a TDD deviation.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — frontend commits `43fb01f`, `ce4613c`
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`


