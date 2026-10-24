# T18 - Promote organization member to manager

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T18
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):** ORG-04, ORG-05
- **Use case(s):** UC-16
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T18-promote-organization-member-to-manager`
- **PR link(s):** 
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Promote organization member to manager” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-04, ORG-05. **Depends on:** Init Membership entity, Organization-scoped authorization.
- `PATCH /api/organizations/{organizationId}/members/{userId}/role` with `MANAGER`, for managers only. Promotion only: there is no demotion.
- Reject users who are not members, and inform when the user is already a manager.

Dependencies from the backlog: 7, 9.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

  - **Relevant code:** `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Membership.java` stores user, organization, role, and status; `Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` already has `findByUserIdAndOrganizationId`; `Business/src/main/java/com/ubb/events/MembershipService.java` and `implementation/MembershipServiceImpl.java` contain the adjacent membership workflows; `RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` exposes those workflows and uses `@RequireOrgPermission(EffectiveOrgRole.MANAGER)` for manager-only actions. Existing focused tests are `Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java` and `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`.
  - **Related completed tasks:** T7 Init Membership entity, T9 Organization-scoped authorization, T16 Manage organization membership requests, and T17 View organization members.
  - **Constraints / assumptions:** Backend only; Java 21, Spring Boot, JPA, JUnit 5, and Gradle wrapper are established in `../../context/project-context.md`. Organization IDs are numeric path variables. Backend repo was clean on `master` with no unpushed commits; fetched and confirmed up to date with `origin/master`. User clarified that promotion requires an ACTIVE membership and an already-manager target returns HTTP 409.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the promotion as a backend-only vertical workflow across the existing membership service and controller. Add a small request/response contract for the `MANAGER` role, keep organization-scoped manager authorization on the endpoint, and place all membership lookup, active-status validation, promotion-only behavior, persistence, and conflict handling in `MembershipServiceImpl`. Use the existing `MembershipRepository.findByUserIdAndOrganizationId` lookup and shared `NotFoundException`/`ConflictException` mappings so the API returns the established error shape and status codes. A separate role-management abstraction or demotion endpoint is not needed because the backlog scope is one promotion operation.

### Steps

1. **Business/service layer:** extend `MembershipService` with a `promoteToManager(organizationId, userId)` operation. In `MembershipServiceImpl`, load the membership within the organization, return not found for nonmembers, reject a non-`ACTIVE` membership with the established business-rule error mapping, return a conflict when the current role is already `MANAGER`, set the role to `MANAGER`, and save the membership transactionally. Do not add demotion behavior; the controller validates that the request body selects `MANAGER` before invoking this operation.
2. **API/controller layer:** add `PATCH /api/organizations/{organizationId}/members/{userId}/role`, protected with `@RequireOrgPermission(EffectiveOrgRole.MANAGER)`, validating the role request and mapping the service result to the established response/status conventions. Document success, authentication, forbidden, validation, not-found, and conflict responses with OpenAPI annotations.
3. **Error and contract alignment:** use the existing global exception handler and `ErrorResponse` mapping; keep messages client-safe and ensure organization scoping is enforced by the repository lookup plus the authorization interceptor. No frontend changes are in scope.
4. **TDD sequence:** first add service unit tests and controller API tests (including the minimum compile stubs), run the focused tests to obtain a behavioral red result, record the exact command/failures in the task log, then implement the workflow and rerun focused tests for green before the affected backend suite.

### Files to create / modify

- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MembershipService.java` — add the promotion service operation.
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/MembershipServiceImpl.java` — implement active-member lookup, promotion-only rule, conflict/not-found validation, and persistence.
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` — expose and document the manager-only PATCH endpoint.
- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/RolePromotionRequest.java` and `RolePromotionResponse.java` — request and response contracts for promotion.
- `../../repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java` — unit tests for promotion rules and persistence.
- `../../repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java` — API-level authorization, validation, success, not-found, and conflict coverage.

### Tests to write

- **Acceptance criterion 1 (backlog behavior implemented):** service test promotes an `ACTIVE` `MEMBER` in the requested organization to `MANAGER`, saves exactly that membership, and returns the expected response; verify the role remains unchanged for invalid/non-promotion inputs.
- **Acceptance criterion 1 (business/error rules):** service tests cover a missing membership as `NotFoundException`, a non-`ACTIVE` membership as `BusinessRuleException` (mapped by the existing handler), and an already-`MANAGER` membership as `ConflictException` (mapped to HTTP 409). Include organization scoping by stubbing the repository with both IDs.
- **Acceptance criterion 2 (authorization and API contract):** MockMvc API tests cover unauthenticated (401), authenticated non-manager (403), manager success (200 with `MANAGER` result), malformed organization/user identifiers or invalid/missing role body (400), nonmember/not-found (404), and already-manager conflict (409). Assert the shared error response status/message fields where applicable.
- Tests must be written before the production implementation and must be run first to demonstrate a behavioral failure caused by the missing endpoint/service behavior.

### Risks / open questions

- The user approved request body `{ "role": "MANAGER" }` and a response containing the promoted `userId` and `role`.
- The repository currently exposes the needed user/organization lookup. The inactive-membership rejection should use the existing `BusinessRuleException` mapping (400); only the already-manager condition is required to be a 409 conflict.
- No frontend work, demotion, bulk promotion, or audit-history feature is included.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** Added promotion service coverage in `Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java` and API coverage in `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`.
- **Result:** Not run, per the user's instruction to leave test execution to them. No pre-implementation behavioral red result is recorded; the red checklist remains unchecked.
- **Failure reason:** Not established because the tests were not run before implementation.

### Green (after implementation)

- **Result:** Not run, per the user's instruction to leave test execution to them.
- **Full suite of the affected repo:** Not run.

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

- **Decisions:** The endpoint is manager-only and accepts only the `MANAGER` role. The service looks up membership by user and organization, requires `ACTIVE` status, saves the role change transactionally, returns 404 for a missing membership, 400 for a non-active or invalid promotion, and 409 if the active target is already a manager. The API returns `{ "userId", "role" }` on success.
- **Alternatives considered:** A general role update that supports demotion was excluded because this task specifies promotion only. A separate role-management service was unnecessary for the single operation.
- **Limitations:** Tests were added but not run at the user's request; compilation and runtime behavior still need verification by the user.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [ ] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``
