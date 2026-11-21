# T83 - Membership request list API

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T83
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):**
- **Use case(s):**
- **Workflow:**
- **Drive doc / ADR:**
- **Repo(s):** backend
- **Branch:** `T83-membership-request-list-api`
- **PR link(s):**
- **Tests:** `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java` (`managerCanListAcceptedRequestDetailsWithoutUserId`); tests were not run per the user's explicit instruction.

## Description

Add a separate organization membership-request details API for the Pending, Accepted, and Declined tabs. Keep the existing membership-request API, its service method, and response DTO unchanged so existing consumers are not affected. Preserve existing request creation, acceptance (membership becomes active), and decline (pending membership row is deleted) behavior; add any supporting persistence needed for the details API to retain decision outcomes.

Requested endpoint:

`GET /api/organizations/{organizationId}/membership-requests/details?status=PENDING|ACCEPTED|DECLINED`

Each row should include `requestId`, `firstName`, `lastName`, `email`, optional `avatarUrl`, `requestedAt`, and `status`. This separate response must not include `userId`. The existing endpoint and response fields, including its `ACTIVE` filter, remain unchanged.

## Acceptance criteria

<!-- Proposed from the supplied API reference; confirm or edit. -->

- [ ] The existing membership-request endpoint, service method, response DTO, `PENDING` default, and `PENDING`/`ACTIVE` filtering remain unchanged.
- [ ] A separate manager-only details endpoint filters by `PENDING`, `ACCEPTED`, and `DECLINED`, returning results scoped to the specified organization.
- [ ] Each details response includes `requestId`, `firstName`, `lastName`, `email`, `requestedAt`, and `status`; `avatarUrl` is optional and `userId` is absent.
- [ ] The endpoint follows the backend's organization-scoped authorization and API error-handling conventions.
- [ ] API-level tests cover status filtering and the response fields.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` already exposes a manager-only GET handler with optional `MembershipStatus`; `Business/src/main/java/com/ubb/events/implementation/MembershipServiceImpl.java` defaults the filter to `PENDING` and maps membership/user data; `Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` filters by organization and status. `Model/src/main/java/com/ubb/events/dto/MembershipRequestResponse.java` currently returns membership ID, user ID, role, and status. `Model/src/main/java/com/ubb/events/domain/User.java` has first name, last name, email, and password but no avatar URL.
- **Related completed tasks:** T15 (view organization membership requests), T16 (manage organization membership requests), and T79 (membership requests screen requirements).
- **Constraints / assumptions:** Completed T15/T16 establish persisted membership statuses as `PENDING` and `ACTIVE`; T16 changes accepted requests to `ACTIVE` and deletes declined requests. The details API is separate from the existing API and does not expose `userId`. Supporting history persistence records request outcomes without changing the existing membership state or response contract. `avatarUrl` may be omitted and the user model has no avatar field. Requirement and use case references are not yet identified. The backend working tree has a pre-existing unrelated whitespace change in `Business/src/test/java/com/ubb/events/implementation/EventServiceTest.java`; the user instructed us to continue and leave it intact.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add a request-history persistence record and a separate manager-only `GET /api/organizations/{organizationId}/membership-requests/details` endpoint with its own service method and response DTO. Do not change the existing membership-request list endpoint, service method, or response DTO. The new DTO returns applicant and request details without `userId`. History records retain decision outcomes while existing membership behavior remains unchanged: requests stay `PENDING`, acceptance promotes the membership to `ACTIVE`, and decline deletes the membership row.

This requires a database table/schema change because the production datasource uses Hibernate `ddl-auto=validate` and the repository currently has no migration directory. The implementation must add the project’s agreed schema migration mechanism or provide a reviewed SQL migration artifact before deployment; this is an open deployment concern, not a reason to alter existing membership semantics.

### Steps

1. **Persistence/model:** introduce a membership-request history entity and enum with organization/user references, request timestamp, and `PENDING`/`ACCEPTED`/`DECLINED` statuses; add its repository and schema migration. Keep the existing `Membership` entity, status enum, repository methods, and decision behavior compatible.
2. **Business:** when a request is created, create its history row; when accepted or declined, update that history status in the same transaction before preserving the current membership promotion/deletion logic. Add a separate `listMembershipRequestDetails` service operation using a new list-status enum. Leave the existing `listMembershipRequests` operation unchanged.
3. **API:** add a distinct organization-scoped manager-only `GET /{organizationId}/membership-requests/details` handler. Bind its own status enum, delegate to the new service operation, and return request ID, first name, last name, email, requested-at timestamp, and status, with optional avatar URL and no user ID. Leave the existing GET handler and DTO unchanged. Document success, validation, authentication, and authorization responses.
4. **Error/compatibility:** route invalid status values and malformed identifiers through the existing global error handler; preserve organization authorization and avoid exposing persistence details. Verify existing POST/PATCH behavior remains unchanged apart from recording history.
5. **Traceability:** record exact red/green test commands and outcomes in this task file; do not commit or push until Gate 2 approval.

### Files to create / modify

- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain` — request-history entity and separate details-status enum.
- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto` — new details response DTO without `userId`; preserve existing `MembershipRequestResponse.java` unchanged.
- `../../repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence` — request-history repository and any query projections.
- `../../repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence` — organization/status isolation and timestamp persistence tests.
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MembershipService.java` and `implementation/MembershipServiceImpl.java` — history writes and a separate details-list operation; preserve the existing list operation.
- `../../repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation` — request creation/decision history tests and list mapping/filter tests.
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` — add the separate details route and response contract documentation, leaving the existing route unchanged.
- `../../repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers` — manager authorization, all status filters, response fields, invalid status, and regression tests for existing POST/PATCH endpoints.
- Backend database migration location agreed during implementation (currently not present in the repository) — create the request-history table and indexes.

### Tests to write

- Acceptance criterion 1: regression tests prove the existing API method, response DTO, omitted-status default, `PENDING`/`ACTIVE` filters, POST creation, acceptance, and decline behavior remain unchanged.
- Acceptance criterion 2: repository/service/API tests create pending, accepted, and declined history rows and verify each status filter returns only rows for the requested organization.
- Acceptance criterion 3: API response assertions verify request ID, first name, last name, email, requested-at, and status; verify `userId` is absent and avatar URL may be omitted.
- Acceptance criterion 4: controller tests verify manager-only authorization, unauthenticated/non-manager rejection, malformed IDs, invalid status, and shared error response mapping.
- Acceptance criterion 5: API-level tests exercise the separate details endpoint with each supported status and assert its JSON shape, independently of the existing endpoint.
- Tests are written before implementation and must produce a behavioral red result, then pass after implementation and in the full affected backend suite.

### Risks / open questions

- Production uses `spring.jpa.hibernate.ddl-auto=validate`, but no migration framework is present. A SQL migration artifact was added at `Persistence/src/main/resources/db/migration/V83__create_membership_request_history.sql`; it must be applied before deployment because the application does not run it automatically.
- Existing memberships have no history row or request timestamp. Legacy `PENDING` and `ACTIVE` rows remain visible through the unchanged endpoint. In the details API, legacy pending rows have `requestedAt: null`; legacy active rows are not shown in `ACCEPTED` because they may have been created directly rather than accepted from a request. New request decisions appear in the `ACCEPTED` or `DECLINED` filters.
- Declined rows deleted before T83 cannot be reconstructed. Only declined decisions recorded after this change appear in `DECLINED`; no historical declined user, date, or status is invented.
- `avatarUrl` is not present on `User`; omit it from JSON unless an existing profile/avatar source is identified.
- The task metadata still lacks a requirement, use case, workflow, and Drive/ADR reference.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `MembershipControllerApiTest.managerCanListAcceptedRequestDetailsWithoutUserId` was added after the separate API correction.
- **Attempted command:** `java -version; .\gradlew.bat :RestServices:test --tests com.ubb.events.controllers.MembershipControllerApiTest.managerCanListAcceptedRequestsWithApplicantFields --no-daemon --console=plain`
- **Result:** The command produced no output for over 60 seconds and was interrupted; no test result was returned.
- **Failure reason:** No behavioral red result was obtained. The command was attempted before the separate-API correction; the revised details test was not run. At the user's explicit direction, implementation proceeded without running tests, so the red status remains unchecked.

### Green (after implementation)

- **Result:** Not run at the user's explicit direction.
- **Full suite of the affected repo:** Not run at the user's explicit direction.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | implementation | “the thing is, other pepople osed the api you made changes on, i want to create a separate function in service, and a separate api that retrieves all the data besides the userId so it doesn't interferr with the rest” | Keep the existing API, service method, and response DTO unchanged; add a separate details endpoint, service method, and response DTO without `userId`. | Other people use the existing API, so changes to it could interfere with their consumers. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Added a separate `MembershipRequestHistory` record and `/membership-requests/details` service/API so accepted and declined outcomes can be listed without changing the existing endpoint, service method, response DTO, or Membership lifecycle. The details response includes applicant identity and requested timestamp without `userId`.
- **Alternatives considered:** Persisting decision statuses directly on `Membership` was rejected because it would change existing acceptance/decline behavior. A SQL migration artifact was added because no migration framework exists in the backend.
- **Limitations:** The migration must be applied before production startup with `ddl-auto=validate`. Legacy rows have no history record or request timestamp and therefore return null timestamps; declined rows deleted before this task cannot be reconstructed. Tests were not run at the user's direction.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [ ] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [ ] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``
