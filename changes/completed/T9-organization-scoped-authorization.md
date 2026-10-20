# T9 - Organization-scoped authorization

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T9
- **Type:** architecture
- **Owner:** Irimies Vasile
- **Requirement(s):** ORG-05, EVT-06, R8
- **Use case(s):** 
- **Workflow:** architecture
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T9-organization-scoped-authorization`
- **PR link(s):** 
- **Tests:** `OrganizationAuthorizationServiceImplTest`, `AuthorizationAnnotationTest`, `OrganizationAuthorizationInterceptorTest` (including supplemental role/error/annotation-scan coverage); full backend Gradle suite.

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Organization-scoped authorization” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** architecture. **Requirements:** ORG-05, EVT-06, R8. **Depends on:** Init Membership entity, Sign in.
- Role model: MEMBER and MANAGER per organization, with the permissions derived from the role.
- `@RequireOrgPermission` and `@NoOrgCheck` annotations.
- A `HandlerInterceptor` registered on `/api/organizations/{orgCode}/**`: it denies methods without an annotation, reads `{orgCode}` from the URL, checks the membership and the required permission, and answers 403 for an unknown organization and for a non-member.
- A test that fails if any org-scoped controller method has neither annotation.
- A test that a member of one organization cannot access a resource of another.
- The organization is not stored in the JWT. Applied to all existing org-scoped endpoints, and it checks that no scaffold endpoint is left open.

Dependencies from the backlog: 7, 8.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Membership.java` and `MembershipRole.java` define membership role and status; `Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` is currently a bare JPA repository; `Model/src/main/java/com/ubb/events/domain/Organization.java` has the numeric `organization_id`; `RestServices/src/main/java/com/ubb/events/security/JwtAuthenticationFilter.java` puts the authenticated user ID in the security principal; `OrganizationController.java` is currently an empty scaffold.
- **Related completed tasks:** T7 Init Membership entity (`changes/completed/T7-init-membership-entity.md`) defines membership persistence. T2 Backend skeleton establishes the layered backend. T8 Sign in is merged into backend `master`; its task file remains in `changes/active/` pending the user's documentation cleanup.
- **Constraints / assumptions:** This task touches the backend only; Java 21, Spring Boot, JPA, JUnit 5, Gradle wrapper, membership-based organization authorization, and `master` are defined by `context/project-context.md`. Backend repo is clean and synchronized on `master`. User decision: organization routes use `organizationId`; no `orgCode` field will be added. T9's use case and Drive/ADR reference remain unspecified; do not infer them.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement organization authorization as shared backend infrastructure in `RestServices`, backed by a Business/Persistence membership lookup. The interceptor will resolve the authenticated user id from the T8 security context, read numeric `{organizationId}` from the URL, load the membership role and status, and enforce the minimum role declared on each handler method. `@NoOrgCheck` explicitly allows organization-path methods whose contract is available to authenticated non-members; every other organization-scoped handler must declare `@RequireOrgPermission`. Unknown organizations, missing memberships, and insufficient effective roles return the shared 403 error response. Organization data remains outside the JWT.

The effective role is defined by the approved rule: `ACTIVE` returns the stored role; `PENDING` with stored `MANAGER` returns effective `MEMBER`; `PENDING` with stored `MEMBER` returns effective `NONE`. `NONE` is an authorization-only value in a separate effective-role type (for example, `EffectiveOrgRole`); it must not be added to or persisted through the existing `MembershipRole` enum, whose stored values remain `MEMBER` and `MANAGER`. The interceptor compares that effective role with the annotation's optional minimum role. When the annotation omits its minimum role, the default is `MEMBER`. The interceptor and role semantics should be recorded in the Drive architecture document or an ADR.

### Steps

1. **API/security contract:** define `@RequireOrgPermission` with an optional minimum `EffectiveOrgRole` parameter defaulting to `MEMBER`, and `@NoOrgCheck` as a method-level marker. Define unauthenticated behavior separately from authenticated authorization failures (Spring Security remains responsible for 401).
2. **Persistence:** add a membership query keyed by authenticated user id and numeric organization id that returns the stored role and status. Keep `MembershipRepository` limited to data access and use `OrganizationRepository.existsById` or equivalent to distinguish an unknown organization from a known organization with no membership.
3. **Business:** add a separate effective-role type containing `NONE`, `MEMBER`, and `MANAGER`, plus `OrganizationAuthorizationService` and implementation to calculate the approved effective role from stored role/status and compare it to the requested minimum role. Keep persisted `MembershipRole` unchanged and keep role/status rules in Business, with no JWT organization claims and no controller/database coupling.
4. **API/interceptor:** implement `OrganizationAuthorizationInterceptor` registered only for `/api/organizations/{organizationId}/**`. Require exactly one of `@RequireOrgPermission` or `@NoOrgCheck`; resolve the path variable, call the authorization service, and return the shared 403 `ErrorResponse` for unknown organizations, non-members, and insufficient effective roles. Preserve normal controller execution for explicitly annotated no-check methods.
5. **API annotations and test endpoint:** annotate every existing org-scoped controller method, secure or remove any scaffold handler under the organization path, and add a simple test controller endpoint with explicit annotations for interceptor tests. Leave non-org endpoints outside this interceptor and document 403 responses where applicable.
6. **Verification:** write tests first, obtain a behavioral red result, then implement and run focused tests followed by the full backend suite. Record exact commands and results in this task file; stop at Gate 1 before implementation.

### Files to create / modify

- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/security/RequireOrgPermission.java` (create).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/security/NoOrgCheck.java` (create).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/security/OrganizationAuthorizationInterceptor.java` (create) and MVC configuration to register it for `/api/organizations/{organizationId}/**`.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/EffectiveOrgRole.java` (create, or the equivalent Business-layer effective-role type with `NONE`, `MEMBER`, and `MANAGER`; do not change persisted `MembershipRole`).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/OrganizationAuthorizationService.java` and `implementation/OrganizationAuthorizationServiceImpl.java` (create), using `MembershipRole` and `MembershipStatus`.
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/OrganizationRepository.java` and `MembershipRepository.java` (modify with `existsById` and user/organization membership lookup).
- Existing org-scoped controllers under `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/` (modify to add one authorization annotation per method) and a test-only controller under `RestServices/src/test/java`.
- Corresponding tests under `RestServices/src/test/java`, `Business/src/test/java`, and `Persistence/src/test/java`.

### Tests to write

- Acceptance criterion 1: interceptor tests through a simple mock/test controller prove active MEMBER access at the MEMBER minimum, MANAGER-only denial for MEMBER and success for MANAGER, PENDING MANAGER effective MEMBER behavior, PENDING MEMBER effective NONE behavior, and that organization identity is read from `{organizationId}` rather than JWT claims. Include unknown organization, non-member, missing annotation, and explicit `@NoOrgCheck` cases.
- Acceptance criterion 1: a reflection/architecture test scans every handler mapped below `/api/organizations/{organizationId}/**` and fails when the method has neither `@RequireOrgPermission` nor `@NoOrgCheck`; it also verifies no scaffold handler is left unannotated.
- Acceptance criterion 2: persistence/service tests cover lookup by user and numeric organization id, stored role/status to effective-role conversion, minimum-role comparison, and denial/error behavior; API tests verify the shared 403 `ErrorResponse` shape and status for unknown organizations, non-members, and insufficient effective roles. Include unauthenticated requests and malformed path cases as applicable.
- TDD evidence: write the tests and stubs before implementation, confirm a behavioral failure caused by missing authorization, then implement, rerun focused tests, and run `.\gradlew.bat test` for the backend.

### Risks / open questions

- No unresolved implementation questions remain from the decisions discussed so far. The task's use case and Drive/ADR metadata are still blank and should be completed for traceability when known.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `OrganizationAuthorizationServiceImplTest`, `AuthorizationAnnotationTest`.
- **Result:** `.\gradlew.bat :Business:test --tests "com.ubb.events.implementation.OrganizationAuthorizationServiceImplTest" :RestServices:test --tests "com.ubb.events.security.AuthorizationAnnotationTest" --rerun-tasks` reached test execution; `pendingManagerHasMemberAccessButNotManagerAccess` failed with `AssertionFailedError` at line 28 (2 tests completed, 1 failed).
- **Failure reason:** The service stub returned `NONE`, so the pending MANAGER membership did not produce the approved effective `MEMBER` role. This is a behavioral failure caused by the missing role/status authorization logic. The initial run was blocked by Gradle cache permissions and was rerun with approved elevated access.

### Green (after implementation)

- **Result:** Focused Business and RestServices authorization tests pass: `.\gradlew.bat :RestServices:test --tests "com.ubb.events.security.OrganizationAuthorizationInterceptorTest" :Business:test --tests "com.ubb.events.implementation.OrganizationAuthorizationServiceImplTest" --rerun-tasks`.
- **Full suite of the affected repo:** `.\gradlew.bat test --rerun-tasks` passed after the conditional interceptor configuration fix (22 actionable tasks; Business, Common, Model, Persistence, and RestServices tests all passed).
- **Supplemental post-implementation coverage:** Added tests for ACTIVE MANAGER preservation, unknown organization and non-member denial, an explicit org-A membership versus org-B access denial, insufficient-role 403/ErrorResponse shape, and an annotation scan over current organization-scoped production controllers. The interceptor fixture is a mapped test-only controller under `/api/organizations/{organizationId}/test/**`. Focused tests and the full suite both pass.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The plan used `orgCode` and treated the organization identifier as unresolved. | Use the existing numeric `Organization.id` as `{organizationId}`; do not add an `orgCode` field. | This is the user's confirmed identifier decision. |
| 2 | plan | The plan did not define how membership status affects authorization. | The repository returns stored role/status; Business computes `ACTIVE` => stored role, `PENDING MANAGER` => effective `MEMBER`, and `PENDING MEMBER` => effective `NONE`. | This is the user's confirmed effective-role rule. |
| 3 | plan | The plan assumed production endpoints were available for authorization testing. | Test the interceptor through a simple mock/test controller endpoint. | This is the user's requested test boundary. |
| 4 | plan | The plan treated T8 as an unresolved dependency. | Assume T8 is merged into backend `master`; the user will later move its task file to completed. | This is the user's repository-state clarification. |
| 5 | plan | The existing persisted `MembershipRole` enum has only `MEMBER` and `MANAGER`, while effective authorization also needs `NONE`. | Use a separate effective-role type containing `NONE`, `MEMBER`, and `MANAGER`; keep persisted `MembershipRole` unchanged. | Do not alter stored membership role values to represent the authorization-only `NONE` state. |
| 6 | plan | The default minimum role for `@RequireOrgPermission` was open. | Set the default minimum role to `MEMBER`. | This is the user's chosen default. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Effective authorization uses a separate `EffectiveOrgRole` ordered as `NONE < MEMBER < MANAGER`; membership status is translated in Business. The interceptor reads numeric `organizationId` from the path and user id from the Spring Security principal. `@RequireOrgPermission` defaults to `MEMBER`; `@NoOrgCheck` is an explicit bypass marker.
- **Alternatives considered:** Organization claims in JWT were rejected in favor of resolving membership per request. Persisting `NONE` in `MembershipRole` was rejected because it would alter the database role model.
- **Limitations:** Existing organization controller is still a scaffold without mapped handlers, so the architecture test surface is represented by direct interceptor tests and the annotation contract. The implementation returns a shared 403 JSON body for unknown organizations, non-members, and insufficient roles.

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
