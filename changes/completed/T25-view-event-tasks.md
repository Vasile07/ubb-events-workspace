# T25 - View event tasks

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T25
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** ORGZ-02
- **Use case(s):** UC-07
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T25-view-event-tasks`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/26 (depends on https://github.com/Vasile07/ubb-events-backend/pull/25)
- **Tests:** `EventTaskControllerApiTest`, `EventTaskServiceTest` (new); `EventTaskRepositoryTest`, `EventTaskServiceStructureTest`, `EventTaskControllerStructureTest` (updated)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View event tasks” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORGZ-02. **Depends on:** Organization-scoped authorization, Init EventTask entity, Create an event with tasks.
- `GET /api/organizations/{organizationId}/events/{eventId}/tasks`, for members and managers: the tasks with their descriptions and assignees. Verify that the event belongs to the organization.
- Never returned by the public event endpoints.

Dependencies from the backlog: 9, 20, 23.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code (backend):**
  - From T20: `Model/.../domain/EventTask.java` (`public.tasks`: `description`, optional `@ManyToOne User assignedUser`, required `@ManyToOne Event event`), `Persistence/.../EventTaskRepository.java`, and the empty `EventTaskService`/`EventTaskServiceImpl`/`EventTaskController` scaffolds (`/api/event-tasks`) with their `*StructureTest` classes.
  - From T24 (backend PR #25, not merged yet): `EventRepository.findByIdAndOrganizationId`, `NotFoundException("Event not found")` in `EventServiceImpl`, and `EventController` under `/api/organizations` with `@RequireOrgPermission`, OpenAPI annotations and `EventControllerApiTest`. These are the patterns T25 follows.
  - `Model/.../domain/User.java` (`id`, `firstName`, `lastName`, `email`, `password`) and `dto/OrganizationMemberResponse.java` (a flat DTO style that never exposes the password).
- **Related tasks:** T9 (authorization, completed), T20 (EventTask, completed), T24 (view organization events, PR #25 open).
- **Constraints / assumptions:**
  - **Branching (user decision):** the backend branch `T25-view-event-tasks` will be created from `T24-view-organization-events`, not from `master`, so T25 can reuse T24's event lookup. The T25 PR targets `master` and depends on PR #25. **#25 must be merged with "Create a merge commit"**, not squash, so no rebase is needed.
  - **Dependency on T23 (user decision, as in T24):** T23 (Create an event with tasks) isn't implemented, so tasks only exist through seed data or tests until T23 lands. T25 only reads tasks, so T23 doesn't change its behavior.
  - **Public endpoints:** "Never returned by the public event endpoints." No public event endpoints exist yet (T29/T31 are in `todo/`), and T24's organization endpoints don't include tasks. There is nothing to change for this now; it's a rule for T29/T31.
  - **Pre-existing failing tests on `master`:** 3 tests from T13/T14 (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`), reported to the team in T24.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add `GET /api/organizations/{organizationId}/events/{eventId}/tasks` for active members and managers (`@RequireOrgPermission`, MEMBER level; the T9 interceptor checks membership in `organizationId`).

The service first checks that the event belongs to the organization through `EventRepository.findByIdAndOrganizationId` (from T24). If it doesn't, it throws `NotFoundException("Event not found")` → 404, the same as T24's detail endpoint. This blocks reading the tasks of another organization's event through your own `organizationId`. It then loads the event's tasks and maps them to a new `EventTaskResponse` with the task id, the description and the assignee (or `null` if the task is unassigned). The assignee's password is never included.

`EventTaskController` moves from the `/api/event-tasks` scaffold to the `/api/organizations` root, following the T24 `EventController` pattern. T23/T26 can later add task creation and editing to the same controller and service. The obsolete scaffold structure tests are updated.

**Alternatives considered:**
- Putting the tasks inside T24's event response: rejected, because the backlog says "the task list is a separate endpoint".
- Adding the handler to `EventController`: rejected to keep one controller per resource; `EventTaskController` already exists for tasks.
- 403 for an event of another organization: rejected in favor of 404, consistent with T24.

### Steps

1. **Model (DTO):** add `dto/EventTaskResponse`, a `final` class with `@Getter @AllArgsConstructor @EqualsAndHashCode`. The assignee is a nested object, `assignee: { id, firstName, lastName }`, or `null` if unassigned (**open question 1**). Fields: `id`, `description`, `assignee` (`TaskAssigneeResponse`, a `final` class with `id`, `firstName`, `lastName`).
2. **Persistence:** add `List<EventTask> findAllByEventIdOrderByIdAsc(Long eventId)` to `EventTaskRepository`. Order: by task id, i.e. creation order (**open question 2**).
3. **Business:**
   - add `List<EventTaskResponse> listEventTasks(Long organizationId, Long eventId)` to `EventTaskService`;
   - `EventTaskServiceImpl` gets `EventTaskRepository` and `EventRepository` through its constructor;
   - if `findByIdAndOrganizationId(eventId, organizationId)` is empty, it throws `NotFoundException("Event not found")`; otherwise it maps the tasks.
