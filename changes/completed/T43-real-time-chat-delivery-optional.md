# T43 - Real-time chat delivery (optional)

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T43
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** ORGZ-05
- **Use case(s):** UC-14
- **Workflow:** workflow
- **Drive doc / ADR:** N/A
- **Repo(s):** backend
- **Branch:** `T43-real-time-chat-delivery-optional`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/53
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/MessageServicePublisherTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MessageControllerApiTest.java`; `RestServices/src/test/java/com/ubb/events/config/ChatMessagePublisherAdapterTest.java`, `WebSocketConfigStructureTest.java`, `WebSocketAuthorizationInterceptorTest.java`, and `WebSocketDeliveryIntegrationTest.java`.

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Real-time chat delivery (optional)” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORGZ-05. **Depends on:** View event messages, Send event message.
- WebSocket delivery on top of the REST chat. Only if time allows.

Dependencies from the backlog: 41, 42.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MessageController.java`, `Business/src/main/java/com/ubb/events/MessageService.java`, `Business/src/main/java/com/ubb/events/implementation/MessageServiceImpl.java`, `Persistence/src/main/java/com/ubb/events/persistence/MessageRepository.java`, and `Model/src/main/java/com/ubb/events/domain/Message.java`; these form the existing REST chat workflow.
- **Related completed tasks:** `changes/completed/T41-view-event-messages.md` and `T42-send-event-message.md` define the existing REST chat behavior, route, response contract, and organization authorization patterns; `T40-init-chatmessage-entity.md` provides the entity scaffold.
- **Constraints / assumptions:** Backend only; Java 21, Spring Boot, JPA, JUnit 5, Gradle wrapper, and layered architecture per `context/project-context.md`. Backlog calls WebSocket delivery optional (“Only if time allows”); the approved scope uses STOMP `/ws` delivery over event topics. The user confirmed backend `master` is up to date; the repo was clean before creating the task branch.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the optional delivery slice as a small STOMP-over-WebSocket adapter on top of the existing REST chat workflow. The REST POST remains the source of truth: after a message is successfully persisted, the application publishes the existing `MessageResponse` to an organization/event topic. Clients connect to one documented WebSocket endpoint and subscribe to that topic; no second message-creation protocol or persistence path is added. Existing REST behavior and organization authorization remain unchanged.

Because the backlog says “only if time allows” and does not define a protocol, this proposed contract needs confirmation at Gate 1: `/ws` for the handshake and `/topic/organizations/{organizationId}/events/{eventId}/messages` for subscriptions. JWT authentication and organization membership must be enforced for the WebSocket session/subscription using the same authorization rules as REST. If this contract is not approved, the task scope must be revised before implementation.

### Steps

1. **Tests first (REST/service and WebSocket):** add tests for the publisher contract, successful REST sends publishing exactly one response to the event topic, no publication when validation, authorization, or persistence fails, and an authenticated member/manager receiving a newly sent message over WebSocket. Add negative coverage for unauthenticated and non-member connection or subscription attempts. Confirm a behavioral red result before implementation.
2. **Messaging configuration (API adapter):** add the Spring WebSocket/messaging dependency and configure a STOMP endpoint at `/ws` with a simple broker under `/topic`. Keep CORS/origin handling consistent with the existing frontend origin.
3. **Authentication and authorization (API adapter):** carry the existing JWT identity into the WebSocket session during the handshake or STOMP CONNECT and check organization membership before allowing a subscription to an organization/event topic. Do not trust organization IDs supplied by the client without the existing authorization and event-ownership checks.
4. **Business integration:** introduce a narrow `ChatMessagePublisher` port in Business and invoke it only after `MessageService.sendMessage` has persisted and mapped the message. Implement the port in RestServices with `SimpMessagingTemplate`, publishing the existing `MessageResponse` to the event topic. Keep persistence and message validation in their existing layers.
5. **Verification:** run focused Business/RestServices and WebSocket integration tests to green, then the full backend test suite. Record exact commands and any unrelated baseline failures in the task test log.

### Files to create / modify

- `repos/ubb-events-backend/RestServices/build.gradle` (WebSocket/messaging dependency).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/...` (publisher port and send-flow integration).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/...` (STOMP configuration, handshake authentication, subscription authorization, publisher adapter).
- `repos/ubb-events-backend/Business/src/test/java/com/ubb/events/...` and `RestServices/src/test/java/com/ubb/events/...` (unit, API, and WebSocket integration tests).

### Tests to write

