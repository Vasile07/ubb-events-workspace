# T13 - View organizations

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T13
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):** ORG-06
- **Use case(s):** UC-04
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T13-view-organizations`
- **PR link(s):** 
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/OrganizationServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerApiTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View organizations” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-06. **Depends on:** Init Membership entity, Request organization membership.
- `GET /api/organizations` for any signed-in user, outside the organization path. No pagination.
- Each organization shows its public information (organizationId, name, description), the user's status (MEMBER, PENDING or NONE; a declined request counts as NONE), and the user's actual role, not a desired role. For status NONE, role is NONE. For status MEMBER, return the actual membership role. Pending memberships always have a desired role (MEMBER or MANAGER); a null desired role is not possible. For status PENDING, a desired role of MANAGER maps to actual role MEMBER, and a desired role of MEMBER maps to actual role NONE.
- Never expose private member or request data.

Dependencies from the backlog: 7, 12.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.
- [ ] Each organization response includes the user's actual role: NONE status returns role NONE; MEMBER status returns the actual membership role; PENDING with desired role MANAGER returns role MEMBER, and PENDING with desired role MEMBER returns role NONE.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Organization.java` defines `id`, `name`, nullable `description`, and `logoUrl`; it has no `orgCode`. `Membership.java` and `MembershipStatus.java` define the persisted membership status and role data. `MembershipRepository.java` already has `findAllByUserId(Long userId)`. `OrganizationService` and `OrganizationServiceImpl` are empty scaffolds; `OrganizationController` is mapped to `/api/organizations` but has no handlers. Existing structure and API test patterns are in `Business/src/test/java/com/ubb/events/implementation/OrganizationServiceStructureTest.java` and `RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerStructureTest.java`.
- **Related completed tasks:** `T6-init-organization-entity.md` records that Organization uses `organization_id` and explicitly has no `orgCode`; `T7-init-membership-entity.md` records the Membership scaffold and persistence conventions; `T9-organization-scoped-authorization.md` records effective membership role rules, including pending manager requests mapping to effective MEMBER and pending member requests mapping to NONE.
- **Constraints / assumptions:** Backend uses Java 21, Spring Boot, JPA, Gradle wrapper, and separate Model, Persistence, Business, and RestServices modules; API-level tests use the configured JWT/security setup as specified in `../../context/project-context.md`. The user resolved the backlog/entity mismatch by directing T13 to return `organizationId` instead of `orgCode`, consistent with T6. `.gradle-user/` is now ignored by the backend `.gitignore`; the local directory remains in place.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the read-only organization listing as a vertical backend workflow. The controller will authenticate the caller, delegate to the service, and map the result to a public response DTO; the service will load all organizations and combine each with the caller's membership/request state, including the effective-role mapping for pending requests; repositories will remain responsible only for database access. The endpoint will use `organizationId` (the persisted Organization identifier) and will not expose member or request records. Add focused unit tests for role/status mapping and integration/API tests using the real security and database setup. The existing unpaginated list shape is sufficient; pagination and organization-scoped paths are outside this task.

### Steps

1. **Repository/configuration:** add `.gradle-user/` to `../../repos/ubb-events-backend/.gitignore` so the existing local Gradle user directory is ignored; do not commit the directory itself.
2. **Model/API contract:** define the organization-list response DTO containing `organizationId`, `name`, `description`, membership status (`MEMBER`, `PENDING`, or `NONE`), and the user's actual/effective role (`MEMBER`, `MANAGER`, or `NONE`). Keep private member and request fields out of the DTO and document the endpoint response with OpenAPI annotations.
3. **Persistence:** use `OrganizationRepository` to load all organizations and the existing membership/request repository queries to obtain the signed-in user's state. Extend repository queries only when needed to distinguish active membership, pending requests, and declined requests without exposing persistence objects to the API.
4. **Business/service:** add the organization-list operation to `OrganizationService` and implement it in `OrganizationServiceImpl`. For every organization, default to `NONE`/`NONE`; map active membership to `MEMBER` plus its actual role, pending requests to `PENDING` with desired `MANAGER` becoming effective role `MEMBER` and desired `MEMBER` becoming `NONE`, and treat declined requests as `NONE`/`NONE`. Preserve the organization public fields and avoid returning private data. Handle the authenticated user identity from the established security context and propagate repository failures through the existing global error handling.
5. **API/controller/security:** add `GET /api/organizations`, require a signed-in user through the existing JWT/security configuration, pass the authenticated user id to the service, and return the list with the project-standard success and error mappings. Ensure no organization-scoped authorization interceptor is incorrectly required for this outside-the-organization-path endpoint.
6. **Verification:** run the focused module tests and then the affected backend test suite through `gradlew.bat`, recording the red and green TDD evidence in this task file after implementation.

### Files to create / modify

