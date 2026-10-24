# T17 - View organization members

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T17
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):** ORG-04, ORG-05
- **Use case(s):** UC-16
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T17-view-organization-members`
- **PR link(s):** 
- **Tests:** `Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java`; `Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerStructureTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View organization members” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-04, ORG-05. **Depends on:** Init Membership entity, Organization-scoped authorization.
- `GET /api/organizations/{organizationId}/members`: the members with their roles, for members and managers. Not public data.

Dependencies from the backlog: 7, 9.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Membership.java` and `MembershipStatus.java` define persisted membership data; `Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` has existing organization lookup methods; `Business/src/main/java/com/ubb/events/MembershipService.java` and `implementation/MembershipServiceImpl.java` implement listing and decision workflows; `RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` exposes the membership request endpoints; `RestServices/src/main/java/com/ubb/events/security/RequireOrgPermission.java` and `OrganizationAuthorizationInterceptor.java` enforce organization-scoped roles. `Model/src/main/java/com/ubb/events/dto/UserResponse.java` defines public personal fields (`id`, `firstName`, `lastName`, `email`) and deliberately omits password. Existing API tests are in `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`.
- **Related completed tasks:** `T7-init-membership-entity.md` (Membership persistence model), `T9-organization-scoped-authorization.md` (organization authorization), `T10-view-my-profile.md` (public personal data response fields), `T15-view-organization-membership-requests.md` and `T16-manage-organization-membership-requests.md` (adjacent membership workflows).
- **Constraints / assumptions:** Backend only; Java 21, Spring Boot, JPA, JUnit 5, and Gradle wrapper are established in `../../context/project-context.md`. Organization IDs are numeric path variables and membership endpoints use `@RequireOrgPermission`. Backend repo is clean and synchronized with `origin/master`. User clarification: return only `ACTIVE` memberships; each member entry includes the user's personal data in addition to the membership role. Reuse the established public user fields from `UserResponse` and exclude password data.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the vertical read workflow through the existing membership controller, service, and repository layers. Add a dedicated response DTO that combines the established public `UserResponse` fields (`id`, `firstName`, `lastName`, `email`) with the membership `role`; the password is never mapped or serialized. Query memberships by the requested organization and `ACTIVE` status so organization scoping and filtering remain in persistence, while the service maps entities to the response. Protect the endpoint with the existing organization interceptor at the default `MEMBER` level, allowing both members and managers. Reuse the existing membership request endpoint and DTOs rather than changing their contract.

### Steps

1. **Persistence:** add or adapt a repository query that returns memberships for one organization with `MembershipStatus.ACTIVE`, and add repository integration coverage proving memberships from other organizations and non-active memberships are excluded.
2. **Business/service:** introduce the members-list operation and response DTO mapping. Map each active membership to the user's public fields plus role, preserve the repository result as a list (including an empty list), and avoid exposing the `User.password` field. Add service unit tests for organization scoping, ACTIVE-only behavior, role mapping, personal-field mapping, and empty results.
3. **API/security:** add `GET /api/organizations/{organizationId}/members` to `MembershipController`, annotate it with `@RequireOrgPermission` (default `MEMBER`), and document success, authentication, forbidden, and malformed-identifier responses. Keep the numeric organization path variable and pass it unchanged to the service.
4. **Workflow verification:** run the focused persistence, service, and controller tests, then the affected backend suite; record red/green evidence in this task before implementation review.

### Files to create / modify

- `../../repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` (active organization query, if the existing derived method is not sufficient).
- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/OrganizationMemberResponse.java` (or the agreed equivalent dedicated DTO containing `UserResponse` public fields and `role`).
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MembershipService.java` and `implementation/MembershipServiceImpl.java` (members-list contract and mapping).
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` (GET members handler and OpenAPI annotations).
- Corresponding backend tests under `Persistence/src/test/java`, `Business/src/test/java`, and `RestServices/src/test/java`.

### Tests to write

- **Acceptance criterion: backlog workflow is implemented:** API integration test calls `GET /api/organizations/7/members` as an authenticated active member and asserts `200` with each member's `id`, `firstName`, `lastName`, `email`, and `role`.
- **Acceptance criterion: behavior, validation, authorization, and errors are covered:**
  - persistence test verifies the organization-and-`ACTIVE` query excludes `PENDING` memberships and memberships belonging to another organization;
  - service test verifies mapping of both `MEMBER` and `MANAGER` roles, all four `UserResponse` fields, an empty result, and that no password property is present in the response object/JSON contract;
  - API tests verify unauthenticated requests return `401`, non-members return `403`, active members are allowed (and managers are allowed through the same `MEMBER` minimum), malformed organization identifiers return `400`, and the organization id is passed to the service so results cannot cross organization scope.

### Risks / open questions

- The existing repository already exposes `findAllByOrganizationIdAndStatus`; prefer reusing it with `MembershipStatus.ACTIVE` unless a dedicated name improves clarity. The exact DTO nesting/field shape should follow the established `UserResponse` JSON naming while keeping `role` alongside the personal fields.
- The endpoint must use the current numeric `{organizationId}` convention and the existing interceptor; no new authorization mechanism or pagination is in scope.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** Added repository, service, controller API, and controller structure coverage for active member listing before implementation work.
- **Result:** Not run, per the user's explicit instruction: "you can create the tests but do not try to run them, just fully code".
- **Failure reason:** No behavioral red result is available because test execution was intentionally skipped. The red checklist item remains unchecked.

### Green (after implementation)

- **Result:** Not run, per the user's explicit instruction. The implementation is handed over for review without claiming passing tests.
- **Full suite of the affected repo:** Not run for the same reason.

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

- **Decisions:** Reused `findAllByOrganizationIdAndStatus` with `MembershipStatus.ACTIVE`, keeping organization filtering and status filtering in persistence. Added `OrganizationMemberResponse` with the four public user fields and membership role; password is not part of the DTO. The controller uses the existing default `MEMBER` organization permission so active members and managers can access the list.
- **Alternatives considered:** A dedicated repository query name was considered unnecessary because the existing derived query already expresses organization and status filtering. Nesting `UserResponse` was avoided to keep `role` alongside the established user response fields in the endpoint payload.
- **Limitations:** Tests were written but not executed by explicit user instruction, so compilation and behavioral results remain to be verified. No pagination was added because it is outside the approved scope.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [ ] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``
