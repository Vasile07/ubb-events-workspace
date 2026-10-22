# T10 - View my profile

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T10
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** ORG-01
- **Use case(s):** none (ORG-01)
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T10-view-my-profile`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/13
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/UserProfileServiceTest.java`, `Business/src/test/java/com/ubb/events/implementation/UserServiceImplTest.java`, `Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java`, `RestServices/src/test/java/com/ubb/events/controllers/UserControllerApiTest.java`, `RestServices/src/test/java/com/ubb/events/controllers/UserControllerStructureTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View my profile” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-01. **Use case:** none (the requirement says users manage their account details). **Depends on:** Init User entity, Sign in.
- `GET /api/users/me`: user id, name, email and the memberships with their roles. Authentication required, no body.

Dependencies from the backlog: 4, 8.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java` is an empty `/api/users` scaffold; `RestServices/src/main/java/com/ubb/events/security/SecurityConfig.java` requires authentication for `/api/**`, and `JwtAuthenticationFilter` stores the authenticated user ID as the principal. `Business/src/main/java/com/ubb/events/UserService.java` and `implementation/UserServiceImpl.java` provide user lookups and profile-related service conventions; `Model/src/main/java/com/ubb/events/dto/UserResponse.java` maps public user fields without the password. `Model/src/main/java/com/ubb/events/domain/Membership.java`, `MembershipRole.java`, `MembershipStatus.java`, and `Organization.java` define membership and organization data; `Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` currently has only a user-and-organization lookup.
- **Related completed tasks:** `changes/completed/T4-init-user-entity.md` established the User entity and `T8-sign-in.md` established JWT authentication and the authenticated principal.
- **Constraints / assumptions:** Backend only; protected `/api/users/me` must derive the identity from the authenticated principal, never from request input. Password is not returned. Memberships exist as separate records, and each membership in the profile response includes its organization ID and role, as clarified by the user.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the profile endpoint as a backend vertical slice over the existing authenticated-user context. The controller will expose `GET /api/users/me`, obtain the user ID from the JWT-populated principal, and pass it to the Business layer; no request parameter or body can select another user. The service will load the user and that user’s memberships, then map only the public profile fields (id, first name, last name, email) plus a plural `memberships` field whose entries contain `organizationId` and a non-null `role`. Password data and unrelated organization fields stay out of the response. Reuse the existing repository and shared error handling conventions, adding only the membership lookup and DTOs needed by this contract.

The endpoint remains under the existing `/api/**` authentication rule rather than introducing controller-level authorization logic. Requests without credentials and requests with an invalid or expired JWT both return 401 through the existing security handling. A valid token whose user record is absent is treated as not found through the shared exception handler. T10 returns actual memberships only: ACTIVE memberships expose their stored MEMBER or MANAGER role; a PENDING request with stored role MANAGER (desired MANAGER) is returned with actual role MEMBER; a PENDING request with stored role MEMBER (desired MEMBER) is omitted.

### Steps

1. **Model/DTO:** add a profile response DTO containing the profile fields `id`, `firstName`, `lastName`, and `email`, plus a plural `memberships` field. Each nested membership entry contains only `organizationId` and a non-null `role`; ensure no password or entity object is serialized.
2. **Persistence:** extend `MembershipRepository` with a user-scoped list query such as `findAllByUserId(Long userId)`, preserving repository-only database access and deterministic handling of an empty membership list.
3. **Business:** add a profile method to `UserService` and implement it by loading the authenticated user, raising the existing not-found exception when the ID does not exist, loading memberships for that user, omitting PENDING records whose stored role is MEMBER, deriving actual role MEMBER for PENDING records whose stored role is MANAGER, preserving ACTIVE roles, and mapping the result to the profile DTO. Do not accept a user ID from the request or add membership authorization rules beyond ownership by the authenticated principal.
4. **API:** implement `GET /api/users/me` in `UserController`, extract the principal as the numeric user ID established by `JwtAuthenticationFilter`, return HTTP 200 with the profile DTO, and annotate the endpoint for OpenAPI responses (200, 401 for missing/invalid JWT, and 404 using the shared error shape).
5. **Verification:** write tests first and confirm a behavioral red result; then implement, rerun focused tests, and run the complete backend suite. Record exact commands and outcomes in the task test log.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/UserProfileResponse.java` (create, or the project-equivalent profile response DTO).
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/MembershipResponse.java` (create, or a nested membership DTO with organization ID and role).
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` (add user-scoped membership lookup).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/UserService.java` and `Business/src/main/java/com/ubb/events/implementation/UserServiceImpl.java` (add profile retrieval and mapping).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java` (add the protected `/me` endpoint and OpenAPI annotations).
- Corresponding tests under `Business/src/test/java`, `Persistence/src/test/java`, and `RestServices/src/test/java`; update or replace the obsolete empty-controller structure assertion.

### Tests to write

- Acceptance criterion 1: an API integration test with a valid JWT returns HTTP 200 and the authenticated user’s id, name, and email, with the plural `memberships` field whose entries contain the expected organization ID and non-null role; assert that password and unrelated organization fields are absent.
- Acceptance criterion 1: service tests verify the user lookup, membership lookup, DTO mapping, empty membership list, ACTIVE MEMBER/MANAGER role preservation, PENDING stored MANAGER mapping to actual MEMBER, and omission of PENDING stored MEMBER requests; a repository integration test verifies `findAllByUserId` returns only memberships for the requested user.
- Acceptance criterion 2: API tests verify requests without a token and requests with malformed, tampered, or expired JWTs all receive 401, a valid token for a nonexistent user produces the shared 404 response, and the endpoint cannot be redirected to another user through query parameters or a request body. Include multiple ACTIVE memberships and both PENDING cases to cover actual-membership filtering and role mapping.
- Follow the required TDD order: tests compile and fail behaviorally before implementation, then focused tests pass, followed by the wider backend suite.

### Risks / open questions

- The existing scaffold test expects zero controller methods and must be updated when `/me` is introduced; this is an expected task change, not a new endpoint.
- The profile contract requires a non-null actual role and returns actual memberships only. ACTIVE MEMBER/MANAGER values are returned as stored; PENDING stored MANAGER (desired MANAGER) maps to actual MEMBER; PENDING stored MEMBER (desired MEMBER) is omitted from `memberships`.
- Spring Security currently stores the principal as a `Long`; the controller should validate/cast that established representation. Missing, malformed, tampered, and expired JWTs must all use the existing 401 handling.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/UserProfileServiceTest.java` and `Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java`.
- **Result:** Command `.\gradlew.bat :Business:test --tests "com.ubb.events.implementation.UserProfileServiceTest" :Persistence:test --tests "com.ubb.events.persistence.MembershipRepositoryTest" --rerun-tasks` completed compilation and ran the new service tests; all 3 service tests failed behaviorally (`mapsActualMembershipsAndFiltersPendingRequests`, `returnsEmptyMemberships`, `missingUserIsNotFound`). The stub `getProfile` returned `null`, causing two `NullPointerException`s and one assertion failure expecting `NotFoundException`.
- **Failure reason:** The tests reached the service method and failed because profile lookup/membership mapping behavior was absent. This was a valid behavioral red result; the repository test was not reached because Gradle stopped after the failing Business task.

