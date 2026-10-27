# T32 - Register for event

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T32
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** REG-01, REG-02, REG-03, REG-06, LIF-02, LIF-03
- **Use case(s):** UC-11
- **Workflow:** workflow
- **Drive doc / ADR:** [Project requirements, use cases, architecture, ERD, and API document](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban) — REG-01, REG-02, REG-03, REG-06, LIF-02, LIF-03; UC-11.
- **Repo(s):** backend
- **Branch:** `T32-register-for-event`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/34
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceTest.java`, `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceStructureTest.java`, `Model/src/test/java/com/ubb/events/domain/RegistrationTest.java`, `RestServices/src/test/java/com/ubb/events/controllers/RegistrationControllerStructureTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Register for event” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** REG-01, REG-02, REG-03, REG-06, LIF-02, LIF-03. **Depends on:** Sign in, Init Registration entity, Publish event.
- `POST /api/events/{eventId}/registrations`, no body: check that the event is published and not completed or canceled, registration is open, the user is eligible and capacity remains (also under concurrent requests). Returns 201 with the registration details.
- One registration row per user per event: a duplicate shows the current status (409), and a user who cancelled is registered again by reactivating the row.
- Full event or closed registration returns 409 with the reason.

Dependencies from the backlog: 8, 21, 28.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above, using the user's clarification that cancellation deletes the registration row.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Registration.java`; `.../Event.java`; `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java`; `.../EventRepository.java`; `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/RegistrationService.java` and `implementation/RegistrationServiceImpl.java`; `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/RegistrationController.java` and its structure test.
- **Related completed tasks:** T8 (sign in), T21 (init registration entity), T28 (publish event). T9 covers organization scoped authorization, though this endpoint is user scoped.
- **Constraints / assumptions:** Backend only. The existing `RegistrationController` was a scaffold at `/api/registrations`; the implemented endpoint is `/api/events/{eventId}/registrations`. Backend was clean on `master` and up to date with `origin/master` before branch creation. `context/project-context.md` specifies JUnit 5, Gradle wrapper, layered architecture, and shared conflict/error response handling. The canonical Drive requirements/use-cases document is linked in Metadata.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the registration vertical path across persistence, business service and REST API. The service will use the authenticated user, load a published event, derive completion and registration-open state from event date/time and active registration count, and atomically reserve capacity before saving one registration row. Cancellation deletes the row, so a missing row means the user is not currently registered. Conflicts use the shared error mapping; the controller only maps HTTP and documents the endpoint.

### Steps

1. **Model / persistence:** add a registration response DTO if needed, extend `RegistrationRepository` with event/user lookup and a concurrency-safe capacity strategy, and enforce one row per event/user with a database uniqueness constraint where supported.
2. **Business service:** define and implement `registerForEvent(eventId, userId)`, checking event existence, published state, completion (now later than date + end time), future start, capacity, duplicate registration, and persistence. Make check-and-insert transactional and protected by locking or an equivalent database constraint so concurrent requests cannot exceed capacity; map duplicate races to 409.
3. **REST API:** map `POST /api/events/{eventId}/registrations` with no body, obtain the authenticated user id from the established security context, return 201 with registration details, and document 201/401/404/409 responses.
4. **Verification:** write focused tests first, then run affected modules and the full backend suite with the Gradle wrapper.

### Files to create / modify

- `Model/.../domain/Registration.java` and/or a new registration response DTO.
- `Persistence/.../RegistrationRepository.java` and persistence tests/schema support.
- `Business/.../RegistrationService.java` and `implementation/RegistrationServiceImpl.java`.
- `RestServices/.../RegistrationController.java` and matching controller tests.

### Tests to write

- Service tests for success, unknown event, draft/canceled/completed event, past start, full/closed event with reason, duplicate registration, and response mapping.
- Persistence/integration tests for one row per event/user, active count, and concurrent capacity protection.
- Controller API tests for exact POST path/no body, 201 payload, 401 missing authentication, 404 unknown event, and 409 duplicate/lifecycle/full cases.

### Risks / open questions

