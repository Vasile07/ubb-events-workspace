# T11 - Update my profile

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T11
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** ORG-01
- **Use case(s):** none (ORG-01)
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T11-update-my-profile`
- **PR link(s):** 
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/UserServiceImplTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/UserControllerApiTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/UserControllerStructureTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Update my profile” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-01. **Use case:** none. **Depends on:** View my profile.
- `PUT /api/users/me`: update the editable fields. Only the name is editable for now (decided, more fields can be added later). The email is the identifier and cannot be changed. Validation errors use the shared error format.

Dependencies from the backlog: 10.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java` currently exposes authenticated `GET /api/users/me` and resolves the principal ID; `Business/src/main/java/com/ubb/events/UserService.java` and `implementation/UserServiceImpl.java` implement profile retrieval; `Model/src/main/java/com/ubb/events/domain/User.java` stores `firstName`, `lastName`, and `email`; `Model/src/main/java/com/ubb/events/dto/UserProfileResponse.java` exposes both name fields and email. `AuthController` and `RegisterUserRequest` show existing request validation conventions (`@Valid`, `@NotBlank`), while `GlobalExceptionHandler` maps validation failures to the shared `ErrorResponse` shape. `RestServices/src/test/java/com/ubb/events/controllers/UserControllerApiTest.java` has API-level profile/authentication coverage to extend.
- **Related completed tasks:** `changes/completed/T10-view-my-profile.md` provides the existing profile contract, principal handling, API test patterns, and backend structure; `T5-create-account.md` provides existing account/name validation context.
- **Constraints / assumptions:** Backend only; use `PUT /api/users/me`, derive the target user from the authenticated principal, and do not allow email changes. The request must contain the two editable fields `firstName` and `lastName`, as clarified by the user. Use the existing shared validation error response.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement `PUT /api/users/me` as a protected backend vertical slice that reuses the authenticated principal established by T10. The controller will validate a request containing exactly the two editable name fields, `firstName` and `lastName`, and pass the principal user ID plus those values to the Business layer. The service will load that user, update only the two name columns, persist the same entity, and return the updated public profile. Email will not be accepted as an editable field and will remain unchanged. Validation and not-found failures will use the existing shared error response and authentication rules.

The plan keeps the update in the existing `UserService`/`UserServiceImpl` path instead of adding a separate update repository abstraction because the operation targets one authenticated `User` entity. A dedicated request DTO is preferred over reusing registration or response DTOs so email, password, and response-only membership data cannot accidentally become writable. The successful response reuses `UserProfileResponse` and returns `200 OK`, as confirmed by the user, preserving the established public profile contract.

### Steps