### Green (after implementation)

- **Result:** Focused command `.\gradlew.bat :Business:test --tests "com.ubb.events.implementation.UserProfileServiceTest" :Persistence:test --tests "com.ubb.events.persistence.MembershipRepositoryTest" :RestServices:test --tests "com.ubb.events.controllers.UserControllerApiTest" --tests "com.ubb.events.controllers.UserControllerStructureTest" --rerun-tasks` passed. The profile service, repository query, authenticated endpoint, 401 cases, 404 for a valid token whose user is absent, query/body isolation, and updated controller structure test all passed. The two 404/input-isolation API cases were added after the original red run as post-implementation verification.
- **Full suite of the affected repo:** `.\gradlew.bat test --rerun-tasks` passed all backend module tests (22 actionable tasks) after the post-implementation API coverage additions, constructor cleanup, import correction, and final membership predicate clarification. Focused Business rerun `.\gradlew.bat :Business:test --tests "com.ubb.events.implementation.UserServiceImplTest" --tests "com.ubb.events.implementation.UserProfileServiceTest" --rerun-tasks` also passed.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | `memberships` is a plural profile DTO field; unauthenticated and invalid JWTs both yield 401; membership role is never null; ACTIVE roles are actual stored MEMBER/MANAGER; PENDING desired MANAGER maps to actual MEMBER; PENDING desired MEMBER records are omitted because T10 returns actual memberships only. | Revised the plan, DTO shape, tests, and risks to use plural `memberships`, separate profile and nested membership fields, cover 401 for missing/invalid JWTs, require non-null role mapping, document the ACTIVE and PENDING desired MANAGER rules, and omit PENDING desired MEMBER records. | As clarified by the user: T10 returns actual memberships only, so PENDING requests whose stored role is MEMBER are omitted. |
| 2 | implementation | The `NotFoundException` reference was fully qualified in the service implementation. | Added the import and changed the throw site to use the simple `NotFoundException` name. | As requested by the user for clearer implementation code. |
| 3 | implementation | The membership filter was not suggestive enough about which pending case is excluded. | Changed it to the negated combined check `!(status == PENDING && role == MEMBER)`, so only PENDING MEMBER requests are excluded. | As finally clarified by the user for clearer membership filtering logic using AND with equality checks. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** The authenticated principal remains the sole identity source. Profile membership mapping is kept in the Business service: ACTIVE roles are preserved, pending MANAGER requests expose actual role MEMBER, and pending MEMBER requests are omitted. A dedicated DTO prevents password/entity serialization. Missing credentials now use the security entry point to return the required 401 error shape.
- **Alternatives considered:** Returning the persistence entities or the existing public `UserResponse` was rejected because entities could expose unrelated organization and password fields and `UserResponse` has no memberships. A request user ID was rejected because it would allow selecting another profile.
- **Limitations:** The API integration test creates a user and verifies the profile shape with an empty membership list; service tests cover the membership status and role combinations. Expired-token behavior remains covered by the existing JWT/security tests rather than generating a time-expired token in this new class.

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
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`
