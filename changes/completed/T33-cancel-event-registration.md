# T33 - Cancel event registration

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T33
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** REG-04
- **Use case(s):** UC-18
- **Workflow:** workflow
- **Drive doc / ADR:** [Project requirements, use cases, architecture, ERD, and API document](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban) — REG-04 and UC-18.
- **Repo(s):** backend
- **Branch:** `T33-cancel-event-registration`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/35
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceTest.java`; `Persistence/src/test/java/com/ubb/events/persistence/RegistrationRepositoryTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/RegistrationControllerStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/RegistrationControllerApiTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Cancel event registration” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** REG-04. **Depends on:** Register for event.
- `DELETE /api/events/{eventId}/registrations/me`: cancel the user's own active registration until the event starts, freeing a place. Returns 204.
- Respond clearly when there is no registration or the event has started.

Dependencies from the backlog: 32.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/RegistrationService.java` and `implementation/RegistrationServiceImpl.java`; `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java` and `EventRepository.java`; `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/RegistrationController.java`; corresponding registration service/controller tests; `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Registration.java` and `Event.java`.
- **Related completed tasks:** T32 Register for event (includes the user's clarification that cancellation deletes the registration row).
- **Constraints / assumptions:** Backend only. The registration endpoint currently authenticates via `Authentication` principal and calls `RegistrationService`; `EventRepository` already supports a pessimistic event lock; registrations are identified by event and user, and the unique constraint is on `(event_id, user_id)`. The backend is clean on `master`, with no unpushed commits, and was fetched; `master` is already up to date with `origin/master`. `context/project-context.md` specifies JUnit 5, Gradle wrapper, layered architecture and shared error handling. User clarified: no active registration returns 404; event already started returns 409.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement cancellation as the inverse of T32's row-based registration model. The authenticated user's event registration will be located through the existing repository, and the service will delete that row in one transaction after locking/loading the event and confirming its start time has not passed. The REST controller will expose `DELETE /api/events/{eventId}/registrations/me`, return 204 with no body on success, and leave not-found/conflict mapping to the shared exception handler. A separate cancelled status or reactivation flow is not needed because T32 established that cancellation deletes the row.

### Steps

1. **Persistence:** add the repository query needed to locate the registration by event and user (reusing the existing event/user lookup where possible) and ensure deletion is performed through the JPA repository within the service transaction.
2. **Business/service:** add `cancelRegistration(eventId, userId)` to the service contract and implementation. Load the event (404 when it does not exist), acquire the existing event lock, compare the current date/time with the event start, and raise a conflict when the event has started. Locate the authenticated user's active row, raise `NotFoundException` when none exists, and delete it so capacity is freed. Keep business rules out of the controller.
3. **REST API:** add the `DELETE /{eventId}/registrations/me` mapping beside the existing registration endpoint, extract the authenticated principal using the established security convention, invoke the service, and return `ResponseEntity.noContent()`. Document 204, 401, 404 and 409 responses with clear operation/error descriptions.
4. **Verification:** write tests before implementation, obtain a behavioral red result, implement the vertical path, run focused module tests, then run the full backend suite and record exact commands/results in the task log.

### Files to create / modify

- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/RegistrationService.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/RegistrationServiceImpl.java`
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java`
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/RegistrationController.java`
- Existing registration service/controller tests and, if needed, a persistence integration test under the corresponding `src/test/java` package.

### Tests to write

- **Acceptance criterion 1 / service behavior:** unit tests for successful cancellation (the exact event/user row is deleted), unknown event (404), no active registration (404), cancellation after the event start (409), and boundary behavior at the start instant. Verify that failed cases do not delete a row.
- **Acceptance criterion 1 / API behavior:** an API/controller test for the exact `DELETE /api/events/{eventId}/registrations/me` mapping, authenticated user propagation, 204 response with an empty body, and shared 404/409 error mapping; include unauthenticated access returning 401 through the real security setup where the existing test harness supports it.
- **Acceptance criterion 2 / persistence and authorization:** repository/integration coverage that deletion removes only the authenticated user's registration for the requested event and that another user's row remains. Include the existing unique event/user relation and transaction behavior as applicable to the repository test setup.