- User clarified: anyone may register; registration is open while the registered count is below capacity and the event start is in the future; a user with no registration row has not registered or has cancelled.
- User clarified completion: compare the current date and time with the event's date and end time; it is completed when now is later, including when the dates match but the current time is later than the event end time.
- User clarified cancellation: delete the registration row. A row means the user is registered; no row means they have not registered or have cancelled. This supersedes the backlog sentence about reactivating a cancelled row and means registration status is not needed for this workflow.
- Registration-open interpretation for the plan: compare the event's combined date and start time with the current date and time, in addition to checking registered count < capacity.
- Capacity correctness depends on database locking/constraint behavior and must be verified against the real persistence setup. The exact authenticated-principal accessor and schema migration mechanism should follow existing repository conventions.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceTest.java` and updated `RegistrationServiceStructureTest.java` were added before the attempted run.
- **Result:** No valid behavioral red result was observed. The initial command `.\gradlew.bat --no-daemon --console=plain --stacktrace :Business:test --tests "com.ubb.events.implementation.RegistrationServiceTest"` stalled without output and was interrupted. During resume, the focused command `.\gradlew.bat --no-daemon --console=plain :Business:test --tests "com.ubb.events.implementation.RegistrationServiceStructureTest" --tests "com.ubb.events.implementation.RegistrationServiceTest"` first failed at test compilation because the structure test did not handle `NoSuchMethodException`. After declaring that exception, the focused run succeeded. The user also reports running the new tests successfully and manually registering a user for an event; no command or test output was provided. These are post-implementation checks, not a valid pre-implementation red result.
- **Failure reason:** The initial execution environment stalled on Java/Gradle. The later compile failure was test setup only, not behavioral evidence; it was fixed by declaring `NoSuchMethodException`. The registration behavior had already been partially written before a valid red run, which is a deviation from the required TDD order. The red status item remains unchecked; implementation green is documented separately as post-implementation verification.

### Green (after implementation)

- **Result:** The focused four-class command `.\gradlew.bat --no-daemon --console=plain :Model:test --tests "com.ubb.events.domain.RegistrationTest" :RestServices:test --tests "com.ubb.events.controllers.RegistrationControllerStructureTest" :Business:test --tests "com.ubb.events.implementation.RegistrationServiceStructureTest" --tests "com.ubb.events.implementation.RegistrationServiceTest"` passed after updating the stale registration mapping assertions. This is post-implementation verification; the valid red sequence is still absent, and automated controller API and concurrent persistence tests from the plan remain incomplete. The implementation-green item is checked based on these focused tests and the user's manual verification.
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` ran all modules. Business: 94 tests, 2 failures (`OrganizationServiceStructureTest.implementationIsAnnotatedAndImplementsServiceWithoutOperations`, `OrganizationServiceStructureTest.serviceInterfaceIsEmpty`). Model: 13 tests passed. Persistence completed successfully. RestServices: 128 tests, 1 failure (`OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`). These three failures match the pre-existing failures recorded by T28. Overall build failed with those 3 known failures.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The backlog says a cancelled registration row is reactivated. | Per the user's clarification, cancellation deletes the row; a row means registered, and no row means not registered or cancelled. The plan will omit cancelled-row reactivation/status. | The user confirmed the existing intended row-deletion logic. |
| 2 | implementation | The user said there are errors in `RegistrationServiceStructureTest`. | Declared `NoSuchMethodException` on the reflection-based test method and renamed it to describe the registration operation it checks. | The user identified the test errors; the compiler reported two unhandled checked exceptions from reflection calls. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

<!-- DRAFT: verify before using as a study note. -->
- **Decisions:** The service locks the event row while checking capacity and saving, and the registration table has a unique `(event_id, user_id)` constraint to protect one-row-per-user/event semantics. The service derives completion and opening from the event date plus end/start time. Cancellation remains row deletion, per the user's clarification.
- **Alternatives considered:** Relying only on an application-level count without a lock or unique constraint was avoided because concurrent requests could overbook the event or create duplicate rows.
- **Limitations:** No behavioral red run was observed before implementation, and the task log records this TDD deviation. The approved plan's controller API and concurrent persistence tests remain incomplete. The full backend suite has three known unrelated failures from T28.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
