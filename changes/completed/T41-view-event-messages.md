# T41 - View event messages

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T41
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** ORGZ-05, ORG-05
- **Use case(s):** UC-14
- **Workflow:** workflow
- **Drive doc / ADR:** N/A
- **Repo(s):** backend
- **Branch:** `T41-view-event-messages`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/43
- **Tests:** `Persistence/src/test/java/com/ubb/events/persistence/MessageRepositoryTest.java`; `Business/src/test/java/com/ubb/events/implementation/MessageServiceTest.java`, `MessageServiceStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MessageControllerApiTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View event messages” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORGZ-05, ORG-05. **Depends on:** Organization-scoped authorization, Init ChatMessage entity.
- `GET /api/organizations/{organizationId}/events/{eventId}/messages`: the messages of the event, for members and managers. No messages returns an empty list, and others get 403.
- Available from the draft state. Not paginated.

Dependencies from the backlog: 9, 40.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Message.java` and `Event.java` define message/event associations; `Persistence/src/main/java/com/ubb/events/persistence/MessageRepository.java` is currently a bare `JpaRepository`; `Business/src/main/java/com/ubb/events/MessageService.java` and `implementation/MessageServiceImpl.java` are empty scaffolds; `RestServices/src/main/java/com/ubb/events/security/OrganizationAuthorizationInterceptor.java` and `RequireOrgPermission.java` enforce membership roles on organization routes; `RestServices/src/main/java/com/ubb/events/controllers/EventTaskController.java` provides the analogous organization/event GET endpoint.
- **Related completed tasks:** `changes/completed/T40-init-chatmessage-entity.md` added the Message entity and scaffolds; `changes/completed/T9-organization-scoped-authorization.md` documents membership authorization.
- **Constraints / assumptions:** Backend only; Java 21, Spring Boot, JPA, JUnit 5, Gradle wrapper, and layered architecture per `context/project-context.md`. The backend repo was clean on `T39-view-event-follow-up` with no unpushed commits; it is now clean and synchronized on `master`. T40 provides an Event relation on Message. Organization routes are intercepted; each new handler must declare `@RequireOrgPermission`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the vertical read-only workflow across persistence, business, and REST layers. Add an event-scoped repository query, validate that the event belongs to the requested organization, map messages to a small response DTO, and expose the specified GET route with member-level organization authorization. The controller will contain mapping and HTTP documentation only; the service will own event existence/organization validation and message retrieval. The route is intentionally available for draft events and returns an empty JSON list when no messages exist.

The plan assumes the response contains the message identifier, sender identifier, and content, which are the stable fields available from the current Message entity. If the API contract requires sender profile fields or a different JSON shape, that must be confirmed before implementation.

### Steps

1. **Tests first (persistence, service, API):** add focused tests for the event-scoped repository query, organization/event validation, response mapping, empty results, authenticated member and manager access, unauthenticated access, non-member 403, and draft-event access. Confirm a behavioral red result before implementation.
2. **Persistence:** extend `MessageRepository` with an ordered `findAllByEventId` query (using message id as the deterministic order) for the event message list.
3. **Business:** add `listEventMessages(organizationId, eventId)` to `MessageService` and implement it in `MessageServiceImpl`. Verify the event exists under the organization through `EventRepository`; throw the shared not-found error otherwise. Load messages and map them to the response DTO without applying event-status restrictions.
4. **Model/API DTO:** create an immutable `MessageResponse` containing the agreed message id, sender id, and content fields.
5. **API:** add `GET /api/organizations/{organizationId}/events/{eventId}/messages` to `MessageController`, annotate it with `@RequireOrgPermission` (default member minimum), return `200 OK` with a non-paginated list, and document 200, 401, 403, and 404 responses. Delegate directly to `MessageService`.
6. **Verification:** run the focused tests to green, then run the full backend test suite and record exact commands/results in the task test log.

### Files to create / modify

- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MessageRepository.java` (add event query).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MessageService.java` (add list method).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/MessageServiceImpl.java` (implement validation, retrieval, mapping).
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/MessageResponse.java` (create response DTO; exact sender fields require confirmation).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MessageController.java` (add authorized GET handler and OpenAPI annotations).
- `repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/MessageServiceTest.java` (create/update).
- `repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/MessageControllerApiTest.java` (create).
- `repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence/MessageRepositoryTest.java` (create/update as needed).

### Tests to write

- Acceptance criterion 1: an API-level happy-path test proves a member receives messages for an event in the requested organization; a manager receives the same endpoint; a draft event is readable; and a service/repository test proves the returned messages belong to the requested event and are deterministically ordered.
- Acceptance criterion 2: API tests cover unauthenticated `401`, non-member `403`, organization/event mismatch `404`, and an event with no messages returning `200` plus `[]`. Service tests cover missing event and repository mapping. Tests must be written before implementation and demonstrate a behavioral failure caused by the missing workflow.

### Risks / open questions

- The backlog does not define the JSON response schema or message ordering. Proposed contract is `{id, senderId, content}` ordered by message id ascending; confirm this before implementation.
- The backlog says the endpoint is available from draft state but does not explicitly define behavior for other event states. The plan applies no status restriction, matching the stated draft availability and the read-only nature of the endpoint; confirm if a status allow-list is required.
- Organization authorization is path-based and checks membership before service execution; event ownership still needs service validation to prevent cross-organization access.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Persistence/src/test/java/com/ubb/events/persistence/MessageRepositoryTest.java`; `Business/src/test/java/com/ubb/events/implementation/MessageServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MessageControllerApiTest.java`.
- **Result:** `./gradlew.bat :Persistence:test --tests com.ubb.events.persistence.MessageRepositoryTest :Business:test --tests com.ubb.events.implementation.MessageServiceTest :RestServices:test --tests com.ubb.events.controllers.MessageControllerApiTest` reached and ran tests; `MessageServiceTest` had 2 behavioral failures at lines 34 and 42. Persistence passed; API tests were not reached because Gradle stopped at the failing Business task.
- **Failure reason:** The service was still a stub returning an empty list and did not throw `NotFoundException` for an event outside the organization. Compilation and test setup succeeded, so the red result demonstrated missing planned behavior.

### Green (after implementation)

- **Result:** Focused command passed: `./gradlew.bat :Persistence:test --tests com.ubb.events.persistence.MessageRepositoryTest :Business:test --tests com.ubb.events.implementation.MessageServiceTest --tests com.ubb.events.implementation.MessageServiceStructureTest :RestServices:test --tests com.ubb.events.controllers.MessageControllerApiTest` (`BUILD SUCCESSFUL`). It covered existing Message persistence constraints plus repository filtering/order, service mapping and missing-event validation, member/manager access, empty list, unauthenticated 401, and non-member 403.
- **Full suite of the affected repo:** `./gradlew.bat test` was run. It currently fails in the pre-existing `OrganizationServiceStructureTest` (2 assertions expecting the old empty organization service scaffold, while `OrganizationService` already contains implemented operations); no T41 tests fail. This is unrelated to T41 and was not changed.

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

- **Decisions:** Added an event-scoped, ascending message-id query; validated event ownership through `EventRepository.findByIdAndOrganizationId`; returned `{id, senderId, content}`; used `@RequireOrgPermission` for member-level access and allowed all event states, including draft.
- **Alternatives considered:** Kept the route organization-scoped as specified instead of retaining the T40 scaffold root `/api/chat-messages`; kept event ownership validation in the service rather than duplicating persistence logic in the controller.
- **Limitations:** The full backend suite run reported two unrelated stale `OrganizationServiceStructureTest` assertions; the task branch was committed and merged through PR #43. Message ordering is by generated ID ascending.

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