- `../../repos/ubb-events-backend/.gitignore` (modify: ignore `.gradle-user/`).
- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto` (create or modify the public organization-list response DTO and status/role representation as required by the established API conventions).
- `../../repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/OrganizationRepository.java` and `MembershipRepository.java` (modify only if additional read queries are required).
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/OrganizationService.java` (modify: list operation contract).
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/OrganizationServiceImpl.java` (modify: organization and membership status/effective-role rules).
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/OrganizationController.java` (modify: authenticated `GET /api/organizations` endpoint, mapping, and OpenAPI documentation).
- `../../repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/OrganizationServiceTest.java` (create/modify: service mapping and edge cases).
- `repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerTest.java` (create/modify: API-level authorization, response shape, and error behavior).
- `../../repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence/OrganizationRepositoryTest.java` and/or `MembershipRepositoryTest.java` (modify only if new repository queries are added).

### Tests to write

- **Acceptance criterion 1 — backlog workflow:** an API integration test for authenticated `GET /api/organizations` returns every organization without pagination and includes only `organizationId`, `name`, `description`, status, and role; a test verifies private member/request fields are absent.
- **Acceptance criterion 2 — behavior, authorization, and errors:** API tests verify an authenticated caller receives `200`, an unauthenticated caller is rejected by the real JWT security setup, and repository/service failures are mapped by the existing global error contract where applicable. Add repository integration tests for any new query semantics.
- **Acceptance criterion 3 — actual/effective role:** service unit tests cover NONE → `NONE`, active MEMBER with actual MEMBER/MANAGER role, pending desired MANAGER → `PENDING` plus effective `MEMBER`, pending desired MEMBER → `PENDING` plus effective `NONE`, declined request → `NONE`/`NONE`, and the invariant that a pending desired role is never null.
- Write these tests before implementation, run them to obtain a behavioral red result, then implement and rerun for green; finish with the wider affected-repo suite.

### Risks / open questions

- The current `Membership` scaffold exposes `role` and `status`, while the task requires a pending request's desired role. Confirm the dependency task's persisted field/entity for desired role before implementation; use that existing source rather than adding a speculative duplicate field.
- Confirm the established security test fixture and authenticated-user extraction helper when writing API tests; use the real configured security setup required by project context.
- If the database schema cannot represent multiple historical declined requests, select the current request/membership record according to the dependency task's persistence rules and document that limitation in the design notes.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `OrganizationServiceTest`, `OrganizationControllerStructureTest`, `OrganizationControllerApiTest`.
- **Result:** No test reached execution. The implementer's initial wrapper attempt was blocked creating the default Gradle distribution lock (`FileNotFoundException ... gradle-9.8.0-bin.zip.lck: Access is denied`); its `GRADLE_USER_HOME=.gradle-user` retry stalled during distribution retrieval. With JDK 21 selected explicitly, the focused command `gradlew.bat :Business:test --tests com.ubb.events.implementation.OrganizationServiceTest :RestServices:test --tests com.ubb.events.controllers.OrganizationControllerApiTest --tests com.ubb.events.controllers.OrganizationControllerStructureTest --no-daemon --console=plain` failed before tests with `Could not connect to the Gradle daemon` in the sandbox. The outside-sandbox retry was rejected and did not run. Earlier no-daemon retries stalled without output and were interrupted. No behavioral red result was obtained.
- **Failure reason:** These were Gradle/environment failures before any test execution, not behavioral failures caused by missing functionality. Implementation proceeded without the required behavioral red; this is a TDD process deviation and the red status remains unchecked.

### Green (after implementation)

- **Result:** The user reports that all tests passed after fixing the test issues. The exact test command and output were not provided to the agent; earlier agent attempts failed before test execution, so this result is user-reported and not independently verified here.
- **Full suite of the affected repo:** The user reports all tests passed; the exact full-suite command and output were not provided to the agent.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The backlog specified orgCode, but T6 established Organization has no orgCode. | Use organizationId instead of orgCode in organization responses. | The user's direction is consistent with the existing Organization entity. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions (draft for user review):** Return persisted `organization_id` as `organizationId`; expose only public organization fields and computed status/role. Reuse `Membership.role` and `Membership.status` for effective pending-role mapping established by T9. Add `.gradle-user/` to the backend ignore list as requested.
- **Alternatives considered:** A separate pending-request entity/query was not added because the current dependency model persists pending desired role in `Membership.role`.
- **Limitations:** Tests could not be executed because Gradle failed before test execution (cache permission/retrieval issues followed by a loopback connection failure). The required pre-implementation behavioral red was not obtained, so the TDD sequence is incomplete. The API-level test uses the real Spring Security/JWT filter and MockMvc but stubs the service, so persistence wiring and database behavior remain unverified. T12 is still a todo task and describes a separate `MembershipRequest` entity; this implementation uses the existing `Membership.role` and `Membership.status` fields, following T9's role mapping. Confirm that this is the intended source of pending/declined state during review.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green) — user-reported; command/output not provided
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — `e6adedd` (`T13: Add organization listing API`), branch `T13-view-organizations`
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``
