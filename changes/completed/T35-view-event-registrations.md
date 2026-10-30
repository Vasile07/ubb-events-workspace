# T35 - View event registrations

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T35
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** REG-05, ORG-05
- **Use case(s):** UC-12
- **Workflow:** workflow
- **Drive doc / ADR:** [Requirements, use cases, architecture, and API specification](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban)
- **Repo(s):** backend
- **Branch:** `T35-view-event-registrations`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/37
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceTest.java`; `Persistence/src/test/java/com/ubb/events/persistence/RegistrationRepositoryTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/EventControllerStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/EventControllerApiTest.java`.

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View event registrations” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** REG-05, ORG-05. **Depends on:** Organization-scoped authorization, Register for event.
- `GET /api/organizations/{organizationId}/events/{eventId}/registrations`: the registered users of an event, for members and managers of the organizing organization (403 for others). Verify that the event belongs to the organization.
- Cancelled registrations are not shown as registered. Empty list case. Visible fields follow the privacy policy. No export.

Dependencies from the backlog: 9, 32.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Registration.java`, `Event.java`, and `dto/EventRegistrationResponse.java`; `Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java`; `Business/src/main/java/com/ubb/events/RegistrationService.java` and `implementation/RegistrationServiceImpl.java`; `RestServices/src/main/java/com/ubb/events/controllers/EventController.java`; organization authorization annotations/interceptor in `RestServices/src/main/java/com/ubb/events/security/`.
- **Related completed tasks:** `changes/completed/T9-organization-scoped-authorization.md` establishes `@RequireOrgPermission` and membership authorization; `changes/completed/T32-register-for-event.md` establishes that registrations are persisted as rows and cancellation deletes the row.
- **Constraints / assumptions:** Backend only; Java 21, Spring Boot, layered architecture, JUnit 5 and Gradle wrapper per `context/project-context.md`. Backend is clean, on `master`, and up to date with `origin/master`. Only existing registration rows should count as registered, consistent with T32 cancellation behavior. User clarification: expose each registrant's name and email, consistent with the privacy policy.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add the organization-scoped registration-list workflow through the existing backend layers. The service will first load the event by both event ID and organization ID so a valid event from another organization is treated as not found, then query the registration rows with their users and map only first name, last name, and email into a response DTO. The controller will expose the specified GET endpoint, require the existing organization MEMBER permission, and rely on the shared exception handler for 401/403/404 responses. Because T32 cancels by deleting the row, the query naturally excludes cancelled registrations and returns an empty JSON list when none remain.

The list will be implemented as a dedicated service operation and repository query rather than reusing the participant's “my registrations” projection. This keeps organization/event ownership validation and privacy filtering in the business/API contract and avoids adding export or pagination behavior that is outside the task.

### Steps

1. **Persistence:** add a repository query that joins registrations to users and selects registrations for an event whose event organization matches the requested organization; preserve deterministic ordering if the existing conventions support it. Ensure the query returns no rows for cancelled registrations (deleted rows) and does not expose password or other user fields.
2. **Model / DTO:** add a response type containing only the registrant’s first name, last name, and email (the user-approved privacy fields), and any projection needed to construct it without exposing domain entities from the API.
3. **Business:** extend `RegistrationService` with an organization/event list operation. Load/verify the event using `findByIdAndOrganizationId` (or an equivalent repository method) before querying registrations; raise the shared not-found error when either identifier does not describe the same event. Map each result to the privacy-limited DTO and return an empty list when there are no active rows.
4. **API:** add `GET /api/organizations/{organizationId}/events/{eventId}/registrations` to the appropriate controller, annotate it with `@RequireOrgPermission` at the MEMBER minimum, document the operation and 200/401/403/404 responses, and keep authorization in the existing interceptor. Do not add export, pagination, or a second authorization mechanism.
5. **Verification:** write tests first and obtain a behavioral red result, then implement and run focused module tests followed by the full backend Gradle suite. Record exact commands, failures and results in this task file.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/` — create the privacy-limited event-registration response DTO (or reuse an existing DTO only if it contains exactly the approved fields).
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java` — add the event/organization registration projection/query.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/RegistrationService.java` and `implementation/RegistrationServiceImpl.java` — add the organization-scoped list operation and ownership validation.
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/EventController.java` — add the GET handler to the existing organization event controller, with OpenAPI annotations and `@RequireOrgPermission`.
- Corresponding tests under `Persistence/src/test/java`, `Business/src/test/java`, and `RestServices/src/test/java` in the backend repository.

