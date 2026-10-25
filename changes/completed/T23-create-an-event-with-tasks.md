# T23 - Create an event with tasks

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T23
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):** EVT-01, EVT-06, ORGZ-01, LIF-01
- **Use case(s):** UC-06
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T23-create-an-event-with-tasks`
- **PR link(s):** 
- **Tests:** `EventServiceTest`, `EventServiceStructureTest`, `EventControllerApiTest`, `EventControllerStructureTest`; existing `EventRepositoryTest` and `EventTaskRepositoryTest` cover persistence and reload after `TestEntityManager.clear()`.

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Create an event with tasks” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** EVT-01, EVT-06, ORGZ-01, LIF-01. **Depends on:** Init Membership entity, Organization-scoped authorization, Init Event entity, Init EventTask entity, Event lifecycle model.
- `POST /api/organizations/{organizationId}/events`, for managers: create an event as a draft with validation (title, description, start and end date and time with the end after the start, location, capacity, registration information). Returns 201 with the event id and the draft status.
- Create written tasks on the event and assign them to members, rejecting a non-member of the organizing organization.

Dependencies from the backlog: 7, 9, 19, 20, 22.

**Validation rules moved here from T28 (publish event).** Publishing only checks that the event starts in the future, so creating an event must enforce these (decided by Blaj Horatiu in T28, Human Corrections #2):

- `title` is required and not blank.
- `description` is required and not blank.
- `date` and `startTime` are required, and the start (`date` + `startTime`) must be **in the future**, measured with the injected `java.time.Clock` (`ClockConfig`, Europe/Bucharest). Never call `LocalDateTime.now()` without the clock.
- `endTime` is required and must be **after `startTime`** (same day; events can't cross midnight, because the schema has a single `date`).
- `location` is required and not blank.
- `capacity` is required and must be **> 0**.
- Collect **every** problem as an `ErrorDetail(field, message)` and throw `BusinessRuleException("…", errors)` → 400, so the manager sees everything to correct at once. Use the field names `title`, `description`, `date`, `startTime`, `endTime`, `location`, `capacity`, the same names T28 used.
- Put the checks in a reusable private method in `EventServiceImpl` (for example `validateEventDetails(...)`), because T26 (edit) must apply exactly the same rules.

**Alignment with the work already done (T19, T20, T24, T25, T27–T30):**

- **Entities:**
  - `Event` (T19): organization `@ManyToOne`, `categoryId` (a temporary scalar `Long`; there is no Category entity yet), `title`, `description`, `date`, `startTime`, `endTime`, `location`, `capacity`, and `status` (`EventStatus`: DRAFT, PUBLISHED, CANCELED). There is **no** `deleted` flag and **no** creator field; CANCELED means canceled or deleted.
  - `EventTask` (T20): `description` (required), `assignedUser` (`@ManyToOne User`, nullable), `event` (required). There is no title, due date or status.
- **Where to put the code:**
  - `POST "/{organizationId}/events"` goes into the existing `EventController` (root `/api/organizations`), with `@RequireOrgPermission(EffectiveOrgRole.MANAGER)` and OpenAPI `@Operation` + `@ApiResponses` (201, 400, 401, 403).
  - The logic goes into `EventService`/`EventServiceImpl` (`@Transactional`).
  - Return **201** with the existing `EventResponse` DTO; the status is always **DRAFT**.
- **Tasks created with the event:** each assignee is optional. If an assignee is given, it must be an **ACTIVE member** of the event's organization (`MembershipRepository.findByUserIdAndOrganizationId` + `MembershipStatus.ACTIVE`). Otherwise return 400 with an `ErrorDetail` and save nothing (the whole creation is one transaction). Reuse `EventTaskRepository` and the `TaskAssigneeResponse` style (`{id, firstName, lastName}`, never email or password).
- **Errors:** validation → `BusinessRuleException` (400, with an `ErrorDetail` list); wrong state → `ConflictException` (409); unknown or foreign event → `NotFoundException("Event not found")` (404). Authorization stays in the T9 interceptor; the JWT carries only the user id.
- **Never expose tasks** on public endpoints (`GET /api/events` from T29/T30).
- **Code style:**
  - DTOs are `final` classes with Lombok `@Getter @AllArgsConstructor @EqualsAndHashCode` (no records); use constructor injection.
  - Tests come first (TDD):
    - Mockito service tests with a fixed `Clock` (`Clock.fixed(..., ZoneId.of("Europe/Bucharest"))`);
    - `@DataJpaTest` repository tests that call `TestEntityManager.clear()` before reloading;
    - MockMvc API tests with a real JWT (`JwtTokenService.issue(userId)`) and `@MockitoBean OrganizationAuthorizationService`;
    - update the `EventControllerStructureTest`/`EventServiceStructureTest` classes when signatures change.
- **Branching:** branch `T23-create-an-event-with-tasks` from `master` **after** backend PRs #28 → #29 → #30 are merged, because #28 brings the `Clock` bean and they all change `EventService`/`EventController`. If they're not merged yet, branch from `T30-filter-events` and ask the user. Merge with "Create a merge commit", never squash.
- **Full suite:** `.\gradlew.bat --no-daemon --console=plain --continue clean test`. There are 3 **pre-existing** failures on `master` that are not T23's (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`).
- **Keep it consistent for T22 and T26:**
  - Leave the DRAFT → PUBLISHED rule in `publishEvent` (T28).
  - An event counts as "ended" when `date` + `endTime` (or `startTime` if missing) is not after now (T30 ordering); T22 should derive COMPLETED with the same definition.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** Backend `RestServices/.../controllers/EventController.java` (organization-scoped GET/publish and manager authorization), `Business/.../EventService.java` and `implementation/EventServiceImpl.java` (event lookup, publish validation, `Clock` injection, response mapping), `EventTaskController.java` / `EventTaskServiceImpl.java` (event-task response mapping), `Persistence/.../EventRepository.java`, `EventTaskRepository.java`, `MembershipRepository.java`, and DTOs `EventResponse`, `TaskAssigneeResponse`. Existing tests include `EventControllerApiTest`, `EventControllerStructureTest`, `EventServiceTest`, `EventServiceStructureTest`, and repository tests.
