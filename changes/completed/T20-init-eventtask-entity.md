# T20 - Init EventTask entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T20
- **Type:** entity
- **Owner:** Blaj Horatiu
- **Requirement(s):** R7
- **Use case(s):** UC-06 Create event; UC-07 Edit event and manage assigned tasks.
- **Workflow:** entity
- **Drive doc / ADR:** [Project requirements, use cases, architecture, ERD, API specification, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban)
- **Repo(s):** backend
- **Branch:** `T20-init-eventtask-entity`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/16
- **Tests:** `EventTaskTest`, `EventTaskRepositoryTest`, `EventTaskServiceStructureTest`, `EventTaskControllerStructureTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init EventTask entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/EventTask.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Init Membership entity, Init Event entity.
- Entity scaffold for EventTask: title, description and an optional assignee (a member of the organization). A task has no due date and no status.

Dependencies from the backlog: 7, 19.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/docs/coding-conventions.md`. Entity patterns: `Model/.../domain/Event.java` (T19), `Message.java` and `Registration.java` (required `@ManyToOne Event` on `event_id`), `Membership.java` (`@ManyToOne User`). Scaffold patterns: `FollowUpRepository`, `FollowUpService`/`FollowUpServiceImpl`, `MessageController` (`/api/chat-messages`, a kebab-case path) and their `*StructureTest` classes. Repository tests: `MessageRepositoryTest` (saves Organization → Event → entity, clears the `TestEntityManager` before reloading).
- **Related completed tasks:** T19 (Event entity, merged in backend PR #15), T7 (Membership), T4 (User), T40 (Message).
- **Spec:** `docs/entities/Task.md` (table `public.tasks`). The task description refers to `docs/entities/EventTask.md`, which does not exist.
- **Constraints / assumptions:** T19 is merged, so `Event` is on backend `master` and T20 branches from `master`. User decisions: `Task.md` is authoritative over the backlog (no `title`), and the class is named `EventTask`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the EventTask persistence scaffold from `docs/entities/Task.md`. Map class `EventTask` to `public.tasks` with:

- an identity-generated `task_id`;
- a required `String description`;
- an optional `@ManyToOne User assignedUser` on nullable `assigned_user_id`;
- a required `@ManyToOne(optional = false) Event event` on `event_id`.

Add the empty repository, service interface and implementation, and controller scaffold in the same style as FollowUp/Message. There are no business rules and no endpoints.

User decisions: `Task.md` is authoritative, so there is no `title`, no due date and no status. The class is called `EventTask`, which avoids a clash with the generic word "Task".

The backlog rule "the assignee is a member of the organization" is a business rule, so it is not enforced in this entity scaffold. The workflow tasks (T23/T26) own it.

Alternatives:

- A `title` column was rejected because it is not in the schema.
- An assignee typed as `Membership` was rejected because the schema's foreign key points to `users.user_id`.

### Steps

1. **Model:** add `EventTask`:
   - `@Entity` and `@Table(name = "tasks", schema = "public")`, with Lombok `@Getter @Setter @NoArgsConstructor @AllArgsConstructor`;
   - field order `id`, `description`, `assignedUser`, `event`, matching `Task.md`;
   - `id`: `Long`, `task_id`, identity;
   - `description`: `String`, `@Column(name = "description", nullable = false)`;
   - `assignedUser`: `User`, `@ManyToOne` + `@JoinColumn(name = "assigned_user_id")`, nullable;
   - `event`: `Event`, `@ManyToOne(optional = false)` + `@JoinColumn(name = "event_id", nullable = false)`.
2. **Persistence:** add `EventTaskRepository extends JpaRepository<EventTask, Long>` (`@Repository`).
3. **Business:** add an empty `EventTaskService` interface and an `@Service EventTaskServiceImpl`.
4. **API:** add `EventTaskController`: `@RestController`, `@RequestMapping("/api/event-tasks")` (kebab-case like `/api/chat-messages`), constructor-injected `EventTaskService`, and no handlers.
5. **Verification:** write the tests first and get a behavioral red against compile-only stubs, then implement, get the focused tests green, and run the full backend suite (`.\gradlew.bat clean test`). Record everything in the Test log.

### Files to create / modify

Backend (`repos/ubb-events-backend/`), create only:

- `Model/src/main/java/com/ubb/events/domain/EventTask.java`
- `Model/src/test/java/com/ubb/events/domain/EventTaskTest.java`
- `Persistence/src/main/java/com/ubb/events/persistence/EventTaskRepository.java`
- `Persistence/src/test/java/com/ubb/events/persistence/EventTaskRepositoryTest.java`
- `Business/src/main/java/com/ubb/events/EventTaskService.java`
- `Business/src/main/java/com/ubb/events/implementation/EventTaskServiceImpl.java`
- `Business/src/test/java/com/ubb/events/implementation/EventTaskServiceStructureTest.java`
- `RestServices/src/main/java/com/ubb/events/controllers/EventTaskController.java`
- `RestServices/src/test/java/com/ubb/events/controllers/EventTaskControllerStructureTest.java`

### Tests to write

Acceptance criteria → tests:

- **AC1: implemented according to the specification (`Task.md`):** `EventTaskTest.eventTaskHasExpectedJpaMapping` checks:
  - the entity and table `public.tasks`;
  - an identity id on `task_id`;
  - `description` as a non-null column;
  - `assignedUser` as a `User` `@ManyToOne` with a **nullable** `assigned_user_id` join column;
  - `event` as an `Event` `@ManyToOne(optional = false)` with a non-null `event_id` join column;
  - no `title`, `dueDate` or `status` field.
- **AC2: scaffold behavior and constraints are covered:**
  - `EventTaskRepositoryTest.savesAndReloadsTaskWithEventAndAssignedUser`: saves Organization → Event → User → EventTask, clears the context, reloads, and checks the description, event id and assigned user id.
  - `EventTaskRepositoryTest.savesTaskWithoutAssignedUser`: the assignee is optional, and a null assignee is stored and reloaded as null.
  - `EventTaskRepositoryTest.rejectsNullRequiredValues`: `DataIntegrityViolationException` for a null description and a null event.
  - `EventTaskServiceStructureTest` and `EventTaskControllerStructureTest`: the same assertions as the FollowUp/Message structure tests, with path `/api/event-tasks`.
- **Expected red:** stubs where `EventTask` has only `@Entity` + `@Id` and `@Transient` fields, and the service and controller lack their annotations. They compile and fail on assertions or integrity checks.

### Risks / open questions

- **Assignee membership rule:** "the assignee is a member of the organization" is not enforced by the schema or this scaffold. Workflow tasks must validate it in the service layer.
- **Controller path:** `/api/event-tasks` is a scaffold with no handlers. Workflow tasks will probably use organization-scoped paths (`/api/organizations/{orgCode}/...`) and may replace it.
- **Task description:** it references a non-existent `docs/entities/EventTask.md`. This plan uses `Task.md` instead.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventTaskTest` (Model), `EventTaskRepositoryTest` (Persistence), `EventTaskServiceStructureTest` (Business), `EventTaskControllerStructureTest` (RestServices).
- **Stubs used:**
  - `EventTask` with only `@Entity` + `@Id`, `@Transient` fields `description`/`assignedUser`/`event`, and no `@Table`.
  - `EventTaskRepository`.
  - `EventTaskServiceImpl` without `@Service`, and `EventTaskController` without `@RestController`/`@RequestMapping`.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Model:test --tests "com.ubb.events.domain.EventTaskTest" :Persistence:test --tests "com.ubb.events.persistence.EventTaskRepositoryTest" :Business:test --tests "com.ubb.events.implementation.EventTaskServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.EventTaskControllerStructureTest"`
- **Result:** `BUILD FAILED`, 6 of 6 tests failed and everything compiled.
  - `EventTaskTest.eventTaskHasExpectedJpaMapping` (`AssertionFailedError` at `EventTaskTest.java:22`, no `@Table`).
  - `EventTaskRepositoryTest`:
    - `savesAndReloadsTaskWithEventAndAssignedUser` (`:38`, reloaded description is null);
    - `savesTaskWithoutAssignedUser` (`:52`);
    - `rejectsNullRequiredValues` (`:61`, no `DataIntegrityViolationException`).
  - `EventTaskServiceStructureTest` (`:12`, missing `@Service`).
  - `EventTaskControllerStructureTest` (`:15`, missing `@RestController`).
- **Failure reason:** behavioral. The failures come from the missing planned mapping: no table or column mapping, description and event associations not persisted, nulls not rejected, and the Spring stereotypes missing. There were no compilation or setup errors.

### Green (after implementation)

- **Result:** the same focused command (without `--continue`) gives `BUILD SUCCESSFUL`, and all 6 focused tests pass.
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain clean test` gives `BUILD SUCCESSFUL` across Common, Model, Persistence, Business and RestServices: 105 tests, 0 failures, 0 errors, 0 skipped (counted from `*/build/test-results/test/TEST-*.xml`).

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
  - `docs/entities/Task.md` is authoritative (user decision). `EventTask` maps `public.tasks` with a required `description`, an optional `@ManyToOne User assignedUser` on `assigned_user_id`, and a required `@ManyToOne Event event` on `event_id`. There is no title, due date or status.
  - The class is named `EventTask` (user decision) to avoid confusion with the generic word "Task", while the table stays `tasks`.
  - The service and controller (`/api/event-tasks`, kebab-case like `/api/chat-messages`) are empty scaffolds, so no endpoint is exposed.
- **Alternatives considered:**
  - A backlog `title` column: rejected, not in the schema.
  - An assignee typed as `Membership`: rejected, because the schema's foreign key points to `users.user_id`.
  - A class named `Task`: rejected by the user.
- **Limitations:**
  - "The assignee is a member of the event's organization" is not enforced by the schema or this scaffold. Workflow tasks (T23/T26) must validate it in the service layer.
  - The `/api/event-tasks` scaffold path may be replaced by organization-scoped paths in the workflow tasks.

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