### Tests to write

- **Acceptance criterion 1 / service:** a service test returns the name/email projection for registrations of the requested event and organization; a registration belonging to another event or organization is not returned; a cancelled registration represented by a deleted row is absent; and no registrations produces an empty list.
- **Acceptance criterion 1 / API:** an API-level test calls the exact GET path and verifies HTTP 200 plus a JSON list containing only first name, last name, and email (no password, membership role, or unrelated fields). Verify OpenAPI metadata includes the endpoint and response codes.
- **Acceptance criterion 2 / authorization and errors:** integration/controller tests verify an authenticated MEMBER and MANAGER of the organizing organization can read the list, a user from another organization receives 403, an unknown organization receives the interceptor’s shared 403 response, an unknown event or an event belonging to another organization receives the shared 404 response, and unauthenticated access receives 401 through the existing security setup.
- **Persistence:** repository/integration coverage verifies the organization/event filter, user join, stable mapping fields, and empty-result behavior against the real persistence configuration.
- Follow the required TDD order: tests and compile stubs first, a behavioral red result caused by the missing list behavior, implementation, focused green tests, then the wider backend suite.

### Risks / open questions

- The repository currently exposes entities and a “my registrations” DTO; the final response DTO/projection shape should be kept limited to first name, last name, and email as clarified by the user. If the API specification in Drive defines a different display-name format, update the plan before Gate 1.
- No unresolved questions remain; the privacy fields were clarified with the user before Gate 1.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceTest.java` — `listsRegistrantNameAndEmailForOwnedEvent`.
- **Result:** `.\gradlew.bat --no-daemon --console=plain :Business:test --tests "com.ubb.events.implementation.RegistrationServiceTest.listsRegistrantNameAndEmailForOwnedEvent"` compiled the test and reported `1 test completed, 1 failed`.
- **Failure reason:** The test failed with `UnsupportedOperationException` from the deliberate `RegistrationServiceImpl.listEventRegistrations` compile stub. This is a behavioral failure demonstrating the missing workflow; the earlier compile-only stub mismatch was corrected before this run.

### Green (after implementation)

- **Result:** After the DTO and constructor corrections, focused service command `.\gradlew.bat --no-daemon --console=plain :Business:test --tests "com.ubb.events.implementation.RegistrationServiceTest"` passed. Focused persistence command `.\gradlew.bat --no-daemon --console=plain :Persistence:test --tests "com.ubb.events.persistence.RegistrationRepositoryTest"` passed, including event/organization filtering, joined user fields, and empty-list coverage. Focused API and structure command `.\gradlew.bat --no-daemon --console=plain :RestServices:test --tests "com.ubb.events.controllers.EventControllerStructureTest" --tests "com.ubb.events.controllers.EventControllerApiTest"` passed.
- **Full suite of the affected repo:** Final run after the DTO and constructor corrections, `.\gradlew.bat --no-daemon --console=plain clean test --continue`, completed with three pre-existing failed test cases across two classes: `OrganizationServiceStructureTest.implementationIsAnnotatedAndImplementsServiceWithoutOperations`, `OrganizationServiceStructureTest.serviceInterfaceIsEmpty`, and `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization` (the latter reports the existing `OrganizationController.list` scaffold). T35 tests compiled and passed; Model and Persistence suites passed, and RestServices reported 143 tests with only that pre-existing failure.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | implementation | The EventRegistrationResponse dto is a record, but the rules specify that for this project, dto must be a class, please refactor | Converted EventRegistrationResponse from a record to a Lombok-backed class with getters, an all-arguments constructor, and value equality. | To follow the project DTO convention. |
| 2 | implementation | In EventController there is one constructor with only one argument that I asked about previously. Can you remove it without affecting the app? | Removed the unused one-argument constructor and retained the `@Autowired` two-argument constructor for `EventService` and `RegistrationService`; updated the structure test accordingly. | To keep one explicit production constructor with all required dependencies. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use `EventRepository.findByIdAndOrganizationId` before querying registrations so an event from another organization is returned as 404. Return a dedicated DTO containing only first name, last name, and email. Require organization MEMBER permission; MANAGER inherits access.
- **Alternatives considered:** Reusing the participant “my registrations” projection was rejected because it does not validate organization ownership and has a different response shape. Export and pagination were excluded because the task explicitly says no export and the project conventions use unpaginated lists.
- **Limitations:** Full-suite verification remains red on three failures already present in the baseline authorization/organization scaffold. The task metadata still lacks a Drive/ADR link.

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