- **Related completed tasks:** T19 (Event entity), T20 (EventTask entity), T24 (organization event viewing), T25 (view event tasks), T28 (publish event; field validation moved to create/edit and `Clock` added), T29 and T30 (public event discovery/filtering). T28's completed task records the validation correction relevant to T23.
- **Constraints / assumptions:** Task scope is backend only. Workspace guidance requires the task branch `T23-create-an-event-with-tasks` from `master`; backend is clean on `master`, and local `master` is up to date after fetch/pull. The task specifies manager authorization in the T9 interceptor, one transaction for event plus tasks, active same-organization membership for any assignee, aggregated `BusinessRuleException` validation, and reuse of `Clock` for future-start validation. Project conventions require constructor injection and DTO classes (not records). Public discovery must not expose tasks. No unresolved task-level questions found.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the workflow as one transactional backend operation. Add request DTOs for event details and optional task assignments, expose a manager-only `POST /api/organizations/{organizationId}/events`, and extend `EventService` with a create method. The service will validate all event fields with the injected Europe/Bucharest `Clock`, aggregate field errors, create the event as `DRAFT`, validate each optional assignee as an ACTIVE member of the same organization, then persist the event and its tasks together. Reuse the existing response mapping and task-assignee shape; authorization remains in the organization interceptor. Keep task data out of public discovery responses. A separate task-creation endpoint or controller-level business validation is not planned because the backlog requires creation with the event and business rules belong in the service.

### Steps

1. **Tests first (service):** extend `EventServiceTest` and `EventServiceStructureTest` for the create signature, fixed-clock validation, draft status, response mapping, aggregated errors, assignee membership checks, and transaction-oriented save behavior. Cover valid events with and without tasks, every invalid event field, a non-future start, missing/blank task descriptions if applicable to the request contract, inactive/non-member assignees, and the guarantee that invalid input does not save the event or tasks.
2. **Tests first (persistence):** add or extend `@DataJpaTest` coverage for the repository interactions needed by creation, including event/task persistence and reloading after `TestEntityManager.clear()`; verify the existing membership lookup supports organization-scoped assignee validation.
3. **Tests first (API/security):** extend `EventControllerApiTest` and `EventControllerStructureTest` with a real JWT and mocked `OrganizationAuthorizationService`: assert 201 and a DRAFT `EventResponse`, malformed/invalid input as 400 with all `ErrorDetail`s, non-member assignee as 400, manager authorization metadata, unauthenticated 401, and non-manager 403. Update OpenAPI response declarations as needed.
4. **API layer:** add final Lombok request DTOs for event creation and task assignment (using the established constructor-injection/DTO conventions), map `POST /{organizationId}/events` in `EventController`, annotate it with manager permission and 201/400/401/403 responses, and return `ResponseEntity.status(CREATED)`.
5. **Service layer:** add the create operation to `EventService` and implement it in `EventServiceImpl` as `@Transactional`. Add reusable `validateEventDetails(...)` checks for required/nonblank fields, future start using the injected `Clock`, end time after start, and positive capacity; throw one `BusinessRuleException` containing every field error. Create the DRAFT event, resolve optional assignees through `MembershipRepository.findByUserIdAndOrganizationId` and `MembershipStatus.ACTIVE`, reject invalid assignees with an `ErrorDetail`, create `EventTask` entities, save atomically, and return `EventResponse`.
6. **Persistence/model integration:** inject and use `EventTaskRepository` and `MembershipRepository` without changing entity semantics; ensure task assignee responses expose only id, first name, and last name. Preserve existing publish and public discovery behavior, including no task exposure.
7. **Verification:** run focused backend tests, then `./gradlew.bat --no-daemon --console=plain --continue clean test`; distinguish the three documented pre-existing failures from T23 regressions and record exact red/green evidence in the task file.

