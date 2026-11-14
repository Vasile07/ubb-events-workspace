# T42 - Send event message

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T42
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** ORGZ-05, ORG-05
- **Use case(s):** UC-14
- **Workflow:** workflow
- **Drive doc / ADR:** N/A for now
- **Repo(s):** backend
- **Branch:** `T42-send-event-message`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/44
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/MessageServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MessageControllerApiTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MessageControllerStructureTest.java`; `Persistence/src/test/java/com/ubb/events/persistence/MessageRepositoryTest.java`.

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Send event message” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORGZ-05, ORG-05. **Depends on:** Organization-scoped authorization, Init ChatMessage entity.
- `POST` on the same path: the author comes from the token, the text cannot be empty and has a maximum length. Returns 201 with the message, author and time.
- Refused for a canceled event (read-only).

Dependencies from the backlog: 9, 40.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Message.java` (currently has event, sender, and content, without a timestamp); `Persistence/src/main/java/com/ubb/events/persistence/MessageRepository.java`; `Business/src/main/java/com/ubb/events/MessageService.java`; `Business/src/main/java/com/ubb/events/implementation/MessageServiceImpl.java`; `RestServices/src/main/java/com/ubb/events/controllers/MessageController.java`; `RestServices/src/main/java/com/ubb/events/security/OrganizationAuthorizationInterceptor.java`; event workflow patterns in `EventServiceImpl`, `EventRepository`, `EventController`, and their corresponding API tests.
- **Related completed tasks:** `changes/completed/T40-init-chatmessage-entity.md` establishes the Message persistence scaffold; `changes/completed/T41-view-event-messages.md` establishes the event-scoped message endpoint and authorization patterns; `changes/completed/T9-organization-scoped-authorization.md` documents organization membership authorization.
- **Constraints / assumptions:** Backend only; Java 21, Spring Boot, JPA, JUnit 5, Gradle wrapper, and layered architecture per `context/project-context.md`. The current `Message` model has event, sender, and content fields; this task needs to add timestamp persistence. T40/T41 documentation uses the Java/domain name `Message`. The plan proposes reusing the T41 event message route, a 1000-character content limit, request `{content}`, response `{id, senderId, content, sentAt}`, and ISO-8601 date-time with offset. The backend repo was clean on `T41-view-event-messages`; it was switched to `master`, fetched and fast-forwarded to the latest `origin/master`, then `T42-send-event-message` was created from that clean updated master.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the send-message workflow across the backend persistence, business, and REST layers. Reuse the T41 organization/event route and `@RequireOrgPermission` authorization. The service will obtain the authenticated author from the security context, validate the event belongs to the organization and is not canceled, validate non-blank content with a maximum length of 1000 characters, create the message with the current timestamp, and return a response containing the persisted message, author, content, and timestamp. Use an ISO-8601 date-time representation for the timestamp. A canceled event will produce the existing conflict-style 409 response with a client-safe message such as `A canceled event cannot receive messages`.

### Steps

