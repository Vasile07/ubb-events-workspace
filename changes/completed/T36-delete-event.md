# T36 - Delete event

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T36
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** EVT-05, EVT-06
- **Use case(s):** UC-17
- **Workflow:** workflow
- **Drive doc / ADR:** [Requirements, use cases, architecture, ERD, API specification, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban) — EVT-05, EVT-06, UC-17
- **Repo(s):** backend
- **Branch:** `T36-delete-event`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/38
- **Tests:** `EventServiceTest`, `EventControllerApiTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Delete event” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** EVT-05, EVT-06. **Depends on:** Organization-scoped authorization, Event lifecycle model, Register for event.
- `DELETE /api/organizations/{organizationId}/events/{eventId}`, for managers: a soft delete (set the `deleted` flag), refused for an event that has already ended, with the reason. Returns 204.
- The event is returned with the status CANCELED by the public endpoints (unless it was a draft) and by the organization endpoints. It cannot be edited, published or registered for, and its chat and announcements become read-only.

Dependencies from the backlog: 9, 22, 32.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the user-approved clarification recorded in Human Corrections.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** Backend `Event` / `EventStatus`, `EventRepository`, `EventService` / `EventServiceImpl`, and `EventController`; manager authorization uses `@RequireOrgPermission(EffectiveOrgRole.MANAGER)` and the organization authorization interceptor. `EventServiceImpl` already defines ended as current time at or after the event end time (falling back to start time when end time is absent). Registration rejects non-PUBLISHED events. Chat and announcement service implementations are currently empty scaffolds. Relevant tests: `EventServiceTest`, `EventControllerApiTest`, and `EventControllerStructureTest`.
- **Related completed tasks:** T9 (organization-scoped authorization), T19 (Event entity), T23 (create event), T24 (view organization events), T28 (publish event), T29 (discover events), T31 (inspect event), T32 (register for event), T33 (cancel registration), T35 (view event registrations), T40 (chat message entity), T44 (announcement entity).
- **Constraints / assumptions:** Backend only; backend repository is clean on `master` and synced with `origin/master`. The event model currently has no `deleted` field, and no event delete API/service exists. `EventStatus` includes `DRAFT`, `PUBLISHED`, and `CANCELED`; public discovery selects published events, inspection hides drafts, and organization endpoints expose event status. Shared errors map not found to 404 and conflicts to 409. Existing event-ended logic uses end time or start time fallback.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add a manager-only DELETE endpoint and implement deletion by changing the event's existing status to `CANCELED`. Do not add a database column. This makes the deletion visible to organization and public inspection endpoints, excludes the event from published-event discovery, and prevents publication and registration through the existing lifecycle checks. Per the user's clarification, deleted drafts also become `CANCELED`. An already-canceled event is treated as already deleted and rejected with a conflict, preserving the previously reviewed repeated-delete behavior.

### Steps

1. **Model / persistence:** use the existing `EventStatus` field only; remove the superseded `deleted` property and related persistence/query predicates. Confirm repository discovery continues selecting only `PUBLISHED` events.
2. **Business service:** add transactional `deleteEvent(organizationId, eventId)`. Load by organization, return 404 when missing, reject an already-canceled event with 409, reject events ended at or before the injected clock (using the end time or start-time fallback) with a client-safe 409 reason, set status to `CANCELED`, and save. Ensure public and organization responses expose the stored status; canceled events remain unavailable for publication and registration.
3. **API:** add `DELETE /api/organizations/{organizationId}/events/{eventId}` to `EventController`, require `EffectiveOrgRole.MANAGER`, return 204, and document 204/401/403/404/409 responses.
4. **Verification:** write tests first and confirm behavioral red before implementing the status transition. Cover published and draft deletion, ended/missing/already-canceled cases, status returned in organization and public inspection, public discovery exclusion, registration and publication refusal, manager authorization, and the 204 response. Then run focused tests and the full backend suite with the Gradle wrapper.

### Files to create / modify

- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/EventService.java` and `implementation/EventServiceImpl.java`.
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/EventController.java`.
- Existing event, registration, and controller tests under `Business`, `Persistence`, `Model`, and `RestServices`; restore the model test assertion that `Event` has no `deleted` field.

### Tests to write

- `EventServiceTest`: deleting a future published event and a future draft sets status to `CANCELED`; ended events fail with a reason; missing and already-canceled events fail; organization and public inspection return `CANCELED`; publication is rejected afterward.
- `EventControllerApiTest` / `EventControllerStructureTest`: exact DELETE route, manager-only authorization, 204 response, unauthenticated/member rejection, and shared 404/409 responses.
- `EventRepositoryTest`: published-event discovery continues to exclude canceled events.
- `RegistrationServiceTest`: canceled events cannot be registered for.

### Risks / open questions

- The backlog sentence “unless it was a draft” is superseded by the user's clarification that all deleted events, including drafts, become `CANCELED`.
- A canceled event does not record whether it was canceled by this endpoint or another workflow; DELETE treats any already-canceled event as an already-completed deletion and returns 409.
- Chat and announcement services are empty scaffolds, and no event-edit workflow exists in this checkout. Their future read-only/edit guards remain integration points for those workflows; this task will not invent endpoints.
## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventServiceTest.deleteSetsFuturePublishedEventCanceledAndSavesIt`; `EventServiceTest.deletingDraftSetsCanceledStatusInOrganizationView`.
- **Result:** `.\gradlew.bat --no-daemon --console=plain --info :Business:test --tests "com.ubb.events.implementation.EventServiceTest.deleteSetsFuturePublishedEventCanceledAndSavesIt" --tests "com.ubb.events.implementation.EventServiceTest.deletingDraftSetsCanceledStatusInOrganizationView"` compiled and ran both tests. Both failed: expected `CANCELED` but observed `PUBLISHED` and `DRAFT`, respectively.
- **Failure reason:** The delete service still set the superseded `deleted` flag without changing `EventStatus`. These behavioral failures demonstrated the planned status transition was missing.