### Risks / open questions

- The existing registration service currently locks the event row through `EventRepository`; cancellation should use the same lock to serialize the start-time check with concurrent registration requests.
- The exact test fixture for authenticated principals and global error handling should follow the existing RestServices API tests; no new security mechanism is planned.
- The requirement specifies behavior after the event starts but does not define a special rule for a draft/canceled event before its start. The plan therefore limits the new business check to event existence, start time, and the user's active row; any additional event lifecycle rule requires user clarification.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** Added `cancelsOwnActiveRegistration`, `rejectsCancellationWhenNoActiveRegistration`, and `rejectsCancellationAfterEventStart` to `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceTest.java` before implementing cancellation; added the controller endpoint test skeleton and minimum interface/stub needed to compile.
- **Result:** `.\gradlew.bat --no-daemon --console=plain --stacktrace :Business:test --tests "com.ubb.events.implementation.RegistrationServiceTest" --rerun-tasks` failed behaviorally: 8 tests completed, 3 failed. The three new tests failed because the cancellation stub threw `UnsupportedOperationException`.
- **Failure reason:** The tests compiled and exercised the missing cancellation behavior; this was a valid behavioral red result, not a compilation or setup failure.

### Green (after implementation)

- **Result:** The initial focused command `.\gradlew.bat --no-daemon --console=plain :Business:test --tests "com.ubb.events.implementation.RegistrationServiceTest" :RestServices:test --tests "com.ubb.events.controllers.RegistrationControllerStructureTest" --rerun-tasks` passed after implementation. A later focused command added `RegistrationControllerApiTest` and the unknown-event service test and passed with real Spring Security/MockMvc API coverage. After that, the repository test `deletingOneUsersRegistrationLeavesAnotherUsersRegistration` was added and this focused command passed: `.\gradlew.bat --no-daemon --console=plain :Persistence:test --tests "com.ubb.events.persistence.RegistrationRepositoryTest" :Business:test --tests "com.ubb.events.implementation.RegistrationServiceTest" :RestServices:test --tests "com.ubb.events.controllers.RegistrationControllerStructureTest" --tests "com.ubb.events.controllers.RegistrationControllerApiTest" --rerun-tasks`.
- **Full suite of the affected repo:** The final `.\gradlew.bat --no-daemon --console=plain clean test --continue` run after all test additions executed all modules. Model and Persistence passed. Business ran 98 tests with 2 pre-existing failures in `OrganizationServiceStructureTest`; RestServices ran 134 tests with 1 pre-existing failure in `OrganizationAuthorizationInterceptorTest`. These failures are unrelated to T33 and match known baseline failures.
- **User verification:** The user reports that they reran the tests and manually tested the endpoint successfully; no command output was supplied for that run.

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

- **Decisions:** Cancellation reuses the existing event pessimistic lock and event/user registration lookup, checks the event start instant in the service transaction, then deletes the registration row. Missing event or active row maps to 404; a start time reached maps to 409; successful deletion returns 204. The controller extracts the authenticated principal and contains no business rules.
- **Alternatives considered:** A cancellation status or row reactivation was not added because T32 established row deletion as the registration model. Application-level checks without the existing event lock were not used because registration and cancellation lifecycle checks should be serialized.
- **Limitations:** The full backend suite retains three unrelated baseline failures: two Business organization structure assertions and one RestServices authorization interceptor assertion. Exact start-time equality is not tested; the service coverage uses the deterministic post-start rule, while testing equality precisely would require an injectable clock and expand the approved scope. Repository deletion isolation is covered with the existing H2 `@DataJpaTest` fixture.

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
