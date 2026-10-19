# T5 - Create account

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T5
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** ORG-01
- **Use case(s):** UC-01
- **Workflow:** workflow
- **Drive doc / ADR:** [Requirements and use cases (ORG-01 / UC-01)](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban)
- **Repo(s):** backend
- **Branch:** `T5-create-account`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/5 (merged)
- **Tests:** `Model/src/test/java/com/ubb/events/domain/UserTest.java`; `Persistence/src/test/java/com/ubb/events/persistence/UserRepositoryTest.java`; `Business/src/test/java/com/ubb/events/implementation/UserServiceImplTest.java`; `Common/src/test/java/com/ubb/events/common/configs/PasswordEncoderConfigTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/AuthControllerApiTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/UserControllerStructureTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Create account” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-01. **Depends on:** Init User entity.
- `POST /api/auth/register`: validation of the required fields (name, email, password), unique email address, password hashing, "email already used".
- Returns 201 with the created user details (never the password).

Dependencies from the backlog: 4.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/User.java`, `Persistence/src/main/java/com/ubb/events/persistence/UserRepository.java`, `Business/src/main/java/com/ubb/events/UserService.java`, `Business/src/main/java/com/ubb/events/implementation/UserServiceImpl.java`, and the existing empty `RestServices/src/main/java/com/ubb/events/controllers/UserController.java`. Existing tests are `Model/src/test/java/com/ubb/events/domain/UserTest.java`, `Persistence/src/test/java/com/ubb/events/persistence/UserRepositoryTest.java`, and `Business/src/test/java/com/ubb/events/implementation/UserServiceImplTest.java`.
- **Related completed tasks:** `T4-init-user-entity.md` supplies the User persistence/service scaffold and states password is never exposed; `T3-api-conventions-and-error-handling.md` establishes Bean Validation and shared error response conventions.
- **Constraints / assumptions:** Backend only; Java 21/Spring Boot, JPA, Gradle wrapper, Lombok conventions. Registration is `POST /api/auth/register`, returns 201 without password, validates input, and reports duplicate email through the shared conflict/error handling. Existing User fields are `firstName`, `lastName`, `email`, and `password`; password hashing belongs in the business layer and must not be returned. The generic backlog field `name` means separate request fields `firstName` and `lastName`, matching the User entity.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add a separate public AuthController with a registration endpoint at `POST /api/auth/register`, while keeping the UserController base path at `/api/users` for profile and other user routes. Keep request validation and HTTP mapping in the RestServices layer, put duplicate-email detection and password hashing in the Business layer, and use the existing User repository only for persistence. Return a dedicated user response DTO containing the created identifier, `firstName`, `lastName`, and `email`, so the stored or submitted password cannot cross the API boundary. Use the shared conflict and validation handling established by T3; registration does not require authorization because it creates the account used for later sign-in.

The plan keeps the existing User service scaffold and adds the registration path, while correcting the User password mapping to remove password uniqueness. AuthController shares the existing UserService instead of introducing a separate authentication service. Registration remains public and requires no authentication.

### Steps

1. **Persistence/model specification:** update `docs/entities/User.md` and `Model/src/main/java/com/ubb/events/domain/User.java` so password storage is required but not unique, while email remains unique; add an email lookup method to `UserRepository` (`existsByEmail` or equivalent) for the duplicate-email rule.
2. **Business:** extend the user service with a registration operation that rejects an already-used email with `ConflictException`, hashes the raw password through the BCrypt `PasswordEncoder`, saves the new User, and returns a password-free result. Preserve the repository as the only persistence dependency and avoid returning the raw or hashed password to the API.
3. **API/model:** add validated registration request fields `firstName`, `lastName`, `email`, and `password`, plus a password-free response DTO. Add `AuthController` at `/api/auth` with `POST /register`, map the request to the service, return HTTP 201 with the response DTO, and annotate the operation and documented 201/400/409 responses for OpenAPI. Keep `UserController` mapped to `/api/users`.
4. **Configuration:** create the BCrypt password encoder in the `Common` module and inject it into the business service. Keep the registration endpoint public with no authentication or authorization requirement.
5. **Verification:** execute the focused module tests, then the affected backend suite, and record the required red and green results in this task before implementation review.

### Files to create / modify

- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/UserRepository.java` (add email lookup method).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/UserService.java` (add registration contract).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/UserServiceImpl.java` (duplicate check, hashing, persistence, and password-free result mapping).
- `docs/entities/User.md` (update the User password constraint to remove uniqueness).
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/User.java` (remove password column uniqueness).
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/...` (validated registration request and password-free user response DTOs, following the existing DTO conventions).
- `repos/ubb-events-backend/Common/src/main/java/com/ubb/events/common/...` (BCrypt `PasswordEncoder` bean/configuration).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/AuthController.java` (add `/api/auth/register`).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java` (retain `/api/users` base path).
- Corresponding tests under `Persistence/src/test/java`, `Business/src/test/java`, and `RestServices/src/test/java` in the backend repository.

### Tests to write

- Acceptance criterion 1: an API integration test posts a valid request and asserts `201 Created`, the returned `id`, `firstName`, `lastName`, and `email`, and the absence of any `password` field; a service test verifies the persisted password is encoded and is not equal to the raw password.
- Acceptance criterion 1: a duplicate-email service/API test asserts the shared conflict response (`409`) and confirms no second User is persisted.
- Acceptance criterion 1: API validation tests cover missing/blank `firstName`, `lastName`, `email`, and `password`, plus a password that violates each rule (minimum 8 characters, at least one number, uppercase letter, lowercase letter, and special character), and assert the shared validation response (`400`) with field-level errors.
- Acceptance criterion 2: repository/service tests cover email lookup and the duplicate branch; API tests cover the public route, 201 mapping, validation, conflict mapping, and password omission. The tests must use the real Spring MVC/error handler wiring and configured test persistence/security support where applicable.
- Write these tests before the implementation, run them to obtain a behavioral red result, then implement and rerun the focused tests and the wider backend suite.

### Risks / open questions

- The password policy is fixed for this task at minimum 8 characters with at least one number, uppercase letter, lowercase letter, and special character; the request DTO and tests must enforce each rule.
- Password storage must not be unique. Update both the User entity mapping and the `docs/entities/User.md` specification; email remains the unique account key.
- BCrypt encoding is provided from the Common module. Login/JWT authorization remains out of scope.
- The endpoint is intentionally public and requires no authentication or organization membership authorization.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `AuthControllerApiTest` using `./gradlew.bat :RestServices:test --tests com.ubb.events.controllers.AuthControllerApiTest --rerun-tasks` for the controlled red sensitivity check.
- **Result:** The original pre-implementation red attempt was blocked by Gradle wrapper lock-file access, and the next attempt found a matcher defect rather than a product failure. Later, after implementation, the registration save operation was temporarily disabled and the API test class was rerun. Eight tests executed; `registersUserAndDoesNotReturnPassword` and `rejectsDuplicateEmailWithConflict` failed as expected, while the five password-policy cases and other validation test passed. The committed service implementation was restored immediately afterward.
- **Failure reason:** With registration disabled, the success and duplicate-email tests received server errors instead of their expected `201 Created` responses. This delayed controlled check confirms the API tests detect missing registration behavior; it does not replace the missing pre-implementation red run.

### Green (after implementation)

- **Result:** Focused Business, Persistence, Common, and RestServices T5 tests pass. After the controller split, `AuthControllerApiTest`, `UserControllerStructureTest`, and API documentation tests passed. The API suite covers valid registration, password omission, duplicate email conflict, and each password rule (minimum length, number, uppercase, lowercase, special character).
- **Full suite of the affected repo:** After the delayed red sensitivity check and source restoration, `./gradlew.bat test --rerun-tasks` completed successfully on 2026-10-19 (`BUILD SUCCESSFUL`, 22 actionable tasks).

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The backlog says “name,” which is generic wording. | Use `firstName` and `lastName` in the request, matching the User entity. | The user clarified that the request should have separate first and last name fields. |
| 2 | plan | Password validation must be stronger than presence-only validation. | Require at least 8 characters, one number, one uppercase character, one lowercase character, and one special character. | The user specified the password policy. |
| 3 | plan | The password must not be unique, and the persisted User specification must reflect that. | Remove password uniqueness from the User entity and update `docs/entities/User.md`. | The user explicitly requested this correction and documentation update. |
| 4 | plan | Password encoding and endpoint access needed explicit decisions. | Create a BCrypt password encoder in Common; keep registration public with no authentication. | The user specified the encoder location and public endpoint behavior. |
| 5 | implementation | The registration endpoint was placed on `UserController`, changing its base path away from `/api/users`. | Restore `UserController` to `/api/users` and add a separate `AuthController` at `/api/auth` for registration. | The user clarified that user profile and other user features belong under `/api/users`, while authentication routes belong under `/api/auth`. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Registration is handled by the existing User service and exposed by a separate public `AuthController` at `POST /api/auth/register`. The `UserController` retains its `/api/users` base path for profile and other user routes. The request DTO validates names, email, and the five-part password policy. The service checks email uniqueness before saving and hashes the raw password with a BCrypt `PasswordEncoder` bean from Common. The response is a dedicated password-free DTO. Password storage is required but not unique; email remains unique.
- **Alternatives considered:** A separate authentication service was unnecessary; `AuthController` shares the existing UserService, while `UserController` remains available for user profile routes. A generic password encoder abstraction was retained through Spring's `PasswordEncoder` interface so the BCrypt implementation can be replaced centrally.
- **Limitations:** Login, JWT/session issuance, email verification, and password reset are outside T5. The initial pre-implementation red execution was unavailable because of the Gradle wrapper cache permission error; the delayed controlled red check verifies test sensitivity but does not reproduce the original TDD timing. The full green suite was rerun afterward.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing before implementation (red; delayed sensitivity check only, see Test log)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
