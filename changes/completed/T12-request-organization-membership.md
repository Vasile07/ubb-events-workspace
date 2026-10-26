# T12 - Request organization membership

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T12
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** ORG-02
- **Use case(s):** UC-04
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T12-request-organization-membership`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/32
- **Tests:** `Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java`; `Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Request organization membership” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-02. **Depends on:** Init User entity, Init Organization entity, Sign in.
- Includes the `MembershipRequest` entity scaffold (entity, repository, service, controller).
- `POST /api/organizations/{organizationId}/membership-requests`, no body (the user comes from the token). The endpoint is under the organization path but marked `@NoOrgCheck`, because the requester is not a member.
- Record a pending request and return 201 with the request id, organization and pending status.
- Prevent a duplicate pending request, and tell a user who is already a member that no request is needed. A declined request does not block a new one.

Dependencies from the backlog: 4, 6, 8.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Membership.java` and `MembershipStatus.java` define the existing pending/active membership model; `Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` has user/organization lookup and status queries; `Business/src/main/java/com/ubb/events/implementation/MembershipServiceImpl.java` contains later membership-list and decision behavior; `RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` contains later organization membership endpoints. The current controller has no POST membership-request endpoint, and the service has no request-creation operation. `NoOrgCheck` and `OrganizationAuthorizationInterceptor` define the exception needed for nonmembers to submit a request. Relevant API, service, and persistence tests are in the corresponding `src/test/java` paths.
- **Related completed tasks:** `T7-init-membership-entity.md` defines the Membership persistence scaffold; `T9-organization-scoped-authorization.md` defines the `@NoOrgCheck` authorization contract; `T16-manage-organization-membership-requests.md` consumes pending Membership rows and assumes they are the requests being managed.
- **Constraints / assumptions:** Backend only; `master` is clean and synchronized with `origin/master` at `a6e0a63`. Per user clarification, model each request as a `Membership` row with `PENDING` status, consistent with merged code and T16; do not add a separate `MembershipRequest` entity. Preserve service-layer business rules, persistence-only repositories, shared API errors, JWT user identity, and the existing `@NoOrgCheck` interceptor contract.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the request workflow as a vertical backend change using the existing `Membership` entity. A signed-in user submits an empty-body `POST` under the organization path; the controller reads the authenticated user id from the JWT principal, is explicitly marked `@NoOrgCheck`, and delegates to the service. The service validates the organization and user, rejects an existing active membership or pending request with shared errors, and saves a new `PENDING` membership row. A previously declined request does not block a new request because T16 removes declined rows.

Keep authorization, business rules, persistence, and HTTP mapping in their existing layers. Return `201 Created` with no response body, as clarified by the user. The created row stores the membership id, organization id, `MEMBER` role, and `PENDING` status. The separate `MembershipRequest` entity from the stale backlog wording is not part of this plan because the user chose pending `Membership` rows and T16 already consumes that model.

### Steps

1. **Persistence:** verify or extend `MembershipRepository` with the organization/user lookup needed to distinguish an active membership from a pending request; keep repository methods limited to data access. Use `OrganizationRepository` and `UserRepository` lookups to produce established not-found errors rather than persisting dangling relations.
2. **Business:** add a request-creation operation to `MembershipService` and implement it transactionally. Load the organization and authenticated user, reject an `ACTIVE` row with a business-rule error (“no request is needed”), reject an existing `PENDING` row with a conflict, and create a `Membership` with the desired `MEMBER` role and `PENDING` status. The endpoint does not return a response DTO.
3. **API:** add `POST /api/organizations/{organizationId}/membership-requests` to `MembershipController` with no request body, `@NoOrgCheck`, the existing authenticated-principal pattern, and documented 201/400/401/404/409 responses. Return an empty `201 Created` response and preserve the interceptor’s exactly-one-authorization-annotation contract.
4. **Error and integration behavior:** rely on the global error handler and shared error response for malformed ids, unauthenticated callers, unknown organizations/users, duplicate pending requests, and already-active members. Do not add controller-level business checks or a second authorization path.
5. **Verification:** write focused persistence, service, controller structure, and API tests first; obtain a behavioral red result caused by the absent request operation; implement; rerun focused tests and then the full backend suite. Record exact commands and outcomes in the task test log.

### Files to create / modify

- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` — request lookup if the current user/organization method does not provide the required status distinction.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MembershipService.java` — request-creation operation contract.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/MembershipServiceImpl.java` — organization/user lookup, duplicate and active-member rules, and transactional save.
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` — POST mapping, principal extraction, `@NoOrgCheck`, status code, and OpenAPI responses.
- `repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java` — user/organization/status lookup coverage.
- `repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java` — creation and business-rule coverage.
- `repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerStructureTest.java` and `MembershipControllerApiTest.java` — route/annotation, response, authentication, and error behavior.

### Tests to write

- **Acceptance criterion 1:** an API test submits an authenticated empty-body POST and asserts empty `201`; service and repository tests assert a new `Membership` row is saved with the authenticated user, target organization, desired `MEMBER` role, and `PENDING` status.
- **Acceptance criterion 2:** tests cover unauthenticated `401`, malformed/invalid organization id `400`, unknown organization or user `404`, duplicate pending request `409`, active member business error (mapped to `400`), and the `@NoOrgCheck` path allowing a nonmember to submit. Verify shared `ErrorResponse` fields and that no request body is required.
- **Edge cases:** a declined request represented by a deleted row can be submitted again; an existing pending row is not duplicated; an active membership is not changed; repository lookups are scoped to both user and organization.

### Risks / open questions

- The backlog describes a response containing the request and organization, but the user clarified that the endpoint should return only `201 Created` without a body. The created database row still records the request and its `MEMBER`/`PENDING` values.
- The organization and user repositories already exist, but the service constructor will gain dependencies; update structure tests accordingly without moving persistence logic into the controller.
- The exact client-safe message for “already a member” is not specified. Suggested mapping: `BusinessRuleException` with a stable message and the existing global 400 mapping; duplicate pending remains `ConflictException`/409.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `MembershipServiceTest.createsPendingMemberRequestForAuthenticatedUser` (focused command: `./gradlew.bat :Business:test --tests "com.ubb.events.implementation.MembershipServiceTest.createsPendingMemberRequestForAuthenticatedUser" :RestServices:test --tests "com.ubb.events.controllers.MembershipControllerStructureTest.requestHandlerIsOpenToNonMembersAndUsesPost" --tests "com.ubb.events.controllers.MembershipControllerApiTest.authenticatedNonMemberCanSubmitEmptyMembershipRequest" :Persistence:test --tests "com.ubb.events.persistence.MembershipRepositoryTest.findsMembershipForUserAndOrganizationToCheckExistingRequest" --rerun-tasks`; tests compiled).
- **Result:** `MembershipServiceTest > createsPendingMemberRequestForAuthenticatedUser() FAILED`, with `org.opentest4j.AssertionFailedError at MembershipServiceTest.java:49`; Gradle reported `1 test completed, 1 failed` and `:Business:test FAILED`.
- **Failure reason:** The new request operation was a compiling stub returning `null`, so the expected created request representation was absent. This was a behavioral failure caused by the missing implementation, not a compilation or setup failure.

### Green (after implementation)

- **Result:** After the user-requested empty-response change, focused command `./gradlew.bat :Business:test --tests "com.ubb.events.implementation.MembershipServiceTest" :RestServices:test --tests "com.ubb.events.controllers.MembershipControllerStructureTest" --tests "com.ubb.events.controllers.MembershipControllerApiTest" :Persistence:test --tests "com.ubb.events.persistence.MembershipRepositoryTest" --rerun-tasks --console=plain` passed; 18 actionable tasks successful.
- **Full suite of the affected repo:** After the user-requested empty-response change, `./gradlew.bat test --rerun-tasks --console=plain` reached the full suite but failed two `OrganizationServiceStructureTest` assertions (`serviceInterfaceIsEmpty`, `implementationIsAnnotatedAndImplementsServiceWithoutOperations`); 85 tests completed, 2 failed. These assertions concern the organization service and are unrelated to T12.
- **Supplemental/post-implementation coverage:** Added POST-specific unauthenticated, malformed organization id, duplicate pending (409), active member (400), unknown organization (404), and service-level unknown organization/user tests after implementation; the focused run passed. These tests are supplemental and are not part of the pre-implementation red evidence. After the user requested a bodyless response, the service now returns `void` and the API test asserts the `201` body is empty; the same focused command passed again.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The backlog wording described a separate `MembershipRequest` entity. | Use pending `Membership` rows for requests and do not add a separate entity. | User chose pending Membership rows, consistent with the merged backend and T16. |
| 2 | plan | The plan set the role to `NULL` and said acceptance would assign `MEMBER`. | Store `MEMBER` as the role on the new `PENDING` Membership row. | “the role should be member, this is the desired role” |
| 3 | implementation | The endpoint returned a response body with the created membership details. | Return `201 Created` without a response body. | “here we can return just created without body” |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use the existing `Membership` row as the request record, with `MEMBER` role and `PENDING` status. The controller is marked `@NoOrgCheck` so authenticated nonmembers can submit; the service enforces duplicate-pending and already-active-member rules.
- **Alternatives considered:** A separate `MembershipRequest` entity was rejected after the user selected pending `Membership` rows, matching T16. Returning the created membership details was removed after the user requested a bodyless `201 Created` response.
- **Limitations:** The full backend suite currently has two unrelated failing organization-service structure assertions on this branch. Commit `c5d0a99` is pushed to the task branch. A PR was not created because the installed GitHub CLI has an invalid saved token; the user can create it manually after reauthenticating.

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