### Files to create / modify

- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto` — create the event-creation and task-assignment request DTOs, following existing final Lombok DTO style.
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/EventService.java` and `.../implementation/EventServiceImpl.java` — add the create contract, reusable validation, membership checks, event/task persistence, and response mapping.
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/EventController.java` — add the manager-only POST endpoint and OpenAPI annotations.
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/EventServiceImpl.java` — inject `EventTaskRepository` and `MembershipRepository` as required.
- `../../repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/EventServiceTest.java` and `EventServiceStructureTest.java` — service and signature tests.
- `../../repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence/EventRepositoryTest.java` and `EventTaskRepositoryTest.java` — persistence coverage where needed.
- `../../repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/EventControllerApiTest.java` and `EventControllerStructureTest`.java` — API, authorization, and contract tests.

### Tests to write

- `createEventWithNoTasksReturnsDraftResponse` and `createEventWithAssignedActiveMemberPersistsTasks` map to the successful workflow and 201 acceptance criteria.
- `createEventAggregatesAllInvalidFieldErrors`, `rejectsPastOrCurrentStart`, `rejectsEndTimeNotAfterStart`, and `rejectsNonPositiveCapacity` map to the validation requirements and verify the injected fixed clock.
- `rejectsAssigneeOutsideOrganizationOrInactiveMember` maps to organization membership enforcement and verifies no event/task save occurs.
- `postCreateEventRequiresManagerAndReturnsExpectedStatuses` maps to manager authorization, JWT/API behavior, and 201/400/401/403 coverage.
- Repository reload tests verify persisted event status, organization relation, and task/assignee relations after `TestEntityManager.clear()`.

### Risks / open questions

- The exact JSON property names and nesting for the creation request must follow existing DTO/controller conventions; confirm against neighboring request DTOs while implementing, without changing the specified endpoint behavior.
- The task description is required by the `EventTask` entity; the plan assumes blank task descriptions are rejected with a field-level error if the request DTO permits them. If the existing API contract defines a different task payload, update the plan before implementation.
- The branch prerequisite is satisfied locally (`master` contains PRs #28–#30); the implementer must still re-check repository cleanliness and branch from up-to-date `master` after Gate 1.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventServiceTest` creation cases, `EventServiceStructureTest`, `EventControllerApiTest`, and `EventControllerStructureTest`.
- **Result:** Before the user instructed that tests must not be run, the focused command could not reach test execution: the Gradle wrapper hung because the Java shim did not start; using the installed JDK 21 failed before tests with a missing Gradle instrumentation agent, and running Gradle without the wrapper agent failed to initialize `native-platform.dll`. No test assertions ran. The user then explicitly directed that tests be written but not run, and implementation should proceed.
- **Failure reason:** The observed failure was environmental, not behavioral, so there is no valid red result. Tests were not rerun after the user instruction.

### Green (after implementation)

- **Result:** Not run, per the user's instruction. No passing result is claimed.
- **Full suite of the affected repo:** Not run, per the user's instruction.

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

- **Decisions:** The manager-only create endpoint delegates to one transactional service operation. The service validates event details against the injected `Clock`, checks optional assignees against ACTIVE memberships in the same organization, and saves the DRAFT event with its tasks in the same transaction. The create request includes the scalar `categoryId` required by the current events schema; no Category entity exists.
- **Alternatives considered:** Returning tasks from the create endpoint was not selected because the task specifies the existing `EventResponse`; task details remain available through the organization-scoped task endpoint. A separate task-creation endpoint was not selected because creation with tasks is specified as one workflow.
- **Limitations:** Tests were authored but not executed at the user's direction, so there is no behavioral red or green evidence. The current event entity/schema has no Category entity and no registration-information fields; the implementation uses fields currently supported by the entity.

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
