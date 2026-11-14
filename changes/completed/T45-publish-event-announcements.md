# T45 - Publish event announcements

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T45
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** ORGZ-04
- **Use case(s):** UC-15
- **Workflow:** workflow
- **Drive doc / ADR:** N/A
- **Repo(s):** backend
- **Branch:** `T45-publish-event-announcements`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/46
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/AnnouncementServiceTest.java`; `Business/src/test/java/com/ubb/events/implementation/AnnouncementServiceStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/AnnouncementControllerApiTest.java`.

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Publish event announcements” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORGZ-04. **Depends on:** Organization-scoped authorization, Init Announcement entity.
- `POST /api/organizations/{organizationId}/events/{eventId}/announcements` with a title and content, for members and managers. Returns 201. Refused for a canceled event.
- Deviates from ORGZ-04, which says the responsible manager publishes. UC-15 itself only describes reading.

Dependencies from the backlog: 9, 44.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Announcement.java`; `Persistence/.../AnnouncementRepository.java`; `Business/.../AnnouncementService.java` and `implementation/AnnouncementServiceImpl.java`; `RestServices/.../controllers/AnnouncementController.java` (currently empty scaffold); `MessageController.java` and `MessageServiceImpl.java` for organization/event endpoint and ownership checks; `OrganizationAuthorizationInterceptor.java` and `RequireOrgPermission.java` for member authorization; Event entity/status and event workflow services for canceled status handling.
- **Related completed tasks:** `changes/completed/T44-init-announcement-entity.md` (announcement fields/scaffold); `T41-view-event-messages.md` (organization/event route and authorization); `T28-publish-event.md` (event publication/status behavior); `T9-organization-scoped-authorization.md` (membership authorization).
- **Constraints / assumptions:** Backend only, workflow task; Java 21, Spring Boot, JPA, JUnit 5, Gradle wrapper; keep validation and canceled-event business rules in the service, authorization in the organization interceptor, require nonblank title and content without length limits for now, and return 201. Backend repo was clean on `T42-send-event-message` and had no unpushed commits; switched to local `master`. Fetch/pull failed due a local DNS/thread startup error (`getaddrinfo() thread failed to start`), the user confirmed local `master` is current because T42 was just pushed and merged. Workspace task file was moved to `changes/active`; `git mv` could not write `.git/index.lock`, so it was moved with `Move-Item`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the announcement publication vertical slice in the backend by replacing the T44 controller scaffold with an organization and event scoped `POST` endpoint. The controller will validate a request containing nonblank `title` and `content`, require the existing member-level organization permission (which also allows managers), and return `201 Created`. The service will load the event through the organization-scoped repository query, reject canceled events with the shared conflict error, persist the announcement, and map it to a response. This keeps authorization in the existing interceptor, business rules in the service, and database access in the repository. No title/content length limits will be introduced yet.

The response will expose the persisted announcement identity, event identity, title, and content so the newly created resource can be confirmed by the caller. A separate manager-only permission or author field is not planned because the backlog explicitly allows members and managers and the T44 schema has no author column.

### Steps