1. **Model/DTO:** create an update request DTO with `firstName` and `lastName`, applying the existing `@NotBlank` validation convention and exposing no email, password, user ID, or membership fields. Keep the existing `UserProfileResponse` as the successful response contract.
2. **Business:** add an update method to `UserService` and implement it in `UserServiceImpl` by loading the user ID from the authenticated principal, throwing the existing `NotFoundException` when absent, changing only `firstName` and `lastName`, saving the entity through `UserRepository`, and mapping the saved user to the existing profile response. Do not look up or mutate by request-supplied email or user ID.
3. **API:** add `PUT /api/users/me` to `UserController`, accepting `@Valid` request JSON, extracting the numeric principal ID using the T10 convention, and returning the updated profile with HTTP 200. Annotate OpenAPI responses for success, 400 validation errors, 401 missing/invalid authentication, and 404 missing authenticated user.
4. **Error and immutability behavior:** rely on the existing global validation handler for missing, blank, or malformed name fields; ignore or reject attempts to change email, password, memberships, or another user’s ID according to the final request DTO and JSON binding behavior. Preserve the existing `/api/**` authentication requirement.
5. **Verification:** write API and service tests first and obtain a behavioral red result, then implement and rerun focused tests followed by the complete backend suite. Record exact commands and outcomes in the task test log.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/UpdateUserProfileRequest.java` (create).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/UserService.java` and `Business/src/main/java/com/ubb/events/implementation/UserServiceImpl.java` (add authenticated profile update operation).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java` (add protected `PUT /api/users/me` and OpenAPI metadata).
- `repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/UserControllerApiTest.java` (extend API coverage); add or extend Business service tests for update behavior.
- `repos/ubb-events-backend/Model/src/test/...` only if the project’s existing DTO validation tests require a dedicated request-level test location; otherwise keep validation covered through the API test.

### Tests to write

- Acceptance criterion 1: an authenticated API test registers and logs in a user, calls `PUT /api/users/me` with `{"firstName":"Updated","lastName":"Name"}`, asserts HTTP 200 and the updated fields, then calls `GET /api/users/me` to verify persistence. Assert the email is unchanged and the response does not expose the password.
- Acceptance criterion 1: a Business unit test verifies the service loads the principal’s user, changes only first and last name, saves that entity, and maps the saved profile; verify no email mutation and no repository save when the user is missing.
- Acceptance criterion 2: API tests verify unauthenticated and malformed/tampered JWT requests return 401; a valid token for a nonexistent user returns the shared 404 shape; missing or blank `firstName`/`lastName` returns 400 with field-level shared errors; and the request cannot select another profile through query/body fields. Include a regression assertion that email remains the original value after update.
- Follow the required TDD order: tests compile and fail behaviorally before implementation, then focused tests pass, followed by `.\gradlew.bat test --rerun-tasks` for the backend.

### Risks / open questions

- Successful `PUT` returns the existing `UserProfileResponse` with `200 OK`, as confirmed by the user.
- The backlog says only the name is editable and the user clarified that this means two request fields, `firstName` and `lastName`; email must remain immutable. The plan does not add trimming, case normalization, or partial-update semantics because those rules are not specified.
- Unknown JSON properties (for example `email` or `userId`) may be ignored by the current Jackson configuration. The typed request DTO ensures they cannot be applied; if the team requires rejection instead, that is a separate contract decision.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/UserServiceImplTest.java` (`updatesOnlyNamesAndPreservesEmail`, `missingUserCannotBeUpdated`) and `RestServices/src/test/java/com/ubb/events/controllers/UserControllerApiTest.java` (update success, validation, authentication cases).
- **Result:** Command `.\\gradlew.bat :Business:test --tests "com.ubb.events.implementation.UserServiceImplTest" :RestServices:test --tests "com.ubb.events.controllers.UserControllerApiTest" --rerun-tasks` completed compilation and ran the Business tests; 6 tests completed, 2 failed. Both new service tests failed with `UnsupportedOperationException: profile update not implemented` at `UserServiceImpl.updateProfile` (the intentional stub). The Gradle build stopped at `:Business:test` before executing the RestServices test task.
- **Failure reason:** This is a behavioral red result caused by the planned update operation being unimplemented, not a compile, setup, assertion, or environment failure.

### Green (after implementation)

- **Result:** Focused Business command `.\\gradlew.bat :Business:test --tests "com.ubb.events.implementation.UserServiceImplTest" --rerun-tasks` passed. The focused API command `.\\gradlew.bat :RestServices:test --tests "com.ubb.events.controllers.UserControllerApiTest" --rerun-tasks` passed after adding authenticated identity, persistence, forged-ID query/body, tampered-token, and missing-user coverage. The controller structure test asserts both `/me` GET and PUT handlers.
- **Full suite of the affected repo:** Latest command `.\\gradlew.bat test --rerun-tasks` completed successfully: `BUILD SUCCESSFUL`, all 22 actionable tasks completed across Model, Common, Persistence, Business, and RestServices.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
|   |                               |                              |                                |     |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Draft for user review: the update uses a dedicated `UpdateUserProfileRequest` containing only validated `firstName` and `lastName`; the authenticated principal supplies the target user ID. The service mutates and saves only those two fields, then returns the existing `UserProfileResponse`, preserving email and excluding password. Controller validation and global error handling provide the shared 400 response, while the existing security configuration protects the endpoint.
- **Alternatives considered:** Draft for user review: reusing registration or response DTOs was rejected because it could make email, password, ID, or memberships writable. A separate repository abstraction was unnecessary because the existing `UserRepository` loads and saves the authenticated entity. Returning 204 was not chosen because the approved contract is 200 with the updated profile.
- **Limitations:** Draft for user review: unknown JSON properties such as `email` may be ignored by current Jackson configuration; they cannot change the entity because the request DTO has no such writable field. Name trimming, normalization, and partial updates are not implemented because they are outside the approved requirements.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`
