# T8 - Sign in

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T8
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** ORG-01
- **Use case(s):** UC-02, UC-03
- **Workflow:** workflow
- **Drive doc / ADR:** [Requirements and use cases](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban)
- **Repo(s):** backend
- **Branch:** `T8-sign-in`
- **PR link(s):** 
- **Tests:** `Common/src/test/java/com/ubb/events/common/security/JwtTokenServiceTest.java`; `Business/src/test/java/com/ubb/events/implementation/UserServiceImplTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/AuthControllerApiTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/HealthControllerTest.java`; `RestServices/src/test/java/com/ubb/events/security/JwtAuthenticationFilterTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Sign in” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-01. **Depends on:** Create account.
- `POST /api/auth/login`: JWT-based sign in, with the validation filter that sets the current user.
- The token holds only the user identity and expiry, not the organizations.
- The login response returns a JWT only. Organization membership status is provided by `GET /api/organizations` (T13), not by login or JWT claims.
- Same message for any invalid credentials, and a clear response when credentials are missing.
- Log out (UC-03) is client-side only, so there is no endpoint. An expired or invalid token gets a 401.

Dependencies from the backlog: 5.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/AuthController.java` currently exposes register only; `Business/src/main/java/com/ubb/events/UserService.java` and `implementation/UserServiceImpl.java` handle registration; `Persistence/src/main/java/com/ubb/events/persistence/UserRepository.java` has email existence lookup; `Model/src/main/java/com/ubb/events/domain/User.java` stores the encoded password.
- **Related completed tasks:** T5 Create account (`changes/completed/T5-create-account.md`); T4 User entity.
- **Constraints / assumptions:** JWT token contains only user identity and expiry; login returns the JWT only; no server logout endpoint. Backend master is clean and up to date. Membership status is retrieved separately through T13; T8 does not query memberships. T8 requires account creation but not membership data.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement sign-in as a backend vertical slice under the existing `/api/auth` controller. Look up the user by email, compare the submitted password in the Business layer, and issue a signed JWT containing only the user identity and expiry. Add a request filter that validates the token and sets the current user identity for valid requests; malformed, invalid, or expired tokens return 401. The login response contains the JWT only. The UI obtains organization membership status separately from `GET /api/organizations` (T13), which already specifies `MEMBER`, `PENDING`, or `NONE`. Membership status and roles remain outside the JWT and are not authorization truth. Logout stays client-side.

Configure the JWT signing secret as `JWT_SECRET` in `.env.example` with a placeholder value, and map it as `app.jwt.secret=${JWT_SECRET}` in `RestServices/src/main/resources/application.properties` using the existing optional `.env` import; require the secret to be set, with no hardcoded default. Do not hardcode a secret or commit a real value. Sign JWTs using HMAC-SHA256 (HS256) and set the expiry to 24 hours after issuance.

### Steps

1. **Persistence and request model:** add the UserRepository lookup by email and a validated login request DTO. Add a token-only login response DTO if required by existing conventions; do not add user, membership, or role payload fields.
2. **Security:** add JWT signing/parsing, expiry configuration, and a bearer-token filter. Valid tokens set the current user identity; malformed, invalid, or expired tokens return 401. Configure login and registration as public routes and protect the routes that require authentication.
3. **Business:** extend the user/authentication service to validate credential presence, find the user by email, compare the submitted password, return the same client-safe error for an unknown email and incorrect password, and issue a JWT on success.
4. **API:** add `POST /api/auth/login` while preserving registration. Return the JWT only; document success, missing credentials, and invalid credentials in the OpenAPI responses using the shared error shape.
5. **Verification:** write focused persistence, service/security, and API tests first; confirm a behavioral red result, implement the slice, rerun focused tests, then run the full backend suite and record exact evidence here.

### Files to create / modify

- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/UserRepository.java` (email lookup).
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/...` (validated login request and token-only response DTO if needed).
- `repos/ubb-events-backend/Common/src/main/java/com/ubb/events/common/security/...` (JWT component, bearer authentication filter, and related security configuration/properties).
- `repos/ubb-events-backend/.env.example` (document `JWT_SECRET` with a non-secret placeholder) and `repos/ubb-events-backend/RestServices/src/main/resources/application.properties` (map `JWT_SECRET` to `app.jwt.secret`, with no default).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/UserService.java` and `implementation/UserServiceImpl.java` (credential validation and token issuance, or a narrowly scoped authentication service if existing conventions require it).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/AuthController.java` (add login while preserving register).
- Corresponding tests under `Persistence/src/test/java`, `Business/src/test/java`, `Common/src/test/java`, and `RestServices/src/test/java`.

### Tests to write

- Valid login returns a signed JWT; decoded claims contain only user identity and expiry, and the response has no user, membership, or role payload. Verify JWT signing reads the secret mapped from `JWT_SECRET`, uses HMAC-SHA256 (HS256), and expires after 24 hours.
- Password comparison and email lookup work; unknown email and incorrect password produce the same status and exact client-safe message, with no token.
- Missing email, password, or both produce clear field-level validation errors in the shared error response shape.
- A valid bearer token sets the current user identity; malformed, tampered, invalid, and expired tokens return 401.
- No logout endpoint is added; logout remains client-side.
- Follow the required TDD order: behavioral red before implementation, then focused green tests and the wider backend suite.

### Risks / open questions

