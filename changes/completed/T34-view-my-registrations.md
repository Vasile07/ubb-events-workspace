# T34 - View my registrations

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T34
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** REG-04
- **Use case(s):** UC-18
- **Workflow:** workflow
- **Drive doc / ADR:** [Requirements, use cases, architecture, ERD, and API specification](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban) — REG-04 and UC-18.
- **Repo(s):** backend
- **Branch:** `T34-view-my-registrations`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/36
- **Tests:** `Persistence/src/test/java/com/ubb/events/persistence/RegistrationRepositoryTest.java`; `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/UserControllerApiTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/UserRegistrationControllerTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View my registrations” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** REG-04. **Depends on:** Sign in, Register for event.
- `GET /api/users/me/registrations`: only the current user's own registrations (active and cancelled) with the public details of each event. Never other participants' data.

Dependencies from the backlog: 8, 32.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The endpoint returns only the authenticated user's currently persisted (active) registrations with public event details, and never returns other participants' data.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java` (authenticated /me pattern); `RegistrationController.java`; `Business/src/main/java/com/ubb/events/RegistrationService.java` and `implementation/RegistrationServiceImpl.java`; `Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java`; `Model/src/main/java/com/ubb/events/domain/Registration.java` (currently event/user links only, no status). 
- **Related completed tasks:** `changes/completed/T8-sign-in.md` (JWT filter sets authenticated user id); `changes/completed/T32-register-for-event.md` (registration creation and cancellation). 
- **Constraints / assumptions:** Backend only. `context/project-context.md` specifies JWT authentication, layered services/repositories/controllers, JUnit 5, Gradle wrapper, and an API-level test. Backend repo is clean on `master` and up to date with `origin/master`. Per user clarification, T34 returns currently active registrations only. T32 deletes registration rows on cancellation, and the current entity stores no cancellation status or history; cancelled registrations therefore cannot be listed. 

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the user-scoped registrations endpoint as a backend vertical slice. The persistence query will select registrations by the authenticated user id only; the business service will map each row to a response containing the registration identifier and public event details, without exposing participant or other-user data. In this task, an active registration is a persisted registration row. T32 deletes rows when a user cancels, so cancellation history is intentionally not reconstructed. Keep the endpoint read-only and return an empty list when the user has no active registrations.

Use the existing JWT principal extraction and `/api/users/me` controller conventions. Keep database access in `RegistrationRepository`, mapping and business decisions in `RegistrationService`, and HTTP documentation/status handling in the controller. Reuse compatible public event DTO fields where possible; introduce a focused registration-list DTO if an existing event DTO would expose fields outside this endpoint's contract.

### Steps

1. **Persistence:** add a repository method that returns registrations for one user, with the associated event data needed for the response and deterministic ordering. Ensure the query is constrained by `user_id` so another participant's rows cannot be returned.
2. **Model/business:** add the response DTO for a registration plus public event details, extend `RegistrationService` with a list operation, and implement mapping from registration/event entities. Preserve the active-row-only decision and return an empty list for no matches.
3. **API:** add `GET /api/users/me/registrations`, extract the authenticated user id from the JWT principal, delegate to the service, return `200 OK`, and document the success and authentication-required responses. Do not accept a user id or expose user identity in the request/response contract.
4. **Verification:** write tests first, obtain a behavioral red result, implement the slice, rerun focused tests, then run the affected backend suite with the Gradle wrapper and record the exact evidence in the task log.

### Files to create / modify

- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java` — user-scoped registration query and ordering.
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/...` — registration-list response DTO containing only the registration id and public event details, if existing DTOs cannot express the contract safely.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/RegistrationService.java` and `implementation/RegistrationServiceImpl.java` — list operation and entity-to-DTO mapping.
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java` (or a narrowly scoped `/api/users` controller if constructor/test conventions make that cleaner) — authenticated endpoint and OpenAPI responses.
- Corresponding tests under `Persistence/src/test/java`, `Business/src/test/java`, and `RestServices/src/test/java`.

### Tests to write

- **Acceptance criterion 1:** repository/service tests verify that the authenticated user's active registration rows are returned with the expected registration and public event fields, multiple registrations have stable ordering, and an empty result is returned when there are none.
- **Acceptance criterion 2:** API-level tests verify `GET /api/users/me/registrations` returns `200` for a valid JWT, returns only rows belonging to that principal even when another user's registration exists, does not include participant-sensitive fields, and returns `401` for missing or invalid authentication. Add a service test for the mapping and repository interaction.
- Follow the required TDD order: tests compile and fail for the missing list behavior before implementation, then focused tests pass and the wider backend test suite is run.

### Risks / open questions

- The backlog wording mentions cancelled registrations, but the user selected active-only behavior and T32 deletes cancelled rows. The plan treats row presence as the active-registration source of truth; no schema/history change is included.
- “Public details of each event” must match the existing event response conventions. During implementation, select the existing public event DTO when it fits; otherwise use a dedicated DTO that includes only public event fields and never embeds the registering user's data.
- If event associations are lazily loaded, the repository query must fetch the event data needed by the mapper within the service transaction to avoid lazy-loading failures or per-row queries.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/RegistrationServiceTest.java` (`listsOnlyTheAuthenticatedUsersRegistrationsWithPublicEventDetails`). Command: `./gradlew.bat --no-daemon --console=plain :Business:test --tests com.ubb.events.implementation.RegistrationServiceTest`.
- **Result:** Test execution compiled and ran 10 tests; the new test failed at `RegistrationServiceTest.java:67` with an assertion failure.
- **Failure reason:** The list operation was still a stub returning an empty list, so the expected mapped registration was absent. This was a behavioral red result rather than a compilation or test setup failure.

### Green (after implementation)

- **Result:** Focused command `./gradlew.bat --no-daemon --console=plain :Persistence:test --tests com.ubb.events.persistence.RegistrationRepositoryTest :Business:test --tests com.ubb.events.implementation.RegistrationServiceTest :RestServices:test --tests com.ubb.events.controllers.UserControllerApiTest --tests com.ubb.events.controllers.UserRegistrationControllerTest` passed; repository isolation/order, business mapping, and valid/invalid authenticated endpoint tests completed successfully.
- **Full suite of the affected repo:** `./gradlew.bat --no-daemon --console=plain --continue test` completed with the T34 focused tests passing. The suite retained two pre-existing Business failures in `OrganizationServiceStructureTest` and one pre-existing RestServices failure in `OrganizationAuthorizationInterceptorTest`, matching failures documented by T32. Model, Common, and Persistence passed.
- **Follow-up coverage:** After review identified missing persistence and valid-JWT success-path coverage, `RegistrationRepositoryTest` was restored with its original save/reload, null-constraint, count/find, and delete-isolation cases, and the new user-isolation/order case was added. A valid JWT MockMvc test was also added to `UserControllerApiTest`. These follow-up tests were run after the initial green run and are not pre-implementation red evidence; the service test above remains the valid red result.

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

- **Decisions:** Added a user-id-constrained repository query with event fetch and deterministic ordering. The service maps each row to a registration id plus `EventSummaryResponse`, which contains public event fields only. A dedicated `/api/users/me/registrations` controller extracts the authenticated JWT principal and delegates to the service. Persisted rows are the active-registration source of truth.
- **Alternatives considered:** Reusing `UserController` was considered, but a separate controller keeps registration behavior dependent on `RegistrationService` without expanding the existing profile service. Cancellation history was not added because T32 deletes cancelled rows and the user approved active-only behavior.
- **Limitations:** The full backend suite still has the three known unrelated failures noted above. No cancellation history is returned because T32 deletes cancelled rows.

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
