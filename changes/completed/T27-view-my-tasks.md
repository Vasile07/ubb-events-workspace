# T27 - View my tasks

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T27
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** ORGZ-03
- **Use case(s):** none (ORGZ-03)
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T27-view-my-tasks`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/27
- **Tests:** `MyTaskControllerApiTest`, `MyTaskControllerStructureTest` (new); `EventTaskRepositoryTest`, `EventTaskServiceTest`, `EventTaskServiceStructureTest` (updated)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View my tasks” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORGZ-03. **Use case:** none. **Depends on:** Sign in, Init EventTask entity, Create an event with tasks.
- `GET /api/users/me/tasks`: only the tasks assigned to the current user, with the event and organization they belong to. Outside the organization path.

Dependencies from the backlog: 8, 20, 23.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code (backend):**
  - From T20/T25: `Model/.../domain/EventTask.java` (`assignedUser`, `event`), `EventTaskRepository`, `EventTaskService`/`EventTaskServiceImpl` (T25 added `listEventTasks`), and `dto/TaskAssigneeResponse` (T25). These are on branch `T25-view-event-tasks` (PR #26, not merged).
  - `RestServices/.../controllers/UserController.java` (`/api/users`, `GET /me`): it reads the user id from the JWT principal with `((Number) ((Authentication) principal).getPrincipal()).longValue()`. `UserControllerStructureTest` fixes its constructor to `UserController(UserService)`.
  - `SecurityConfig`: `/api/**` requires authentication (401). The organization interceptor only covers `/api/organizations/{organizationId}/**`, so it does not apply here.
  - `Event` (T19) and `Organization` (T6) entities, and the API/service/repository test patterns from T24/T25.
- **Related tasks:** T8 (sign in, completed), T20 (EventTask, completed), T25 (view event tasks, PR #26 open), T23 (create with tasks, not implemented).
- **Constraints / assumptions:**
  - **Branching:** the backend branch `T27-view-my-tasks` is created from `T25-view-event-tasks`, because it extends the same `EventTaskRepository`/`EventTaskService`/DTO code. Merge order: backend #25 → #26 → T27's PR, each with "Create a merge commit".
  - **T23 (user decision, as for T24/T25):** not implemented. Assigned tasks only exist through seed data or tests until T23 lands.
  - **Pre-existing failing tests on `master`:** 3 from T13/T14, recorded in T24.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add `GET /api/users/me/tasks`, outside the organization path. It returns only the tasks whose `assignedUser` is the authenticated user (the id comes from the JWT), each with the event and organization it belongs to. There is no organization check: the user only ever sees tasks assigned to themselves. Without a token, `SecurityConfig` returns 401.

The handler goes in a **new small controller** `MyTaskController` with `@RequestMapping("/api/users")` and `GET /me/tasks`, which depends on `EventTaskService`. That way the teammates' `UserController` (and its structure test, which fixes its constructor) stays unchanged. The service gets `listMyTasks(Long userId)`, which uses a new repository query and maps the results to a new `MyTaskResponse`.

**Alternatives considered:**
- Adding the handler to `UserController`: rejected, because it would change its constructor (`UserService` + `EventTaskService`) and break or require editing the teammates' `UserControllerStructureTest`.
- Adding it to `EventTaskController`: not possible, because that controller's root is `/api/organizations`.

### Steps

1. **Model (DTOs):**
   - `dto/MyTaskResponse { id, description, event: MyTaskEventResponse, organization: MyTaskOrganizationResponse }`;
   - `dto/MyTaskEventResponse { id, title, date, startTime, status }`;
   - `dto/MyTaskOrganizationResponse { id, name }`.

   All are `final` classes with `@Getter @AllArgsConstructor @EqualsAndHashCode` (**open question 1**).
2. **Persistence:** add `List<EventTask> findAllByAssignedUserIdOrderByEventDateAscEventStartTimeAscIdAsc(Long userId)` to `EventTaskRepository`: chronological by event, then by task id (**open question 2**).
3. **Business:** add `List<MyTaskResponse> listMyTasks(Long userId)` to `EventTaskService`. It maps every assigned task with no status filter, so tasks of draft and canceled events are included and their status is visible (**open question 3**).
4. **API:** new `MyTaskController` (`@RestController`, `@RequestMapping("/api/users")`, constructor-injected `EventTaskService`), handler `listMyTasks(Principal)` on `GET /me/tasks`. It extracts the user id the same way as `UserController`, and has OpenAPI `@Operation` with `@ApiResponses` for 200 and 401.
5. **Verification:** tests first and a behavioral red, then implement and get the focused tests green. Then run the full suite with `--continue`; the only failures should be the 3 known pre-existing ones.

### Files to create / modify

Backend (`repos/ubb-events-backend/`, branch `T27-view-my-tasks` from `T25-view-event-tasks`):

- Create `Model/src/main/java/com/ubb/events/dto/MyTaskResponse.java`, `MyTaskEventResponse.java`, `MyTaskOrganizationResponse.java`
- Modify `Persistence/src/main/java/com/ubb/events/persistence/EventTaskRepository.java`
- Modify `Persistence/src/test/java/com/ubb/events/persistence/EventTaskRepositoryTest.java`
- Modify `Business/src/main/java/com/ubb/events/EventTaskService.java`, `Business/src/main/java/com/ubb/events/implementation/EventTaskServiceImpl.java`
- Modify `Business/src/test/java/com/ubb/events/implementation/EventTaskServiceTest.java`, `EventTaskServiceStructureTest.java`
- Create `RestServices/src/main/java/com/ubb/events/controllers/MyTaskController.java`
- Create `RestServices/src/test/java/com/ubb/events/controllers/MyTaskControllerApiTest.java`, `MyTaskControllerStructureTest.java`

### Tests to write

Acceptance criteria → tests:

- **AC1: per the backlog (only the current user's tasks, with event and organization, outside the organization path):**
  - `MyTaskControllerApiTest.returnsOnlyCurrentUsersTasksWithEventAndOrganization`: the service is called with the id from the JWT; 200 with `id`, `description`, `event.id/title/date/startTime/status`, `organization.id/name`, and no assignee/email/password fields.
  - `MyTaskControllerApiTest.userWithoutTasksReceivesEmptyList`.
  - `MyTaskControllerApiTest.doesNotRequireOrganizationMembership`: works without any `OrganizationAuthorizationService` call.
  - `EventTaskServiceTest.listsMyTasksWithEventAndOrganization`: the mapping, including tasks of draft and canceled events.
  - `EventTaskServiceTest.returnsEmptyListWhenUserHasNoTasks`.
  - `EventTaskRepositoryTest.findsOnlyTasksAssignedToUserOrderedByEventDateTimeThenId`: excludes unassigned tasks and tasks of other users, across two organizations.
  - `MyTaskControllerStructureTest`: root `/api/users`, `GET /me/tasks`, and no org-permission annotation (the path is outside `/api/organizations`).
  - `EventTaskServiceStructureTest`: the `listMyTasks(Long)` method.
- **AC2: authorization and error cases:**
  - `MyTaskControllerApiTest.unauthenticatedRequestIsRejected`: 401, and the service is not called.
- **Expected red:**
  - complete DTOs and the declared repository method;
  - the service returns an empty list;
  - the controller exists at `/api/users`, but its handler has no `@GetMapping`.

  The failures will be assertions.

### Risks / open questions

**User decision at Gate 1: all four points confirmed as proposed.**

1. **Response shape:** `{ id, description, event: { id, title, date, startTime, status }, organization: { id, name } }`. Nested objects match the `assignee` style from T25. The event is kept as a summary; the full details stay at T24's endpoint.
2. **Order:** chronologically by the event's `date` and `startTime`, then by task id.
3. **Which tasks:** all tasks assigned to the user, **including** draft and canceled events, with `event.status` shown so the frontend can display or hide them. Alternative: exclude canceled events.
4. **Controller:** a new `MyTaskController` on `/api/users`, so the teammates' `UserController` is not touched.

**Known limitation (found during implementation, noted at the user's request):** if a user leaves an organization or is removed from it, the tasks previously assigned to them stay assigned, so they still appear in `GET /api/users/me/tasks`. No backlog rule covers this case. A follow-up should decide whether leaving or removal unassigns the tasks (likely in T26 or the membership flows), or whether this endpoint should hide tasks of organizations where the user is no longer an active member.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventTaskRepositoryTest` (1 new), `EventTaskServiceTest` (2 new), `EventTaskServiceStructureTest` (1 new), `MyTaskControllerApiTest` (new), `MyTaskControllerStructureTest` (new).
- **Stubs used:**
  - complete `MyTaskResponse`, `MyTaskEventResponse` and `MyTaskOrganizationResponse` DTOs;
  - `EventTaskRepository.findAllByAssignedUserIdOrderByEventDateAscEventStartTimeAscIdAsc` declared;
  - `EventTaskServiceImpl.listMyTasks` returns `List.of()` without querying;
  - `MyTaskController` at `/api/users` with a `listMyTasks(Principal)` handler that has no `@GetMapping` and returns an empty list.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Persistence:test --tests "com.ubb.events.persistence.EventTaskRepositoryTest" :Business:test --tests "com.ubb.events.implementation.EventTaskServiceTest" --tests "com.ubb.events.implementation.EventTaskServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.MyTaskControllerApiTest" --tests "com.ubb.events.controllers.MyTaskControllerStructureTest"`
- **Result:** `BUILD FAILED`, everything compiled, and all 6 new behavior tests failed.
  - `EventTaskServiceTest`:
    - `listsMyTasksWithEventAndOrganization` (`AssertionFailedError` at `:52`, empty list);
    - `returnsEmptyListWhenUserHasNoTasks` (Mockito `verify`: the repository was never queried).
  - `MyTaskControllerApiTest`: `returnsOnlyCurrentUsersTasksWithEventAndOrganization`, `userWithoutTasksReceivesEmptyList`, `doesNotRequireOrganizationMembership` (`AssertionError` on the status code, because `/api/users/me/tasks` is not mapped).
  - `MyTaskControllerStructureTest.myTasksHandlerIsMappedOutsideOrganizationPath` (`:27`, no `@GetMapping`).
- **Passing at red, as expected:**
  - the 3 existing T25 tests in `EventTaskServiceTest`;
  - `EventTaskRepositoryTest` (5 of 5, including the new `findsOnlyTasksAssignedToUserOrderedByEventDateTimeThenId`): Spring Data derives the query from the declared name;
  - `EventTaskServiceStructureTest` (3 of 3): the stub already declares `listMyTasks`;
  - `MyTaskControllerStructureTest.controllerIsUserScopedWithServiceDependency`: the stub already had the planned class structure;
  - `MyTaskControllerApiTest.unauthenticatedRequestIsRejected`: the existing security config returns 401.
- **Failure reason:** behavioral. The failures come from the missing planned functionality: the assigned tasks are not queried or mapped, and the endpoint is not mapped. There were no compilation or setup errors.

### Green (after implementation)

- **Result:** the same focused command gives `BUILD SUCCESSFUL`, and all 19 tests in the focused classes pass (`EventTaskRepositoryTest` 5, `EventTaskServiceTest` 5, `EventTaskServiceStructureTest` 3, `MyTaskControllerApiTest` 4, `MyTaskControllerStructureTest` 2).
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives `BUILD FAILED`: 197 tests, 3 failures, 0 errors, 0 skipped. The 3 failures are the **pre-existing** T13/T14 ones recorded in T24 (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`). There are no new failures, and all T24, T25 and T27 tests pass.

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
  - `GET /api/users/me/tasks` is outside the organization path. The user id comes from the JWT principal, in the same way as `UserController`. There is no organization check, because the query only returns tasks whose `assignedUser` is the caller, so nobody can see someone else's tasks. Without a token, `SecurityConfig` returns 401.
  - The response is `{ id, description, event: { id, title, date, startTime, status }, organization: { id, name } }`. It uses nested objects like T25's `assignee`, and the event is kept as a summary (user decision).
  - Ordering is chronological by the event's `date` and `startTime`, then by task id, done by the derived query `findAllByAssignedUserIdOrderByEventDateAscEventStartTimeAscIdAsc` (user decision).
  - All assigned tasks are returned, including those of draft and canceled events, with `event.status` visible so the frontend can decide (user decision).
  - A new `MyTaskController` on `/api/users` keeps the teammates' `UserController` and its structure test unchanged (user decision). The logic lives in `EventTaskService.listMyTasks`, next to T25's `listEventTasks`.
  - The backend branch was created from T25's branch. Merge order: #25 → #26 → T27.
- **Alternatives considered:**
  - Adding the handler to `UserController`: rejected, because it would change its constructor and its test.
  - Excluding canceled events: rejected by the user.
  - Returning the full event details: rejected, because T24's endpoint already provides them.
- **Limitations:**
  - Tasks only get assigned once T23/T26 exist; until then they come from seed data or tests.
  - If a user leaves an organization, their previously assigned tasks are still listed. No backlog rule covers this; T26 or the membership flows could unassign them.
  - The API tests mock the service, as the existing `*ControllerApiTest` classes do; the query is covered by `EventTaskRepositoryTest`.
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