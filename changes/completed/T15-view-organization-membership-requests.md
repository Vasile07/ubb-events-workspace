# T15 - View organization membership requests

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T15
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):** ORG-03, ORG-05
- **Use case(s):** UC-05
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T15-view-organization-membership-requests`
- **PR link(s):** 
- **Tests:** `Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java`; `Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`; membership structure tests.

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View organization membership requests” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-03, ORG-05. **Depends on:** Init Membership entity, Organization-scoped authorization, Request organization membership.
- `GET /api/organizations/{organizationId}/membership-requests`, optionally filtered by status (pending by default), for managers only.

Dependencies from the backlog: 7, 9, 12.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `Business/src/main/java/com/ubb/events/MembershipService.java` and `implementation/MembershipServiceImpl.java` are empty scaffolds; `RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` has no handlers; `Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` currently exposes lookup by user and organization IDs. `Membership` model and its status enum define the persisted request data. Organization authorization is implemented in `RestServices/src/main/java/com/ubb/events/security/` and Business authorization service.
- **Related completed tasks:** T7 (membership entity scaffold) and T9 (organization-scoped authorization).
- **Constraints / assumptions:** Backend only; Spring Boot/Java 21 with layered controller, service and repository design. Organization-scoped endpoints use numeric `{organizationId}` and manager authorization via T9. Lists are not paginated. `GET /api/organizations/{organizationId}/membership-requests` and pending-by-default behavior are as stated in this task specification.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the vertical read-only workflow in the backend: expose the organization-scoped GET endpoint from the membership controller, require an authenticated manager through the existing organization authorization interceptor, delegate filtering and organization/request validation to `MembershipService`, and use a repository query that scopes memberships to the organization and requested status. Map persisted memberships to a dedicated response DTO without exposing entities. The existing `/api/memberships` scaffold mapping and empty service contract will be replaced by the task endpoint because the backlog specifies the organization-scoped resource path. The plan keeps pagination and request mutation out of scope.

Approved endpoint contract: each result exposes `membershipId`, `userId`, `role`, and `status`. The optional `status` query parameter uses the persisted enum values (`PENDING`/`ACTIVE`), defaults to `PENDING` when omitted, and returns a validation error for an invalid value.

### Steps

1. **Persistence:** add a repository method that returns memberships by `organizationId` and `status` (with deterministic ordering if the existing conventions require it), and cover organization/status isolation in a JPA test.
2. **Business:** define a membership-request response DTO and a service operation accepting `organizationId` and optional status; default a missing status to `PENDING`, query only that organization/status, map entities to DTOs, and raise the established not-found/validation exceptions where applicable. Keep authorization in the existing organization authorization layer rather than duplicating it in the service.
3. **API:** replace the membership scaffold mapping/handler with `GET /api/organizations/{organizationId}/membership-requests`, bind the optional `status` query parameter, annotate it with `@RequireOrgPermission(MANAGER)`, document 200/400/401/403 responses, and return the service DTO list with HTTP 200.
4. **Error and integration behavior:** ensure malformed organization ids, invalid status values, unauthenticated callers, non-members/members, and non-manager members are mapped through the existing global error/security handlers without leaking persistence details.
5. **Traceability:** update the task test log with the required red/green evidence after implementation; do not add pagination, approval/rejection, or unrelated membership operations.

### Files to create / modify

- `../../repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` — organization/status finder.
- `../../repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java` — finder isolation/status tests.
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MembershipService.java` — list-request operation.
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/MembershipServiceImpl.java` — defaulting, lookup, mapping, and validation.
- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto` — dedicated membership-request response DTO (exact fields require confirmation).
- `../../repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation` — service unit tests for default status, explicit status, empty results, and failures.
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` — endpoint mapping, authorization annotation, parameter binding, OpenAPI responses.
- `../../repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers` — API-level authorization, filtering, validation, and response-shape tests.

### Tests to write

- **Acceptance criterion 1 (backlog workflow):** API test for `GET /api/organizations/{organizationId}/membership-requests` returning the agreed request DTO list for an authorized manager; service/repository tests prove organization scoping and status filtering.
- **Acceptance criterion 2 (behavior, validation, authorization, errors):** API tests for unauthenticated (401), authenticated non-member/member (403), manager (200), omitted status defaulting to pending, explicit `ACTIVE`/`PENDING` filtering, invalid status (400), malformed path id (400), and missing organization (403 through the T9 interceptor). Add a service test for an organization with no matching requests returning an empty list.
- Tests must be written and run before implementation to obtain a behavioral red result, then rerun green and with the affected backend suite per `../../AGENTS.md`.

### Risks / open questions

- **Status semantics:** filters use case-sensitive persisted enum values; omission defaults to `PENDING` and invalid values return 400, as approved in the plan.
- **Organization-not-found status:** preserve T9 behavior: the existing authorization interceptor turns a missing organization into 403 before the service runs. API tests should assert this status.
- **Dependency sequencing:** T12 remains in `../todo`. T15 implements only the GET listing operation; check for overlap when T12 is implemented because both workflows touch membership API and persistence files.
- **Existing scaffold compatibility:** changing `MembershipController`'s root mapping may require updating scaffold structure tests; no other membership endpoints are currently present.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** Written but not run: `Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java`, `Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java`, `Business/src/test/java/com/ubb/events/implementation/MembershipServiceStructureTest.java`, `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerStructureTest.java`, and `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`.
- **Result:** Test execution was attempted with `.\\gradlew.bat :Persistence:test --tests "com.ubb.events.persistence.MembershipRepositoryTest"`, `.\\gradlew.bat :Business:test --tests "com.ubb.events.implementation.MembershipServiceTest" --tests "com.ubb.events.implementation.MembershipServiceStructureTest"`, and `.\\gradlew.bat :RestServices:test --tests "com.ubb.events.controllers.MembershipControllerStructureTest"`. The command produced no test output and stalled before test execution. A retry with `.\\gradlew.bat --offline :Business:test --tests "com.ubb.events.implementation.MembershipServiceTest" --console=plain` also stalled. Direct checks `java -version`, `java --version`, and `cmd /c "java -version"` likewise stalled without output.
- **Failure reason:** Gradle/Java stalled before compilation or test execution, so no behavioral failure was observed. This is an environment/tooling blocker and is not valid red evidence under the TDD workflow. The `Tests written and confirmed failing (red)` status item remains unchecked.

### Green (after implementation)

- **Result:** Not run. The user explicitly asked to continue development and will run the tests because Java/Gradle stalls in this environment.
- **Full suite of the affected repo:** Not run for the same environment reason; no pass is claimed.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | implementation | “the service for instance is not implemented” | Implemented `MembershipServiceImpl` with pending-by-default status filtering, repository lookup, and DTO mapping. | The service implementation is required for the approved GET workflow, so development continued after the correction. |
| 2 | implementation | “invalidStatusIsBadRequest” is not passing. | Added explicit handling for invalid request parameter types as a shared 400 validation response, and strengthened the test to check the error response fields. | An invalid enum query value is a client validation error and must not fall through to the generic 500 handler. |
| 3 | implementation | `MembershipControllerStructureTest` failed. | Updated the expected `GetMapping` value to `/{organizationId}/membership-requests`. | The handler mapping includes the organization id path variable before the membership request segment. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use the approved minimal response fields `membershipId`, `userId`, `role`, and `status`. Missing `status` defaults to `PENDING`; explicit persisted enum values are accepted. Organization authorization remains in the existing interceptor.
- **Alternatives considered:** Exposing `Membership` entities directly was rejected to keep persistence types out of the API. Pagination and mutation endpoints were excluded as outside T15.
- **Limitations:** Tests were written but not executed because Java/Gradle stalls before test execution in this environment; the user will run them. T12 POST behavior remains out of scope.

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