- Acceptance criterion 1: a WebSocket integration test connects as an authorized member, subscribes to the event topic, sends through the existing REST endpoint, and observes the persisted `MessageResponse`; a manager receives the same delivery.
- Acceptance criterion 2: tests prove no event is published on rejected REST sends, and unauthenticated/non-member connection or subscription attempts are rejected. Existing REST validation and authorization tests remain green.

### Risks / open questions

- The backlog does not specify protocol, endpoint, topic naming, client-to-server send behavior, or handshake authentication. This plan proposes STOMP `/ws`, the `/topic/organizations/{organizationId}/events/{eventId}/messages` subscription, server broadcast after REST POST, and JWT/membership checks. Confirm this scope before Gate 1.
- There is currently no WebSocket or messaging dependency/configuration in the backend. Adding one may require Spring Security handshake integration and test support; keep implementation limited to delivery of already-persisted REST messages.
- WebSocket integration tests may need an embedded broker/client and can be environment-sensitive. If that infrastructure is declined, replace the integration test with a publisher unit test and record the limitation explicitly.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/MessageServicePublisherTest.java`; `RestServices/src/test/java/com/ubb/events/config/WebSocketConfigStructureTest.java`.
- **Result:** `./gradlew.bat :Business:test --tests com.ubb.events.implementation.MessageServicePublisherTest :RestServices:test --tests com.ubb.events.config.WebSocketConfigStructureTest --rerun-tasks` ran two Business tests; `publishesOnlyAfterMessageIsSaved` failed with `WantedButNotInvoked` because the publisher was never called, and Gradle stopped before the RestServices task. Separately, `./gradlew.bat :RestServices:test --tests com.ubb.events.config.WebSocketConfigStructureTest --rerun-tasks` ran one test; `enablesStompBrokerAndDeclaresExpectedConfiguration` failed at `WebSocketConfigStructureTest.java:13` because the stub config lacked `@EnableWebSocketMessageBroker`.
- **Failure reason:** Both are behavioral failures for missing planned functionality; tests compiled and setup succeeded. A prior sandbox attempt blocked on Gradle cache permissions and was not treated as red evidence.

### Green (after implementation)

- **Result:** Focused command `./gradlew.bat :Business:test --tests com.ubb.events.implementation.MessageServicePublisherTest :RestServices:test --tests com.ubb.events.config.ChatMessagePublisherAdapterTest --tests com.ubb.events.config.WebSocketConfigStructureTest --tests com.ubb.events.config.WebSocketAuthorizationInterceptorTest --tests com.ubb.events.config.WebSocketDeliveryIntegrationTest --tests com.ubb.events.controllers.MessageControllerApiTest --rerun-tasks` passed (`BUILD SUCCESSFUL`). It covered publish-after-save, rejected-send behavior, publisher topic mapping, STOMP configuration, JWT/member subscription authorization, the REST route, and live delivery of the persisted REST `MessageResponse` to member and manager subscribers.
- **Delivery diagnosis:** The broker registered both authenticated sessions, matched the event topic, and wrote messages to `clientOutboundChannel`. The client test initially used a converter that could not decode the server's JSON `MessageResponse`; it reported `No suitable converter for payload type [B]`. The integration client now uses Jackson JSON conversion and verifies the delivered response fields. CONNECT authentication now updates Spring's accessor attached to the inbound message in place, which registers the authenticated STOMP sessions correctly.
- **Full suite of the affected repo:** `./gradlew.bat test --rerun-tasks` ran 154 tests and stopped at two pre-existing `OrganizationServiceStructureTest` failures (`implementationIsAnnotatedAndImplementsServiceWithoutOperations`, `serviceInterfaceIsEmpty`). The focused T43 tests listed above passed. The suite failures are unrelated stale scaffold assertions.

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

- **Decisions:** Added STOMP over WebSocket with `/ws` handshake and `/topic/organizations/{organizationId}/events/{eventId}/messages` subscriptions. REST message creation remains authoritative; the Business service publishes only after persistence through a `ChatMessagePublisher` port. WebSocket CONNECT validates the JWT, and subscriptions require member-level organization access plus event ownership validation. The topic carries the existing `MessageResponse` as JSON.
- **Alternatives considered:** Kept message creation on the existing REST endpoint instead of adding a second STOMP send command, avoiding duplicate validation and persistence paths. Used Spring’s simple in-memory broker because the task only requires delivery and does not define a durable broker deployment.
- **Limitations:** WebSocket delivery is process-local and in-memory; messages are not queued for disconnected clients. The full backend suite still reports two unrelated stale `OrganizationServiceStructureTest` failures.

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
