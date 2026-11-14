# T46 - View event announcements

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T46
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** ORGZ-04
- **Use case(s):** UC-15
- **Workflow:** workflow
- **Drive doc / ADR:** N/A
- **Repo(s):** backend
- **Branch:** `T46-view-event-announcements`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/51
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/AnnouncementServiceTest.java`; `Persistence/src/test/java/com/ubb/events/persistence/AnnouncementRepositoryTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/AnnouncementControllerApiTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View event announcements” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORGZ-04. **Depends on:** Init Membership entity, Register for event, Init Announcement entity.
- `GET /api/events/{eventId}/announcements`, outside the organization path. The service allows registered participants and members or managers of the organizing organization, and denies everyone else.
- A cancelled registration loses access. "No announcements" returns an empty list. Each announcement shows content, author and creation date.

Dependencies from the backlog: 7, 32, 44.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Announcement.java` and `dto/AnnouncementResponse.java`; `Business/src/main/java/com/ubb/events/implementation/AnnouncementServiceImpl.java`; `RestServices/src/main/java/com/ubb/events/controllers/AnnouncementController.java`; `Persistence/.../RegistrationRepository.java` and `MembershipRepository.java`; `RestServices/.../security/RequireOrgPermission.java`. Existing Announcement endpoint currently supports only T45 publication (`POST` on the organization/event path).
- **Related completed tasks:** `T7-init-membership-entity.md`, `T32-register-for-event.md`, `T44-init-announcement-entity.md`, and `T45-publish-event-announcements.md`.
- **Constraints / assumptions:** Backend only; Java 21, Spring Boot, JPA, JUnit 5, Gradle wrapper; task requires participant and organization membership authorization and denies canceled registrations. Following the user's clarification, retain the existing Announcement schema and omit author and creation date from the response. Backend is clean on updated `master` at `ce0fed1`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the read workflow as a public event route with authenticated, service-level access evaluation. The controller will expose `GET /api/events/{eventId}/announcements` and pass the authenticated user ID to `AnnouncementService`. The service will load the event, then allow access when the user has an active membership in the organizing organization (member or manager) or an existing registration row for the event. Because T32 cancellation deletes the registration row, a canceled registration automatically loses participant access. Users who satisfy neither condition receive the shared forbidden error; an event with no announcements returns an empty list. Responses will use the existing announcement fields (`id`, `eventId`, `title`, `content`) and will omit author and creation date, following the user's schema clarification.

The route cannot use the organization interceptor because its URL intentionally has no organization path and must support two alternative access paths. Authorization therefore remains in the business service, with repositories limited to event, membership, registration, and announcement data access.

### Steps

1. **Tests first (Business/RestServices/Persistence):** add service and API tests for an active participant, active organization member, manager, empty announcements, canceled/deleted registration, non-member/non-participant forbidden access, unauthenticated access, unknown event, and response shape without author/date fields. Run focused tests and record a behavioral red result before implementation.
2. **Model / API contract:** reuse `AnnouncementResponse` and extend the announcement service contract for listing by event and authenticated user; keep the response limited to the current T44 schema.
3. **Persistence:** add repository methods needed to list announcements by event in deterministic order and to determine active access (`RegistrationRepository` event/user existence and `MembershipRepository` user/organization lookup). Keep persistence free of authorization decisions.
4. **Business:** implement the list operation transactionally or read-only. Load the event or raise the shared not-found error, evaluate active organization membership (member or manager) and registration access, reject users with neither using the shared forbidden error, query announcements, and map each entity to `AnnouncementResponse`. A missing announcement result must map to an empty list.
5. **REST:** replace/add the announcement GET handler at `/api/events/{eventId}/announcements`, require authentication through the existing security configuration, pass the authenticated principal ID, and document 200, 401, 403, and 404 responses. Leave the existing T45 publication endpoint intact.
6. **Verification:** run focused module tests to green, then the full backend Gradle suite, and record exact commands and any pre-existing failures in the task log.

### Files to create / modify

- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/AnnouncementService.java` and `implementation/AnnouncementServiceImpl.java` — add the authenticated-user announcement listing contract and access rules.
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/AnnouncementRepository.java` — add event-scoped list query; update `RegistrationRepository.java` and/or `MembershipRepository.java` only for the access checks required by the service.
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/AnnouncementController.java` — add the unaffiliated event GET route while preserving publication.
- Corresponding `Business`, `Persistence`, and `RestServices` tests under `src/test/java`.

