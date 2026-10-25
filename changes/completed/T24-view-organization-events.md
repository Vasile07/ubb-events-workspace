# T24 - View organization events

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T24
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** EVT-01, EVT-02
- **Use case(s):** none (supports UC-06, UC-07)
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T24-view-organization-events`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/25
- **Tests:** `EventControllerApiTest`, `EventServiceTest` (new); `EventRepositoryTest`, `EventServiceStructureTest`, `EventControllerStructureTest` (updated)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View organization events” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** EVT-01, EVT-02. **Use case:** none, needed by every organization-side use case. **Depends on:** Organization-scoped authorization, Create an event with tasks.
- `GET /api/organizations/{organizationId}/events` and `.../{eventId}`, for members and managers, returning all events of the organization including drafts and CANCELED ones.
- The event data only: the task list is a separate endpoint.

Dependencies from the backlog: 9, 23.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code (backend):**
  - Organization-scoped endpoint pattern:
    - `RestServices/.../controllers/MembershipController.java` (`@RequestMapping("/api/organizations")`, handlers on `/{organizationId}/...`, `@RequireOrgPermission`, OpenAPI `@Operation`/`@ApiResponses`);
    - `security/OrganizationAuthorizationInterceptor.java` and `OrganizationAuthorizationWebConfig.java` (interceptor on `/api/organizations/{organizationId}/**`; the user id comes from the JWT principal; 401/403 are written as `ErrorResponse`).
  - Service and DTO pattern:
    - `Business/.../implementation/MembershipServiceImpl.java` (constructor-injected repository, stream mapping to DTOs, `NotFoundException` from `findByIdAndOrganizationId`);
    - DTOs such as `Model/.../dto/OrganizationMemberResponse.java` (`final` class with `@Getter @AllArgsConstructor @EqualsAndHashCode`).
  - Code to extend: `Model/.../domain/Event.java` and `EventStatus.java` (T19), `Persistence/.../EventRepository.java`, and the empty `EventService`/`EventServiceImpl`/`EventController` scaffolds.
  - Test patterns:
    - `MembershipControllerApiTest` (`@SpringBootTest` + MockMvc, real JWT via `JwtTokenService`, `@MockitoBean` service and `OrganizationAuthorizationService`);
    - `MembershipServiceTest` (Mockito);
    - `EventRepositoryTest` (`@DataJpaTest`).
- **Related completed tasks:** T9 (organization-scoped authorization), T19 (Event entity), T13–T18 (organization and membership endpoints that set the current API and test patterns).
- **Constraints / assumptions:**
  - **Dependency decision (user):** T23 (Create an event with tasks, owner Kentsch Karina) and T22 (Event lifecycle model, no owner) are both still in `todo/` with no code. The user decided to implement T24 now and infer the shape of T23 from its task file and the existing code.
  - **Path variable:** the code uses `{organizationId}` (a `Long`), as do T23's spec and T13–T18. `context/project-context.md` still says `{orgCode}`, which looks outdated.
  - **Pre-existing failure on `master`:** `.\gradlew.bat clean test` already fails before any T24 change. `OrganizationServiceStructureTest` fails 2 tests because it still expects an empty `OrganizationService` after T13/T14 added methods. Business fails first, so Persistence and RestServices don't run.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add two read-only, organization-scoped endpoints, following the T13–T18 patterns:

- `GET /api/organizations/{organizationId}/events` returns all events of the organization, including `DRAFT` and `CANCELED`.
- `GET /api/organizations/{organizationId}/events/{eventId}` returns one event of that organization.

Both are for active members and managers (`@RequireOrgPermission`, minimum role MEMBER, which managers also satisfy). They return event data only, with no task list (backlog: "the task list is a separate endpoint").

**Inferred from T23:** T23 will add `POST /api/organizations/{organizationId}/events` (managers, creates a draft, returns the id and draft status) to the same event controller and service. So T24:

- moves `EventController` to the `/api/organizations` root, like `MembershipController`;
- introduces a reusable `EventResponse` DTO that maps the Event fields from T19;
- adds read methods to `EventService`.

T23 can then add its POST handler and create method next to them without restructuring. The empty scaffolds (`/api/events` with no handlers, service with no methods) become obsolete and are replaced, as `project-context.md` requires for workflow tasks.

**Status without T22:** T22 has not decided derived statuses (COMPLETED) yet, so the response returns the **stored** `status` (`DRAFT`/`PUBLISHED`/`CANCELED`) unchanged. T22 can later compute derived values in the service mapping without changing the endpoints.

**Alternatives considered:**
- A separate controller for organization events: rejected, because T23's POST belongs with the same resource and `MembershipController` already shows the "one controller per resource under `/api/organizations`" style.
- Returning 403 for an event of another organization: rejected in favour of 404, the same as `findByIdAndOrganizationId` → `NotFoundException` in `MembershipServiceImpl`. This doesn't reveal that the event exists.
- Waiting for T23: rejected by the user.

### Steps

1. **Model (DTO):** add `dto/EventResponse`, a `final` class with `@Getter @AllArgsConstructor @EqualsAndHashCode` and the fields `id`, `organizationId`, `categoryId`, `title`, `description`, `date` (`LocalDate`), `startTime`, `endTime` (`LocalTime`), `location`, `capacity`, `status` (`EventStatus`).
2. **Persistence:** add to `EventRepository`:
   - `List<Event> findAllByOrganizationIdOrderByDateAscStartTimeAsc(Long organizationId)`;
   - `Optional<Event> findByIdAndOrganizationId(Long id, Long organizationId)`.
3. **Business:** add to `EventService`:
   - `List<EventResponse> listOrganizationEvents(Long organizationId)`;
   - `EventResponse getOrganizationEvent(Long organizationId, Long eventId)`.

   `EventServiceImpl` gets `EventRepository` through its constructor and maps entities to `EventResponse`. A missing event, or one that belongs to another organization, throws `NotFoundException("Event not found")` (→ 404). There is no status filtering, so drafts and canceled events are included.
4. **API:** change `EventController` to `@RequestMapping("/api/organizations")` and add:
   - `listOrganizationEvents`: `GET /{organizationId}/events`;
   - `getOrganizationEvent`: `GET /{organizationId}/events/{eventId}`.

   Both use `@RequireOrgPermission` (MEMBER) and OpenAPI `@Operation` with `@ApiResponses` for 200, 400, 401, 403 and 404 (404 only on the detail endpoint). Authorization stays in the interceptor; the controller only maps the request.
5. **Obsolete scaffold tests:** update `EventServiceStructureTest` and `EventControllerStructureTest` from "no operations, `/api/events`" to the new structure: the root mapping, the handler paths, and the `@RequireOrgPermission` annotations.
6. **Verification:** write the tests first and get a behavioral red against stubs, then implement and get the focused tests green. Then run the full suite with `--continue`, so every module runs despite the pre-existing `OrganizationServiceStructureTest` failure, and record exactly which failures are pre-existing.

### Files to create / modify

Backend (`repos/ubb-events-backend/`):

- Create `Model/src/main/java/com/ubb/events/dto/EventResponse.java`
- Modify `Persistence/src/main/java/com/ubb/events/persistence/EventRepository.java`
- Modify `Persistence/src/test/java/com/ubb/events/persistence/EventRepositoryTest.java`
- Modify `Business/src/main/java/com/ubb/events/EventService.java`, `Business/src/main/java/com/ubb/events/implementation/EventServiceImpl.java`
- Create `Business/src/test/java/com/ubb/events/implementation/EventServiceTest.java`
- Modify `Business/src/test/java/com/ubb/events/implementation/EventServiceStructureTest.java`
- Modify `RestServices/src/main/java/com/ubb/events/controllers/EventController.java`
- Create `RestServices/src/test/java/com/ubb/events/controllers/EventControllerApiTest.java`
- Modify `RestServices/src/test/java/com/ubb/events/controllers/EventControllerStructureTest.java`

### Tests to write

Acceptance criteria → tests:

- **AC1: endpoints per the backlog (list and detail, members and managers, all statuses, event data only):**
  - `EventControllerApiTest.memberReceivesAllOrganizationEvents`: 200; the JSON has every `EventResponse` field; the list includes `DRAFT`, `PUBLISHED` and `CANCELED` events; there is no `tasks` field.
  - `EventControllerApiTest.memberReceivesSingleOrganizationEvent`: 200 with the event fields and no `tasks` field.
  - `EventControllerApiTest.managerCanViewOrganizationEvents`: an authorization check at MEMBER level that a manager passes. It uses the same `hasAtLeast(..., MEMBER)` stub that the interceptor calls.
  - `EventServiceTest.listsAllEventsOfOrganizationIncludingDraftAndCanceled`: the mapping of every field and no filtering by status.
  - `EventServiceTest.returnsEmptyListWhenOrganizationHasNoEvents`.
  - `EventServiceTest.returnsEventOfOrganization`.
  - `EventRepositoryTest.findsOnlyEventsOfOrganizationOrderedByDateAndStartTime`.
  - `EventRepositoryTest.findsEventByIdOnlyWithinItsOrganization`.
- **AC2: authorization and error cases:**
  - `EventControllerApiTest.unauthenticatedRequestIsRejected`: 401 on both endpoints.
  - `EventControllerApiTest.nonMemberIsForbidden`: 403 with an `ErrorResponse` body.
  - `EventControllerApiTest.unknownEventReturnsNotFound`: 404 with `status` 404 and `message` "Event not found".
  - `EventServiceTest.throwsNotFoundForMissingEventOrEventOfAnotherOrganization`.
- **Structure tests:** `EventServiceStructureTest` and `EventControllerStructureTest` are updated as in step 5.
- **Expected red:**
  - `EventResponse` is complete, the repository methods are declared, and the service methods return an empty list or `null`.
  - The controller handlers exist without mapping annotations, so the requests get 404 or 403 from the missing mapping.
  - Failures are assertion failures, not compilation errors.

### Risks / open questions

User decisions (confirmed before Gate 1):

1. **List order:** chronological, by `date` then `startTime` ascending. **Confirmed by the user.**
2. **Event of another organization or unknown id:** **404** "Event not found". **Confirmed by the user.**
3. **Status:** the stored value (`DRAFT`/`PUBLISHED`/`CANCELED`) is returned as-is until T22 defines derived statuses such as COMPLETED. **Confirmed by the user.**
4. **`EventResponse` fields:** all Event fields, with `organizationId` instead of the nested organization. T23's "registration information" is undefined and not in the schema, so it's not included.
5. **Coordination with T23 (Karina):** T24 restructures `EventController`/`EventService` and adds `EventResponse`. T23 should build on these. Share this PR with Karina so she can reuse `EventResponse` for the POST response, or reply if she plans a different shape.
6. **Pre-existing failing test on `master`:** `OrganizationServiceStructureTest` (from T13/T14) is outside T24's scope. **User decision:** report it to the team (owners of T13/T14); it is not fixed in T24.
7. **`project-context.md` mentions `{orgCode}`:** the real convention is `{organizationId}`. Agents don't edit documentation, so the team should update it.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventRepositoryTest` (2 new), `EventServiceTest` (new), `EventServiceStructureTest` (updated), `EventControllerApiTest` (new), `EventControllerStructureTest` (updated).
- **Stubs used:**
  - complete `EventResponse` DTO;
  - `EventRepository` query methods declared;
  - `EventServiceImpl` with `@Service` and a constructor-injected repository, but `listOrganizationEvents` returns `List.of()` and `getOrganizationEvent` returns `null` without querying;
  - `EventController` still at `/api/events`, with handler methods that have no `@GetMapping` and no `@RequireOrgPermission`.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Persistence:test --tests "com.ubb.events.persistence.EventRepositoryTest" :Business:test --tests "com.ubb.events.implementation.EventServiceTest" --tests "com.ubb.events.implementation.EventServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.EventControllerApiTest" --tests "com.ubb.events.controllers.EventControllerStructureTest"`
- **Result:** `BUILD FAILED`, everything compiled, 12 of 20 tests failed.
  - `EventServiceTest`, 4 of 4 failed:
    - `listsAllEventsOfOrganizationIncludingDraftAndCanceled` (`AssertionFailedError` at `:45`, empty list);
    - `returnsEmptyListWhenOrganizationHasNoEvents` (Mockito `verify`: the repository was never queried);
    - `returnsEventOfOrganization` (`:69`, null);
    - `throwsNotFoundForMissingEventOrEventOfAnotherOrganization` (`:77`, no exception).
  - `EventControllerApiTest`, 5 of 6 failed:
    - `nonMemberIsForbidden`, `memberReceivesAllOrganizationEvents`, `managerCanViewOrganizationEvents`, `memberReceivesSingleOrganizationEvent`, `unknownEventReturnsNotFound` (`AssertionError` on the status code, because no handler is mapped for `/api/organizations/{organizationId}/events...`).
  - `EventControllerStructureTest`, 3 of 3 failed (`AssertionFailedError`: root mapping `/api/events`, no `@GetMapping`/`@RequireOrgPermission`).
- **Passing at red, as expected:**
  - `EventRepositoryTest` (5 of 5, including the 2 new query tests): Spring Data derives the queries from the method names, so declaring the methods is their implementation. There is no non-trivial stub that would fail them meaningfully.
  - `EventServiceStructureTest` (2 of 2): the stub already had the planned structure.
  - `EventControllerApiTest.unauthenticatedRequestIsRejected`: the existing security config already returns 401 for `/api/**`.
- **Failure reason:** behavioral. The failures come from the missing planned functionality: no repository query or mapping in the service, no `NotFoundException`, no organization-scoped GET mappings, and no authorization annotations. There were no compilation or setup errors.

### Green (after implementation)

- **Result:** the same focused command gives `BUILD SUCCESSFUL`, and all 20 focused tests pass (`EventRepositoryTest` 5, `EventServiceTest` 4, `EventServiceStructureTest` 2, `EventControllerApiTest` 6, `EventControllerStructureTest` 3).
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives `BUILD FAILED`: 175 tests, 3 failures, 0 errors, 0 skipped. All 3 failures are **pre-existing** and unrelated to T24:
  - `OrganizationServiceStructureTest.serviceInterfaceIsEmpty` and `.implementationIsAnnotatedAndImplementsServiceWithoutOperations`: they still assert an empty `OrganizationService` after T13/T14 added methods. They also fail on `master` before T24, as checked during planning.
  - `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`: "OrganizationController.list lacks org authorization" (T13's list handler has neither `@RequireOrgPermission` nor `@NoOrgCheck`). It was not visible on `master` because `:Business:test` fails first and Gradle stops before RestServices. T24 does not modify `OrganizationController` or this test (`git diff master --stat` only touches event files), and `EventController` is not among the controllers it checks.

  All T24 tests and every other test pass.

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

<!-- DRAFT written by the implementer. Verify and correct before using it as a study note. -->
- **Decisions:**
  - Two read-only endpoints under `/api/organizations/{organizationId}`: `GET /events` (chronological by `date`, then `startTime`) and `GET /events/{eventId}`. They are for active members and managers through `@RequireOrgPermission` (MEMBER), which the T9 interceptor enforces. The controller only maps the request.
  - The service maps entities to the new `EventResponse` DTO, which has every Event field and `organizationId` instead of the nested organization. There is no status filtering, so drafts and canceled events are included, and there is no task list.
  - An unknown event, or an event of another organization, gives 404 "Event not found" via `findByIdAndOrganizationId`. This matches the membership endpoints and does not reveal that events of other organizations exist (user decision).
  - The stored status is returned unchanged until T22 defines derived statuses (user decision).
  - **Inferred from T23:** `EventController` moved to the `/api/organizations` root, and `EventService`/`EventResponse` were introduced so that T23's `POST .../events` can be added next to them. The obsolete scaffold tests were updated to the new structure.
- **Alternatives considered:**
  - A separate controller for organization events: rejected, because T23's POST belongs with the same resource.
  - 403 for events of other organizations: rejected by the user.
  - Waiting for T22/T23: rejected by the user.
  - An optional `activeOnly` query parameter (only PUBLISHED/completed events): discussed after implementation and left out by the user. It is not in the backlog, the detail endpoint shouldn't filter, participants get published events through T29, and the organization dashboard can filter on the client.
- **Limitations:**
  - T23 (create) and T22 (lifecycle) don't exist yet, so the endpoints only return events created by other means (seed data or tests) until T23 lands. Karina should build on `EventController`/`EventService`/`EventResponse`.
  - COMPLETED is not derived yet (T22).
  - The API tests mock the service and the authorization service, like the existing `*ControllerApiTest` classes. The real database is covered by `EventRepositoryTest`.
  - The full backend suite still fails because of 3 pre-existing tests from T13/T14, reported to the team.

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
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [x] Moved to `changes/completed/` (at Gate 2, by user preference; PR review and Gate 3 still pending)