4. **API:** change `EventTaskController` to `@RequestMapping("/api/organizations")` and add `listEventTasks` on `GET /{organizationId}/events/{eventId}/tasks`, with `@RequireOrgPermission` and OpenAPI `@Operation` with `@ApiResponses` for 200, 400, 401, 403 and 404.
5. **Obsolete scaffold tests:** update `EventTaskServiceStructureTest` and `EventTaskControllerStructureTest` to the new structure, as T24 did for the event scaffolds.
6. **Verification:** tests first and a behavioral red against stubs, then implement and get the focused tests green. Then run the full suite with `--continue`; the only failures should be the 3 known pre-existing ones.

### Files to create / modify

Backend (`repos/ubb-events-backend/`, branch `T25-view-event-tasks` from `T24-view-organization-events`):

- Create `Model/src/main/java/com/ubb/events/dto/EventTaskResponse.java`, `Model/src/main/java/com/ubb/events/dto/TaskAssigneeResponse.java`
- Modify `Persistence/src/main/java/com/ubb/events/persistence/EventTaskRepository.java`
- Modify `Persistence/src/test/java/com/ubb/events/persistence/EventTaskRepositoryTest.java`
- Modify `Business/src/main/java/com/ubb/events/EventTaskService.java`, `Business/src/main/java/com/ubb/events/implementation/EventTaskServiceImpl.java`
- Create `Business/src/test/java/com/ubb/events/implementation/EventTaskServiceTest.java`
- Modify `Business/src/test/java/com/ubb/events/implementation/EventTaskServiceStructureTest.java`
- Modify `RestServices/src/main/java/com/ubb/events/controllers/EventTaskController.java`
- Create `RestServices/src/test/java/com/ubb/events/controllers/EventTaskControllerApiTest.java`
- Modify `RestServices/src/test/java/com/ubb/events/controllers/EventTaskControllerStructureTest.java`

### Tests to write

Acceptance criteria → tests:

- **AC1: the endpoint per the backlog (members and managers, descriptions and assignees, event belongs to the organization):**
  - `EventTaskControllerApiTest.memberReceivesEventTasksWithAssignees`: 200; it checks `id`, `description`, `assignee.id`/`firstName`/`lastName`, an unassigned task with `assignee: null`, and that no `password`/`email` appears.
  - `EventTaskControllerApiTest.managerCanViewEventTasks`: the MEMBER-level check passes for a manager.
  - `EventTaskControllerApiTest.eventWithoutTasksReturnsEmptyList`.
  - `EventTaskServiceTest.listsTasksOfEventOfOrganizationWithAssignees`: maps the assigned and the unassigned task.
  - `EventTaskServiceTest.returnsEmptyListForEventWithoutTasks`.
  - `EventTaskRepositoryTest.findsOnlyTasksOfEventOrderedById`: excludes the tasks of another event.
- **AC2: authorization and error cases:**
  - `EventTaskControllerApiTest.unauthenticatedRequestIsRejected`: 401.
  - `EventTaskControllerApiTest.nonMemberIsForbidden`: 403, and the service is not called.
  - `EventTaskControllerApiTest.eventOfAnotherOrganizationOrUnknownReturnsNotFound`: 404 "Event not found".
  - `EventTaskServiceTest.throwsNotFoundWhenEventNotInOrganization`: the tasks are not queried.
- **Structure tests:** the updated `EventTaskServiceStructureTest` and `EventTaskControllerStructureTest`.
- **Expected red:**
  - complete DTOs and the declared repository method;
  - the service stub returns an empty list without checking the event;
  - the controller stub stays at `/api/event-tasks` with an unmapped handler.

  The failures will be assertions, not compilation errors.

### Risks / open questions

User decisions (confirmed at Gate 1):