### Tests to write

- **Acceptance criterion 1 / happy path:** API-level tests prove a registered participant, active organization member, and active manager receive HTTP 200 and announcement data containing only `id`, `eventId`, `title`, and `content`.
- **Acceptance criterion 1 / list behavior:** service and persistence tests prove announcements are scoped to the requested event, returned in the chosen stable order, and an event with no rows returns `[]`.
- **Acceptance criterion 2 / authorization and errors:** tests prove a canceled registration represented by a deleted row is denied, a user who is neither participant nor active member is denied with 403, unauthenticated access returns 401, and an unknown event returns 404. Include structure/OpenAPI coverage for the exact route and documented responses.
- Follow the required TDD sequence: tests and compile stubs, behavioral red result, implementation, focused green tests, then the full backend suite.

### Risks / open questions

- The existing announcement entity has no author or creation date. Per the user's clarification, those fields remain absent from the response and no schema migration is planned.
- Authorization is service-level because the endpoint has no `organizationId` path variable and must allow either participant or organization membership. The implementation should treat only active membership as member access and rely on the existing deleted-row cancellation semantics for participant access.
- The existing T45 POST endpoint and its organization interceptor authorization must remain unchanged except for any shared service wiring needed by the new GET operation.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `AnnouncementServiceTest` list-access tests and `AnnouncementControllerApiTest.registeredParticipantCanViewAnnouncements`.
- **Result:** `./gradlew.bat :Business:test --tests com.ubb.events.implementation.AnnouncementServiceTest --rerun-tasks` compiled and ran 6 tests; `listsAnnouncementsForRegisteredParticipant` and `deniesUserWithoutMembershipOrRegistration` failed because the list stub returned an empty list and did not enforce access. `./gradlew.bat :RestServices:test --tests com.ubb.events.controllers.AnnouncementControllerApiTest --rerun-tasks` ran 8 tests; the new registered-participant GET test failed because the GET route was not implemented.
- **Failure reason:** These were behavioral failures after successful compilation and test setup, demonstrating the missing announcement read behavior.

### Green (after implementation)

- **Result:** Focused command `./gradlew.bat :Business:test --tests com.ubb.events.implementation.AnnouncementServiceTest :Persistence:test --tests com.ubb.events.persistence.AnnouncementRepositoryTest :RestServices:test --tests com.ubb.events.controllers.AnnouncementControllerApiTest --rerun-tasks` completed with `BUILD SUCCESSFUL`; T46 Business, Persistence, and REST tests passed, including manager access, event scoping/order, empty results, unknown event, and response field assertions.
- **Supplemental coverage:** Manager access, pending-membership denial, repository event scoping/order, empty and unknown-event API cases, and serialization field assertions were added after the original red run. They are supplemental post-implementation coverage and are not presented as pre-implementation red evidence.
- **Full suite of the affected repo:** `./gradlew.bat test --rerun-tasks` ran 152 tests and stopped on two pre-existing `OrganizationServiceStructureTest` failures (`serviceInterfaceIsEmpty` and `implementationIsAnnotatedAndImplementsServiceWithoutOperations`). T46 tests passed; the failure is unrelated to this task.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “Follow the existing schema and omit them so that it doesn't have any conflicts later on” | Author and creation date were omitted from the announcement response and no schema migration was added. | The existing announcement schema has no author or creation date fields, so the workflow follows that schema. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Added `GET /api/events/{eventId}/announcements`; service authorization accepts an existing registration or ACTIVE organization membership (member/manager), returns an empty list when no announcements exist, and maps only the existing announcement schema fields. Kept T45 publication unchanged.
- **Alternatives considered:** Service-level authorization was used because the route has no organization path and supports two access paths; the organization interceptor was not suitable. Author and creation date were intentionally omitted per the user's schema decision.
- **Limitations:** The full backend suite remains blocked by two pre-existing Organization service structure assertions. Tests cover the endpoint and core service paths; database integration coverage was not added because existing task test patterns use unit and MockMvc coverage for this workflow.

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
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