### Green (after implementation)

- **Result:** `.\gradlew.bat --no-daemon --console=plain :Model:test --tests "com.ubb.events.domain.EventTest" :Business:test --tests "com.ubb.events.implementation.EventServiceTest" --tests "com.ubb.events.implementation.RegistrationServiceTest" :Persistence:test --tests "com.ubb.events.persistence.EventRepositoryTest" :RestServices:test --tests "com.ubb.events.controllers.EventControllerApiTest" --tests "com.ubb.events.controllers.EventControllerStructureTest"` completed `BUILD SUCCESSFUL` after the status-only implementation. The current service and API test classes were then rerun with ` .\gradlew.bat --no-daemon --console=plain :Business:test --tests "com.ubb.events.implementation.EventServiceTest" :RestServices:test --tests "com.ubb.events.controllers.EventControllerApiTest"`; `BUILD SUCCESSFUL`.
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` ran all modules. It failed on three previously recorded baseline tests: `OrganizationServiceStructureTest.implementationIsAnnotatedAndImplementsServiceWithoutOperations`, `OrganizationServiceStructureTest.serviceInterfaceIsEmpty`, and `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`. Model and Persistence passed; the T36-specific Business and RestServices tests passed.
## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “Deleting an event means to set the event's status to CANCELED, do not add a new column in the database, the rest of the rules remain.” | Replace the proposed `deleted` flag with the `CANCELED` status and retain the remaining rules. | The user clarified the deletion state and rejected a new database column. |
| 2 | plan | “Yes, all deleted events become CANCELED.” | Apply the status-only deletion rule to drafts as well as other events. | The user confirmed that the draft exception does not apply. |
## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Deletion uses the existing status field: every deletable event, including a draft, transitions to `CANCELED`. No database column is added. Ended-event and already-canceled conflicts use the shared 409 response.
- **Alternatives considered:** A separate deletion flag could preserve the distinction between a canceled event and a deleted draft, but the user rejected a new column and confirmed drafts should also become `CANCELED`.
- **Limitations:** The status alone cannot distinguish deletion via this endpoint from cancellation by another workflow; already-canceled events are treated as already deleted. Chat, announcement, and event-edit workflows are absent here, so future read-only/edit guards remain with those workflows. The full backend suite has three recorded baseline failures.
## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — `91ebf0f` (`T36: implement event deletion`)
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user — PR #38
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
