# T26 - Edit event and manage tasks

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T26
- **Type:** workflow
- **Owner:** Blaj Horatiu 
- **Requirement(s):** EVT-02, EVT-06, ORGZ-02
- **Use case(s):** UC-07
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T26-edit-event-and-manage-tasks`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/42
- **Tests:** `EventServiceTest`, `EventServiceStructureTest`, `EventTaskServiceTest`, `EventTaskServiceStructureTest`, `EventTaskRepositoryTest`, `EventControllerApiTest`, `EventControllerStructureTest`, `EventTaskControllerApiTest`, `EventTaskControllerStructureTest` (updated)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Edit event and manage tasks” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** EVT-02, EVT-06, ORGZ-02. **Depends on:** Organization-scoped authorization, Init Registration entity, Event lifecycle model, Create an event with tasks.
- `PUT /api/organizations/{organizationId}/events/{eventId}`: edit the event details with permission and state checks and validation. Capacity cannot drop below the active registrations. A canceled event cannot be edited. Any manager of the organization can edit.
- Add, edit, reassign and remove tasks under `.../events/{eventId}/tasks` and `.../tasks/{taskId}`, rejecting a non-member assignee.

Dependencies from the backlog: 9, 21, 22, 23.

**Validation rules moved here from T28 (publish event).** Publishing only checks that the event starts in the future, so editing an event must enforce these on the updated values (decided by Blaj Horatiu in T28, Human Corrections #2). They're the same rules as T23 (create); **reuse T23's validation method** in `EventServiceImpl` rather than duplicating it.

- `title` is required and not blank.
- `description` is required and not blank.
- `date` and `startTime` are required, and the start (`date` + `startTime`) must be **in the future**, measured with the injected `java.time.Clock` (`ClockConfig`, Europe/Bucharest).
- `endTime` is required and must be **after `startTime`** (same day; events can't cross midnight).
- `location` is required and not blank.
- `capacity` is required and must be **> 0**, and (from the backlog) **not below the number of active registrations**.
- Collect **every** problem as an `ErrorDetail(field, message)` and throw `BusinessRuleException("…", errors)` → 400, using the field names `title`, `description`, `date`, `startTime`, `endTime`, `location`, `capacity`.

**Notes from the work already done:**

- A **CANCELED** event can't be edited (backlog) → `ConflictException` (409), consistent with T28's "a canceled event cannot be published".
- **Editing a PUBLISHED event:** these rules apply too, because publishing no longer re-validates the details. Whether a published event can still be edited at all is a T22/UC-07 decision; ask the user.
- Task assignees must be **ACTIVE members** of the event's organization, the same check as T23.
- Reassigning or removing tasks also covers the T27 limitation: tasks stay assigned to users who left the organization until someone reassigns or removes them.
- **Code:** put it in the existing `EventController`/`EventService` (root `/api/organizations`) and `EventTaskController`/`EventTaskService`, with `@RequireOrgPermission(EffectiveOrgRole.MANAGER)`. An unknown or foreign event or task gives 404 (`NotFoundException`). Follow the same DTO, test and branching conventions as noted in T23.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code (backend `master`, after T23/T33–T36/T38/T82):**
  - `Business/.../implementation/EventServiceImpl.java`:
    - `createEvent` (T23): validation, then an ACTIVE-member check for each task assignee (`MembershipRepository.findByUserIdAndOrganizationId` + `MembershipStatus.ACTIVE`), with error fields `tasks[i].description` and `tasks[i].assigneeUserId`;
    - **`validateEventDetails(CreateEventRequest)`** (T23): `categoryId`, `title`, `description`, `date`, `startTime`, start in the future (`Clock`), `endTime` after start, `location`, capacity > 0;
    - `publishEvent` (T28): only the past check;
    - `deleteEvent` (T36/T82): a DRAFT is deleted from the database; PUBLISHED becomes CANCELED; CANCELED → 409; an ended event → 409;
    - `hasEnded(event, now)`: end = `date` + `endTime` (or `startTime`), not after now.
  - `RestServices/.../controllers/EventController.java` (root `/api/organizations`: POST, GET, PATCH publish, DELETE, GET registrations) and `EventTaskController.java` (GET `/{organizationId}/events/{eventId}/tasks`).
  - `Business/.../implementation/EventTaskServiceImpl.java` (`listEventTasks`, `listMyTasks`), `Persistence/.../EventTaskRepository.java`.
  - `RegistrationRepository.countByEventId(eventId)`: registrations have no status, and cancelling one deletes the row (T33), so **active registrations = `countByEventId`**.
  - DTOs: `CreateEventRequest` (event fields + `tasks`), `EventTaskRequest {description, assigneeUserId}`, `EventResponse`, `EventTaskResponse {id, description, assignee}`, `TaskAssigneeResponse`.
  - Test patterns: `EventServiceTest`, `EventTaskServiceTest` (Mockito, fixed `Clock`), `EventControllerApiTest`, `EventTaskControllerApiTest` (MockMvc + real JWT), and the `*StructureTest` classes.
- **Related completed tasks:** T9, T19, T20, T21, T23 (create with tasks), T24/T25 (view), T27, T28 (publish; Human Corrections #2 moved the field validation to create/edit), T33 (cancel registration), T36/T82 (delete; discovery now includes CANCELED).
- **Constraints / assumptions:**
  - **T22 (lifecycle) is still in `todo/`**, so the state rules for editing are decided in this plan (open questions) and implemented in `EventServiceImpl`.
  - The validation rules listed in the Description (moved from T28) apply, and T23's `validateEventDetails` is **reused, not duplicated**.
  - **Branching:** `T26-edit-event-and-manage-tasks` from `master` (everything it needs is merged).
  - **Pre-existing failing tests on `master`:** to be re-checked at the red run (3 from T13/T14 so far).

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Four endpoints for **managers** (`@RequireOrgPermission(EffectiveOrgRole.MANAGER)`, checked by the T9 interceptor):

| Method | Path | Controller | Result |
|---|---|---|---|
| PUT | `/api/organizations/{organizationId}/events/{eventId}` | `EventController` | 200 + the updated `EventResponse` |
| POST | `/api/organizations/{organizationId}/events/{eventId}/tasks` | `EventTaskController` | 201 + `EventTaskResponse` |
| PUT | `/api/organizations/{organizationId}/events/{eventId}/tasks/{taskId}` | `EventTaskController` | 200 + `EventTaskResponse` (edit the description and/or reassign; `assigneeUserId: null` unassigns) |
| DELETE | `/api/organizations/{organizationId}/events/{eventId}/tasks/{taskId}` | `EventTaskController` | 204 |

**Editing the event (`EventService.updateEvent`, `@Transactional`):**
1. Load the event with `findByIdAndOrganizationId` → 404 "Event not found".
2. State check:
   - `CANCELED` → **409** "A canceled event cannot be edited" (backlog);
   - an **ended** event (`hasEnded`) → **409** "Event has already ended and cannot be edited" (open question 1).
3. Validate the new values with **T23's `validateEventDetails`**, refactored to take the field values so that `CreateEventRequest` and the new `UpdateEventRequest` share exactly one implementation. Then add the backlog rule **capacity ≥ active registrations** (`countByEventId`). Every problem is returned at once as **400** "Event details are invalid" with `ErrorDetail`s, the same message T23 uses.
4. Apply the fields; the **status is unchanged** (a PUBLISHED event stays PUBLISHED, a DRAFT stays DRAFT; open question 2). Save and return `EventResponse`.

**Managing tasks (`EventTaskService`, `@Transactional`):**
- The event must belong to the organization (404 "Event not found"), and a task must belong to that event (404 "Task not found", via a new `EventTaskRepository.findByIdAndEventId`).
- The same state check as editing: CANCELED or ended → 409 (open question 3).
- `description` is required and not blank → 400 (field `description`).
- `assigneeUserId` is optional; when given, the user must be an **ACTIVE member** of the event's organization → otherwise 400 (field `assigneeUserId`, message "Assignee must be an active member of the organization", as in T23).
- The assignee check is moved into one shared place (a small private helper per service, or a reusable method; see Steps) so that T23's create and T26 use the same rule.

**Alternatives considered:**
- A PATCH with partial fields: rejected, because the backlog says PUT, and a full replacement is simpler to validate with the existing rules.
- Editing tasks through the event PUT (sending the full task list): rejected, because the backlog names separate task endpoints, and replacing lists would delete and recreate tasks.
- Duplicating T23's validation: rejected, as the T26 todo notes require reuse.

### Steps

1. **Model:**
   - new `dto/UpdateEventRequest { categoryId, title, description, date, startTime, endTime, location, capacity }`, in the same Lombok style as `CreateEventRequest` (setters + no-args constructor for JSON);
   - reuse `EventTaskRequest {description, assigneeUserId}` for task add and edit.
2. **Persistence:** `Optional<EventTask> findByIdAndEventId(Long id, Long eventId)` in `EventTaskRepository`.
3. **Business, events:**
   - refactor `validateEventDetails` to a private method taking the field values, called by `createEvent` (unchanged behavior; the T23 tests must stay green) and by the new `updateEvent(Long organizationId, Long eventId, UpdateEventRequest)`;
   - the capacity ≥ registrations check;
   - the state checks.
4. **Business, tasks:** `addTask`, `updateTask` and `removeTask` in `EventTaskService`/`EventTaskServiceImpl`. `EventTaskServiceImpl` gets `MembershipRepository` and `Clock` injected for the assignee and state checks.
5. **API:**
   - `EventController.updateEvent` (`@PutMapping("/{organizationId}/events/{eventId}")`);
   - `EventTaskController.addTask`, `updateTask` and `removeTask`;
   - all MANAGER, with OpenAPI `@Operation` + `@ApiResponses` (200/201/204, 400, 401, 403, 404, 409).
6. **Structure tests:** update `EventServiceStructureTest`, `EventTaskServiceStructureTest`, `EventControllerStructureTest` and `EventTaskControllerStructureTest` for the new methods and constructors.
7. **Verification:** tests first and a behavioral red, then implement and get the focused tests green, then the full suite with `--continue`.

### Files to create / modify

Backend (`repos/ubb-events-backend/`, branch `T26-edit-event-and-manage-tasks` from `master`):

- Create `Model/src/main/java/com/ubb/events/dto/UpdateEventRequest.java`
- Modify `Persistence/src/main/java/com/ubb/events/persistence/EventTaskRepository.java` (+ `EventTaskRepositoryTest`)
- Modify `Business/src/main/java/com/ubb/events/EventService.java`, `implementation/EventServiceImpl.java` (+ `EventServiceTest`, `EventServiceStructureTest`)
- Modify `Business/src/main/java/com/ubb/events/EventTaskService.java`, `implementation/EventTaskServiceImpl.java` (+ `EventTaskServiceTest`, `EventTaskServiceStructureTest`)
- Modify `RestServices/src/main/java/com/ubb/events/controllers/EventController.java`, `EventTaskController.java` (+ `EventControllerApiTest`, `EventTaskControllerApiTest`, both `*StructureTest`)

### Tests to write

Acceptance criteria → tests:

- **AC1: edit event details:**
  - `EventServiceTest.updatesDraftEventDetails` and `updatesPublishedEventKeepingStatus`;
  - `rejectsInvalidDetailsListingEveryField` (blank title, end before start, capacity 0, past start, missing location → every field at once);
  - `rejectsCapacityBelowActiveRegistrations`;
  - `rejectsEditingCanceledEvent` (409) and `rejectsEditingEndedEvent` (409);
  - `updateThrowsNotFoundForEventOfAnotherOrganization`;
  - the T23 create tests stay green after the refactor.
- **AC1: manage tasks:**
  - `EventTaskServiceTest.addsTaskWithActiveMemberAssignee`, `addsUnassignedTask`;
  - `updatesDescriptionAndReassigns`, `unassignsWithNullAssignee`;
  - `removesTask`;
  - `rejectsNonMemberOrInactiveAssignee` (400);
  - `rejectsBlankDescription` (400);
  - `rejectsTaskOfAnotherEvent` (404 "Task not found");
  - `rejectsEventOfAnotherOrganization` (404);
  - `rejectsTaskChangesOnCanceledOrEndedEvent` (409).
  - `EventTaskRepositoryTest.findsTaskOnlyWithinItsEvent`.
- **AC2: authorization and HTTP:**
  - `EventControllerApiTest`: manager PUT → 200; member → 403; no token → 401; 400 with field errors; 404; 409.
  - `EventTaskControllerApiTest`: POST → 201, PUT → 200, DELETE → 204; member → 403; no token → 401; 400/404/409 mapped.
- **Expected red:** stub methods return `null` or do nothing, and the handlers exist without mappings, so the failures are assertions, not compilation errors.

### Risks / open questions

**User decision at Gate 1: all seven points confirmed as proposed ("all as proposed").**

1. **Ended events:** an event that has already ended can't be edited, and its tasks can't be changed (409). This is consistent with T36, where ended events can't be deleted. Alternative: allow edits after the end.
2. **Editing a PUBLISHED event:** allowed, with the same validation, and the status stays PUBLISHED. UC-07 says "which edits are allowed depends on the event state", and T22 is not done. Alternative: only DRAFT events can be edited.
3. **Task changes on CANCELED or ended events:** rejected with 409, the same as editing.
4. **The past-date rule when editing:** the start must still be in the future (the T28→T26 requirement). This means an event that has **already started but not ended** can't be edited without moving it. Alternative: skip the past check when the date and start time are unchanged.
5. **Capacity:** must be ≥ the current number of registrations (`countByEventId`); otherwise 400 on `capacity`.
6. **Refactoring T23's `validateEventDetails`:** changes code written by Karina (behavior unchanged, her tests must stay green). Tell her in the PR.
7. **The T27 limitation:** reassigning or removing tasks lets managers clean up tasks assigned to people who left. There is no automatic unassignment.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Baseline before T26** (`master` at `92e58dd`): `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives 330 tests and 3 failures, the **pre-existing** T13/T14 ones (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`).
- **Tests:**
  - `EventServiceTest`, 8 new: `updatesDraftEventDetails`, `updatesPublishedEventKeepingStatus`, `rejectsInvalidDetailsListingEveryField`, `rejectsCapacityBelowActiveRegistrations`, `acceptsCapacityEqualToActiveRegistrations`, `rejectsEditingCanceledEvent`, `rejectsEditingEndedEvent`, `updateThrowsNotFoundForEventOfAnotherOrganization`.
  - `EventServiceStructureTest.updateIsTransactional`.
  - `EventTaskServiceTest`, 10 new: `addsTaskWithActiveMemberAssignee`, `addsUnassignedTask`, `updatesDescriptionAndReassigns`, `unassignsWithNullAssignee`, `removesTask`, `rejectsNonMemberOrInactiveAssignee`, `rejectsBlankDescription`, `rejectsTaskOfAnotherEvent`, `rejectsEventOfAnotherOrganization`, `rejectsTaskChangesOnCanceledOrEndedEvent`. The test now builds the service by hand with a fixed `Clock` instead of using `@InjectMocks`.
  - `EventTaskServiceStructureTest`: constructor + `taskWriteOperationsAreTransactional`.
  - `EventTaskRepositoryTest.findsTaskOnlyWithinItsEvent`.
  - `EventControllerApiTest`: `managerUpdatesEvent`, `memberCannotUpdateEventAndTokenIsRequired`, `updateErrorsAreMapped`.
  - `EventControllerStructureTest.updateHandlerIsMappedForManagersOnly`.
  - `EventTaskControllerApiTest`: `managerAddsTask`, `managerUpdatesTask`, `managerRemovesTask`, `memberCannotChangeTasksAndTokenIsRequired`, `taskErrorsAreMapped`.
  - `EventTaskControllerStructureTest.taskWriteHandlersAreMappedForManagersOnly`.
- **Stubs used:**
  - complete `UpdateEventRequest`;
  - `EventTaskRepository.findByIdAndEventId` declared;
  - `EventService.updateEvent` and `EventTaskService.addTask`/`updateTask`/`removeTask` declared, with implementations that return `null` or do nothing (no lookup, checks, validation or save, and no `@Transactional`);
  - `EventTaskServiceImpl(EventTaskRepository, EventRepository, MembershipRepository, Clock)`;
  - the controller handlers `updateEvent`, `addTask`, `updateTask` and `removeTask` exist without mapping or authorization annotations.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Persistence:test --tests "com.ubb.events.persistence.EventTaskRepositoryTest" :Business:test --tests "com.ubb.events.implementation.EventServiceTest" --tests "com.ubb.events.implementation.EventServiceStructureTest" --tests "com.ubb.events.implementation.EventTaskServiceTest" --tests "com.ubb.events.implementation.EventTaskServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.EventControllerApiTest" --tests "com.ubb.events.controllers.EventControllerStructureTest" --tests "com.ubb.events.controllers.EventTaskControllerApiTest" --tests "com.ubb.events.controllers.EventTaskControllerStructureTest"`
- **Result:** `BUILD FAILED`, everything compiled, **30 failures** across 121 tests:
  - `EventServiceTest`, 8 of 8 new failed: "Expected BusinessRuleException/ConflictException/NotFoundException to be thrown, but nothing was thrown", `expected: <EventResponse…> but was: <null>`, or an NPE on the `null` result.
  - `EventServiceStructureTest.updateIsTransactional` (`expected: <true> but was: <false>`).
  - `EventTaskServiceTest`, 10 of 10 new failed: the same patterns, plus `removesTask` (`WantedButNotInvoked: tasks.delete`).
  - `EventTaskServiceStructureTest.taskWriteOperationsAreTransactional`.
  - `EventControllerApiTest`, 3 of 3 new, and `EventTaskControllerApiTest`, 5 of 5 new: "Status expected:<200/201/204/400/403> but was:<500>", because there is no PUT/POST/DELETE handler mapped for these paths. The 401 checks inside them passed.
  - `EventControllerStructureTest.updateHandlerIsMappedForManagersOnly` and `EventTaskControllerStructureTest.taskWriteHandlersAreMappedForManagersOnly` ("must be a PUT/POST handler").
  - `EventTaskRepositoryTest` (6 of 6) passes: Spring Data derives `findByIdAndEventId` from its name. All existing tests in these classes still pass.
- **Failure reason:** behavioral. The failures come from the missing planned functionality: no lookup or 404s, no state checks (409), no validation (400), no capacity-vs-registrations rule, no assignee check, no save or delete, no `@Transactional`, and no HTTP mappings. There were no compilation or setup errors.

### Green (after implementation)

- **Result:** the same focused command gives `BUILD SUCCESSFUL`, and all **128** tests in the focused classes pass:
  - `EventTaskRepositoryTest` 6, `EventServiceTest` 49 (including T23's create tests after the validation refactor), `EventServiceStructureTest` 6;
  - `EventTaskServiceTest` 15, `EventTaskServiceStructureTest` 4;
  - `EventControllerApiTest` 27, `EventControllerStructureTest` 7, `EventTaskControllerApiTest` 11, `EventTaskControllerStructureTest` 3.
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives `BUILD FAILED`: **361 tests** (330 at baseline + 31 new), 3 failures, 0 errors, 0 skipped. The 3 failures are exactly the **pre-existing** T13/T14 ones from the baseline. There are no new failures.

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
  - **Four manager-only endpoints** (`@RequireOrgPermission(MANAGER)`, T9 interceptor):
    - `PUT /api/organizations/{organizationId}/events/{eventId}` (200 + `EventResponse`);
    - `POST …/events/{eventId}/tasks` (201);
    - `PUT …/tasks/{taskId}` (200, edit and/or reassign; `assigneeUserId: null` unassigns);
    - `DELETE …/tasks/{taskId}` (204).
  - **Editing an event** (`EventServiceImpl.updateEvent`, `@Transactional`) works in this order:
    1. 404 when the event isn't in the organization;
    2. **409** when it's CANCELED (backlog) or **already ended** (user decision, consistent with T36's delete);
    3. validation with the **same rules as create**: T23's `validateEventDetails` was refactored into one private method taking the field values, used by both `createEvent` and `updateEvent`. The rules are the ones moved out of publishing in T28, including "start in the future" (user decision).
    4. the backlog rule **capacity ≥ active registrations** (`RegistrationRepository.countByEventId`; registrations have no status and cancelling deletes the row).

    Every problem is returned at once as 400 "Event details are invalid". The status is unchanged, so PUBLISHED events stay PUBLISHED (user decision).
  - **Managing tasks** (`EventTaskServiceImpl`, `@Transactional`) works in this order:
    1. the event must be in the organization (404 "Event not found") and editable: CANCELED or ended → 409 "Tasks of a canceled/an ended event cannot be changed" (user decision);
    2. the task must belong to that event (`findByIdAndEventId`, 404 "Task not found");
    3. the description is required, and an assignee is optional but must be an **ACTIVE member** (the same rule and message as T23) → 400 "Task details are invalid".
- **Alternatives considered:**
  - A PATCH with partial fields: the backlog says PUT, and full replacement reuses the create validation as-is.
  - Replacing the task list through the event PUT: rejected, because the backlog names separate task endpoints and replacing would delete and recreate tasks.
  - Duplicating the validation: rejected (the T26 todo notes and the user decision ask for reuse).
  - Allowing edits of ended events, or only of DRAFT events: rejected by the user.
- **Limitations:**
  - **"Ended" is defined in two places:** `EventServiceImpl.hasEnded` and `EventTaskServiceImpl.editableEvent` both use `date` + `endTime` (or `startTime`). T22 should move it into one lifecycle component.
  - An event that has started but not ended can't be edited without moving its start into the future, because the past-start rule applies on edit (user decision).
  - There is no automatic unassignment when a member leaves (T27). Managers can now reassign or remove tasks manually.
  - Karina's `validateEventDetails` (T23) was refactored, with behavior unchanged and her tests still green; this is mentioned in the PR.
  - The full suite still has the 3 pre-existing T13/T14 failures.

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