1. **Assignee shape:** a nested `assignee: { id, firstName, lastName }`, or `null` when unassigned, **without email**. **Confirmed by the user.**
2. **Task order:** by creation order. The `tasks` table has no creation-date column, so this is implemented as ascending task id, which the identity column assigns in insertion order. **Confirmed by the user.**
3. **Unknown event, or an event of another organization:** 404 "Event not found". **Confirmed by the user.**
4. **Merge order:** PR #25 (T24) must be merged first, with "Create a merge commit". Until then, the T25 PR also shows T24's commit.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventTaskRepositoryTest` (1 new), `EventTaskServiceTest` (new), `EventTaskServiceStructureTest` (updated), `EventTaskControllerApiTest` (new), `EventTaskControllerStructureTest` (updated).
- **Stubs used:**
  - complete `EventTaskResponse` and `TaskAssigneeResponse` DTOs;
  - `EventTaskRepository.findAllByEventIdOrderByIdAsc` declared;
  - `EventTaskServiceImpl` with `@Service` and both repositories injected, but `listEventTasks` returns `List.of()` without checking the event or querying the tasks;
  - `EventTaskController` still at `/api/event-tasks`, with a handler that has no `@GetMapping` and no `@RequireOrgPermission`.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Persistence:test --tests "com.ubb.events.persistence.EventTaskRepositoryTest" :Business:test --tests "com.ubb.events.implementation.EventTaskServiceTest" --tests "com.ubb.events.implementation.EventTaskServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.EventTaskControllerApiTest" --tests "com.ubb.events.controllers.EventTaskControllerStructureTest"`
- **Result:** `BUILD FAILED`, everything compiled, 10 of 17 tests failed.
  - `EventTaskServiceTest`, 3 of 3 failed:
    - `listsTasksOfEventOfOrganizationWithAssignees` (`AssertionFailedError` at `:47`, empty list);
    - `returnsEmptyListForEventWithoutTasks` (Mockito `verify`: the tasks were never queried);
    - `throwsNotFoundWhenEventNotInOrganization` (`:67`, no exception).
  - `EventTaskControllerApiTest`, 5 of 6 failed:
    - `nonMemberIsForbidden`, `memberReceivesEventTasksWithAssignees`, `managerCanViewEventTasks`, `eventWithoutTasksReturnsEmptyList`, `eventOfAnotherOrganizationOrUnknownReturnsNotFound` (`AssertionError` on the status code, because no handler is mapped for `/api/organizations/{organizationId}/events/{eventId}/tasks`).
  - `EventTaskControllerStructureTest`, 2 of 2 failed (root mapping `/api/event-tasks`, no `@GetMapping`/`@RequireOrgPermission`).
- **Passing at red, as expected:**
  - `EventTaskRepositoryTest` (4 of 4, including the new `findsOnlyTasksOfEventOrderedById`): Spring Data derives the query from the declared method name.
  - `EventTaskServiceStructureTest` (2 of 2): the stub already had the planned structure.
  - `EventTaskControllerApiTest.unauthenticatedRequestIsRejected`: the existing security config already returns 401.
- **Failure reason:** behavioral. The failures come from the missing planned functionality: no event-organization check, no task query or mapping, no `NotFoundException`, and no organization-scoped GET mapping or authorization annotation. There were no compilation or setup errors.

### Green (after implementation)

- **Result:** the same focused command gives `BUILD SUCCESSFUL`, and all 17 focused tests pass (`EventTaskRepositoryTest` 4, `EventTaskServiceTest` 3, `EventTaskServiceStructureTest` 2, `EventTaskControllerApiTest` 6, `EventTaskControllerStructureTest` 2).
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives `BUILD FAILED`: 187 tests, 3 failures, 0 errors, 0 skipped. The 3 failures are the **pre-existing** T13/T14 ones recorded in T24 (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`). There are no new failures, and all T24 and T25 tests pass.

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
  - `GET /api/organizations/{organizationId}/events/{eventId}/tasks` is for active members and managers through `@RequireOrgPermission` (MEMBER), which the T9 interceptor enforces. The controller only maps the request.
  - The service first checks `EventRepository.findByIdAndOrganizationId(eventId, organizationId)` (from T24). An unknown event, or an event of another organization, gives 404 "Event not found" before any task is read, so a member of organization A cannot list the tasks of B's events by using A's `organizationId` (user decision).
  - The response is `EventTaskResponse { id, description, assignee }`, with a nested `assignee { id, firstName, lastName }` or `null`. There is no email and no password (user decision).
  - Tasks are ordered by creation order, implemented as ascending task id, because `tasks` has no creation-date column (user decision).
  - `EventTaskController` moved from the `/api/event-tasks` scaffold to `/api/organizations`, like T24's `EventController`, so T23/T26 can add task creation and editing next to it. The obsolete scaffold tests were updated.
  - The backend branch was created from T24's branch (user decision) to reuse its event lookup. PR #25 must be merged with a merge commit first.
- **Alternatives considered:**
  - Embedding the tasks in T24's event response: rejected, because the backlog wants a separate endpoint.
  - Adding the handler to `EventController`: rejected, to keep one controller per resource.
  - 403 for another organization's event: rejected in favor of 404, consistent with T24.
  - Flat assignee fields or including the email: rejected by the user.
- **Limitations:**
  - Tasks only exist through seed data or tests until T23 (create with tasks) lands.
  - "Never returned by the public event endpoints" can't be tested yet, because no public event endpoints exist; T29/T31 must respect it.
  - The API tests mock the service and authorization, as the existing `*ControllerApiTest` classes do; the database query is covered by `EventTaskRepositoryTest`.
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