1. **Tests first (Model/Business/RestServices):** add request/response and service/API tests for a successful member publication and manager publication, nonblank validation for both fields, organization/event lookup and not-found behavior, canceled-event rejection, unauthenticated and non-member authorization, `201 Created`, and response mapping. Run the focused tests before implementation and record a behavioral red result.
2. **API contracts (Model):** add a create request DTO with `@NotBlank` title and content fields and an announcement response DTO containing `id`, `eventId`, `title`, and `content`.
3. **Business/persistence (Business/Persistence):** add a create operation to `AnnouncementService` and implement it transactionally. Resolve the event with the organization-scoped repository method, throw `NotFoundException` when it is absent, throw `ConflictException` for `EventStatus.CANCELED`, save the `Announcement`, and map the saved entity to the response. Add only the repository query needed to enforce event ownership; keep rules out of persistence.
4. **REST (RestServices):** replace the scaffold route with `POST /api/organizations/{organizationId}/events/{eventId}/announcements`, annotate it with the existing default `@RequireOrgPermission` (member and manager access), apply `@Valid`, return `ResponseEntity` with HTTP 201, and document 201, 400, 401, 403, 404, and 409 responses.
5. **Verification:** rerun the focused tests to green, then the full backend test suite, and record exact commands, results, and any unrelated failures in the task Test log.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/CreateAnnouncementRequest.java` (create).
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/AnnouncementResponse.java` (create).
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/EventRepository.java` (add or reuse organization-scoped event lookup as needed) and `AnnouncementRepository.java` only if a save/query helper is required.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/AnnouncementService.java` and `implementation/AnnouncementServiceImpl.java` (create operation and rules).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/AnnouncementController.java` (authorized POST endpoint and OpenAPI responses).
- Corresponding tests under the existing Model, Business, Persistence, and RestServices test source sets.

### Tests to write

- Acceptance criterion 1: API happy-path tests prove a member and a manager can publish an announcement and receive `201` with the saved `id`, `eventId`, `title`, and `content`; a service test proves the announcement is attached to the requested event.
- Acceptance criterion 2: tests cover blank/whitespace title, blank/whitespace content, missing event, event from another organization, canceled event (`409`), unauthenticated (`401`), and non-member (`403`) requests, plus controller structure/OpenAPI mapping where existing patterns provide those tests.

### Risks / open questions

- The backlog does not prescribe a response JSON shape. The plan proposes `{id, eventId, title, content}`, matching the persisted T44 fields and existing workflow response DTOs; confirm or adjust this during Gate 1 if the team has a different API contract.
- The backlog says members and managers may publish, so the endpoint uses the interceptor's default `MEMBER` threshold. The responsible-manager wording in ORGZ-04 is treated as the documented deviation in the task description.
- The T44 scaffold controller currently uses `/api/announcements`; implementation will replace it with the organization/event path specified by T45.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/AnnouncementServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/AnnouncementControllerApiTest.java`.
- **Result:** `.\\gradlew.bat :Business:test --tests com.ubb.events.implementation.AnnouncementServiceTest :RestServices:test --tests com.ubb.events.controllers.AnnouncementControllerApiTest --rerun-tasks` compiled and ran the Business tests; all 3 service tests failed (`createsAnnouncementForOrganizationEvent` with `NullPointerException`, `rejectsCanceledEvent`, and `rejectsMissingEvent` with assertion failures).
- **Failure reason:** The minimum compile stub returned `null` and did not inspect the event or persist an announcement. Compilation and test setup succeeded; the failures demonstrate the missing publication behavior.

- **Supplemental coverage after implementation:** Added manager publication, unauthenticated 401, non-member 403, independent blank-title and blank-content 400 checks, full response title/content assertions, and verification that the service passes both event and organization IDs to the scoped lookup. These tests were added after the initial implementation and are supplemental coverage, not pre-implementation red evidence.

### Green (after implementation)

- **Result:** Focused command `.\\gradlew.bat :Business:test --tests com.ubb.events.implementation.AnnouncementServiceTest :RestServices:test --tests com.ubb.events.controllers.AnnouncementControllerApiTest --rerun-tasks` passed with `BUILD SUCCESSFUL`; the three service tests and six API tests passed, including the supplemental coverage described above.
- **Full suite of the affected repo:** `.\\gradlew.bat test --rerun-tasks` ran 126 Business tests and stopped with two pre-existing `OrganizationServiceStructureTest` failures (`serviceInterfaceIsEmpty` and `implementationIsAnnotatedAndImplementsServiceWithoutOperations`), because those assertions expect the already-implemented Organization service to remain an empty scaffold. T45 tests passed; downstream module tests were not reached in this run.

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

- **Decisions:** Replaced the T44 announcement controller/service scaffold with an organization/event scoped POST endpoint. Members and managers use the existing member permission threshold. Title and content are validated as nonblank, canceled events return 409, and successful responses contain `id`, `eventId`, `title`, and `content`.
- **Alternatives considered:** Kept the business rules in `AnnouncementServiceImpl` and reused `EventRepository.findByIdAndOrganizationId`; a separate author field or manager-only permission was not added because the approved task allows members and managers and the entity has no author column.
- **Limitations:** The full backend suite stopped on two existing Organization service structure assertions; downstream module tests were not reached. The task does not add title/content length limits, per the approved plan.

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