1. **Tests first:** add service and API tests for successful member/manager sends, author identity from the authenticated token, timestamp serialization, blank and over-1000-character content, missing or cross-organization events, unauthenticated/non-member access, and canceled-event rejection. Confirm a behavioral red result before implementation.
2. **Model/persistence:** extend the existing `Message` entity as needed to persist the send timestamp, and add the repository operations required to save a message and verify event-scoped behavior without moving business rules into persistence.
3. **Business:** add a send operation to `MessageService` and implement it in `MessageServiceImpl`. Resolve the authenticated user, validate event ownership and status, reject canceled events with `ConflictException`, create and save the message, and map it to the response DTO.
4. **API DTO/controller:** use the proposed request shape `{content}` and response shape `{id, senderId, content, sentAt}`, where `sentAt` is an ISO-8601 date-time with offset. Add `POST /api/organizations/{organizationId}/events/{eventId}/messages` to `MessageController`, annotate it with member-level `@RequireOrgPermission`, return `201 Created`, and document 201, 400, 401, 403, 404, and 409 responses.
5. **Verification:** run the focused tests to green, then the full backend test suite, and record exact commands and results in the task Test log.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Message.java` (add the persisted timestamp if the current entity does not yet provide one).
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/SendMessageRequest.java` and `MessageResponse.java` (create request/response contracts).
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MessageRepository.java` (add only required data-access operations).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MessageService.java` and `implementation/MessageServiceImpl.java` (send operation and validation).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MessageController.java` (authorized POST endpoint and OpenAPI responses).
- Corresponding unit/integration/API tests under the existing Model, Persistence, Business, and RestServices test source sets.

### Tests to write

- Acceptance criterion 1: API happy-path tests prove a member and manager can send a message and receive `201` with persisted message, author, content, and ISO-8601 timestamp; a service test proves the author is taken from the authenticated user rather than the request body.
- Acceptance criterion 2: tests cover blank content, content longer than 1000 characters, invalid or cross-organization event (`404`), unauthenticated (`401`), non-member (`403`), canceled event (`409` with the client-safe conflict message), and timestamp persistence/serialization.

### Risks / open questions

- The route and authorization contract follow T41. Proposed Gate 1 contract: request `{content}`; response `{id, senderId, content, sentAt}`. `senderId` preserves T41's existing response naming, and `sentAt` is an ISO-8601 date-time with offset. No additional user-facing fields are required.
- The current Message entity has no timestamp field, so adding one is required to satisfy the backlog's response contract. The exact database column name should follow the existing entity naming convention during implementation.
- Proposed canceled-event behavior is `409 Conflict` with `A canceled event cannot receive messages`, matching existing canceled-event conflict handling.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/MessageServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MessageControllerApiTest.java`.
- **Result:** `./gradlew.bat :Business:test --tests com.ubb.events.implementation.MessageServiceTest :RestServices:test --tests com.ubb.events.controllers.MessageControllerApiTest --rerun-tasks` compiled the tests and ran five service tests; three failed: `rejectsCanceledEvent`, `rejectsBlankAndOverlongContent`, and `sendsMessageUsingAuthenticatedAuthorAndCurrentTime`.
- **Failure reason:** The send operation was a compile stub returning `null`, so it neither validated content nor rejected canceled events and caused the success test to dereference `null`. Compilation and test setup succeeded; the failures demonstrate missing planned behavior.

### Green (after implementation)

- **Result:** Focused command `./gradlew.bat :Persistence:test --tests com.ubb.events.persistence.MessageRepositoryTest :Business:test --tests com.ubb.events.implementation.MessageServiceTest :RestServices:test --tests com.ubb.events.controllers.MessageControllerApiTest --tests com.ubb.events.controllers.MessageControllerStructureTest --rerun-tasks` passed on rerun (`BUILD SUCCESSFUL`, 18 actionable tasks). It covered timestamp persistence and save/reload, GET response timestamp mapping, member and manager sends with author from the authenticated principal, 1000-character validation, canceled-event conflict, API 201 response and ISO-8601 timestamp, validation 400, authorization, and controller structure. The manager send and timestamp mapping assertions were added after the initial implementation as supplemental coverage; they are not pre-implementation red evidence.
- **Full suite of the affected repo:** `./gradlew.bat test --rerun-tasks` stopped at `:Business:test` after 123 tests; two pre-existing `OrganizationServiceStructureTest` assertions failed (`serviceInterfaceIsEmpty` and `implementationIsAnnotatedAndImplementsServiceWithoutOperations`), because they expect the already-implemented Organization service to remain an empty scaffold. Gradle did not complete downstream module test tasks in this run. No T42 test failed in the focused run. A live database schema check was unavailable because no literal PostgreSQL endpoint is configured in this environment; tests use the test profile with generated schema.

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

- **Decisions:** Added an organization-scoped POST beside T41's GET route, with member-level organization authorization. The request is `{content}` with a 1000-character maximum; the author is taken from the authenticated JWT principal and resolved through `UserRepository`; the response is `{id, senderId, content, sentAt}` with an ISO-8601 offset date-time. Canceled events return the existing conflict mapping as HTTP 409. Message timestamps are persisted in `sent_at` and defaulted on persistence for legacy constructor use.
- **Alternatives considered:** Passed the authenticated user id from the REST controller into the business service rather than coupling the Business module to Spring Security. Reused the shared validation and global error handler instead of adding controller-specific error mapping.
- **Limitations:** The team added the non-null `sent_at` column manually in the database; this repository contains no migration file for it. The full backend test command currently stops on two unrelated stale Organization service structure assertions.

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