- T13 will return each organization's status and role from `GET /api/organizations`; status NONE maps to role NONE. For PENDING memberships, T13 reports the actual role: desired MANAGER maps to MEMBER, and desired MEMBER maps to NONE; desired role is always set.
- JWT secret is supplied through `JWT_SECRET`, documented in `.env.example`, and mapped to `app.jwt.secret` by `application.properties` with no default. The backend has no existing JWT convention; this task uses HMAC-SHA256 (HS256) with a 24-hour expiry.
- The current repository has no JWT filter implementation, so T8 includes the minimal security infrastructure needed to validate tokens and set the current user. Organization-scoped authorization must remain membership-based and must not read organization data from JWT claims.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `.\gradlew.bat :Business:test --tests com.ubb.events.implementation.UserServiceImplTest :RestServices:test --tests com.ubb.events.controllers.AuthControllerApiTest`
- **Result:** Valid test execution reached `UserServiceImplTest.authenticatesValidCredentialsAndReturnsToken()` and failed with an assertion (`expected token, observed stub`); 4 tests ran, 1 failed. The initial non-elevated run was blocked by Gradle cache permissions, then the same command ran successfully through the approved elevated path.
- **Failure reason:** The behavioral failure demonstrated that login still returned the placeholder token and did not perform credential verification/token issuance. This was a valid red result; it was not a compilation or test setup failure.

### Green (after implementation)

- **Result:** Focused JWT, business, and API tests passed with `.\gradlew.bat :Common:test --tests com.ubb.events.common.security.JwtTokenServiceTest :RestServices:test --tests com.ubb.events.controllers.AuthControllerApiTest` and the business authentication test included in the earlier focused run. Coverage includes HS256/24-hour claims, token-only login, validation errors, equal invalid-credential responses, and invalid bearer-token 401 handling.
- **Full suite of the affected repo:** `.\gradlew.bat test` passed: all 29 RestServices tests and all tests in Model, Common, Persistence, and Business completed successfully.
- **Post-implementation verification deviation:** The initial implementation review identified that valid-principal propagation and expired/malformed/tampered bearer cases were not separately covered, and the filter used servlet `sendError` instead of the shared JSON error shape. Added `JwtAuthenticationFilterTest` and API assertions, changed the filter to return `ErrorResponse` JSON with status 401, then reran `.\gradlew.bat :RestServices:test --tests com.ubb.events.security.JwtAuthenticationFilterTest --tests com.ubb.events.controllers.AuthControllerApiTest` (pass) and `.\gradlew.bat test` (pass). These are post-implementation verification results, not pre-implementation red evidence.

- **Class DTO correction verification:** After replacing the login request/response records with classes and updating call sites to use getters, the first suite run exposed stale record-style accessors in the business test and controller. Those were corrected; the final `.\gradlew.bat test --console=plain` run passed (`BUILD SUCCESSFUL`).

- **API documentation correction verification:** After adding the 401 `@ApiResponse` declaration to the invalid-credentials handler, `.\gradlew.bat test --console=plain` passed (`BUILD SUCCESSFUL`).

- **Constructor removal verification:** After removing the unused two-argument `UserServiceImpl` constructor, `.\gradlew.bat test --console=plain` passed (`BUILD SUCCESSFUL`).

- **Security configuration formatting verification:** After reformatting the authorization matcher chain without changing its rules, `.\gradlew.bat test --console=plain` passed (`BUILD SUCCESSFUL`).

- **Post-rebase verification:** After rebasing onto the latest `origin/master`, `.\gradlew.bat test --console=plain` passed (`BUILD SUCCESSFUL`).

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | Login response includes memberships and roles. | Revised the plan to return only the JWT from login and obtain membership status from T13. | Keep login focused on authentication; organization status is returned by the organization list. |
| 2 | plan | T7 was listed as a dependency, T13 did not include role, and JWT secret configuration was unspecified. | Removed T7 from T8 dependencies; added role to T13 organization responses with NONE status mapping to NONE role; specified JWT_SECRET in .env.example and application.properties. | Remove the unnecessary membership dependency and make the API response and secret configuration requirements explicit. |
| 3 | implementation | Login request and response DTOs used records. | Replaced them with classes following the existing backend DTO convention. | The user specified that request and response types must use classes. |
| 4 | implementation | The InvalidCredentialsException handler lacked an @ApiResponse specification. | Added a 401 @ApiResponse with the shared ErrorResponse schema. | Document the handler's unauthorized response in the API specification. |
| 5 | implementation | The unused two-argument UserServiceImpl constructor passed null for JwtTokenService. | Removed the constructor; production and test callers use the three-argument constructor. | No current caller needed the overload, and it could create an unusable service instance. |
| 6 | implementation | The authorization matcher chain was too long to fit on screen. | Split the allowlisted matchers and authorization rules across multiple lines. | Keep the security configuration readable without changing its rules. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** JWT issuance is centralized in `JwtTokenService`, using HS256 with a 24-hour expiration and only `sub` plus `exp` claims. `JWT_SECRET` is required through `app.jwt.secret=${JWT_SECRET}`; no real secret is committed. A stateless Spring Security filter validates bearer tokens and places the numeric user id in the security context. Login returns a token-only DTO. Login request and response DTOs are classes, following the existing backend DTO convention; membership data remains outside authentication.
- **Alternatives considered:** Memberships and roles could have been embedded in the login response or JWT, but the approved plan keeps them in T13’s organization listing. Server-side logout was rejected because logout is client-side token disposal.
- **Limitations:** The current scaffold has no organization endpoints that consume the authenticated principal yet; later organization workflows must enforce membership authorization using the security context and persistence data. Invalid credentials use a shared 401 message, while missing fields use shared 400 validation errors.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`
