# T37 - Init FollowUp entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T37
- **Type:** entity
- **Owner:** Brudea Armand
- **Requirement(s):** R7
- **Use case(s):** UC-13 Create event follow-up.
- **Workflow:** entity
- **Drive doc / ADR:** https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban
- **Repo(s):** backend
- **Branch:** `T37-init-followup-entity`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/10
- **Tests:** `FollowUpTest`, `FollowUpRepositoryTest`, `FollowUpServiceStructureTest`, `FollowUpControllerStructureTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init FollowUp entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/FollowUp.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Init Event entity.
- Entity scaffold for FollowUp. Fields to confirm: summary and feedback (the team has not decided them yet).

Dependencies from the backlog: 19.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the approved entity specification and plan.
- [x] Relevant scaffold constraints and behavior are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Registration.java` and `Membership.java` show current JPA identity and required relationship conventions; `Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java` and `RestServices/src/main/java/com/ubb/events/controllers/RegistrationController.java` show the repository and empty controller patterns. `repos/ubb-events-backend/docs/coding-conventions.md` defines JPA and layering guidance.
- **Related completed tasks:** T21 (Init Registration entity) provides a backend entity scaffold precedent; its task notes similarly deferred its event relationship while T19 is outstanding.
- **Constraints / assumptions:** Backend only; task type is entity. `Model` currently has no `Event` entity, and T19 (Init Event entity) remains in `changes/todo/`. The user confirmed that `docs/entities/FollowUp.md` is authoritative for required `content` and `event_id`. Per the user direction to proceed like T21, T37 uses a required scalar `Long eventId` temporarily; the T19 worker will later replace the temporary mapping with the Event association in both FollowUp and Registration.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the FollowUp persistence scaffold from `docs/entities/FollowUp.md` as a backend entity task. Following the temporary workaround used by T21 while T19 is unfinished, map `public.followups` with an identity-generated `followup_id`, required scalar `eventId` mapped to `event_id`, and required `content`. Add the repository, empty service contract/implementation, and basic controller scaffold following existing entity conventions; business behavior and authorization remain outside this entity task.

The entity specification is authoritative for `content` and `event_id`, replacing the undecided backlog names `summary` and `feedback`. The scalar `eventId` is temporary: after T19 supplies `Event`, its worker must align the FollowUp and Registration mappings to the Event relationships and restore ORM/foreign-key mapping.

### Steps

1. **Model layer:** add `FollowUp` mapped to `public.followups`, with identity-generated `Long id` mapped to `followup_id`, required scalar `Long eventId` mapped to non-null `event_id`, and required `String content` mapped to `content`, following the current JPA/Lombok conventions and T21's temporary dependency workaround.
2. **Persistence layer:** add `FollowUpRepository extends JpaRepository<FollowUp, Long>` and verify persistence and reload of the required scalar event id and content through the existing repository test setup.
3. **Business layer:** add an empty `FollowUpService` interface and `@Service` implementation. Do not add follow-up business rules, validation beyond entity nullability, or authorization in this scaffold.
4. **API layer:** add a constructor-injected `FollowUpController` scaffold at `/api/followups`, following the existing `RegistrationController` pattern. Expose no CRUD or workflow operations because the task specifies only entity initialization.
5. **Dependency follow-up:** record that the T19 worker must later replace the scalar `eventId` in FollowUp and Registration with the canonical Event associations once the Event model exists.
6. **Verification:** write the focused model, repository, service-structure, and controller-structure tests first; confirm a behavioral red result, then implement and run the focused tests followed by the affected backend suite. Record exact commands and results in the Test log.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/FollowUp.java`
- `repos/ubb-events-backend/Model/src/test/java/com/ubb/events/domain/FollowUpTest.java`
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/FollowUpRepository.java`
- `repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence/FollowUpRepositoryTest.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/FollowUpService.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/FollowUpServiceImpl.java`
- `repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/FollowUpServiceStructureTest.java`
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/FollowUpController.java`
- `repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/FollowUpControllerStructureTest.java`

### Tests to write

- **Acceptance criterion: implementation follows the backlog/entity specification:** `FollowUpTest` checks the entity/table mapping, identity generation, required scalar `eventId`/`event_id` column, required `content` column, and absence of the undecided `summary` and `feedback` fields.
- **Acceptance criterion: relevant scaffold behavior is covered:** `FollowUpRepositoryTest` saves and reloads a FollowUp with a scalar event id and content and rejects null required values; service and controller structure tests verify the repository/service/controller wiring and annotations. No Event association test is added before T19 supplies the Event model, and no workflow, authorization, capacity, or follow-up lifecycle rules are tested because they are not specified for this entity scaffold.
- Write all tests before implementation and confirm the failures are behavioral failures caused by the missing FollowUp scaffold, not compilation or environment failures.

### Risks / open questions

- T19 is still in `changes/todo/`, so this task uses the same temporary scalar `eventId` workaround as T21. The T19 worker must align FollowUp and Registration associations after the Event model exists, including ORM and foreign-key mapping.
- The backlog mentions undecided `summary` and `feedback` fields, while `docs/entities/FollowUp.md` defines `content` and `event_id`; the user resolved this in favor of the entity specification.
- No endpoint behavior or business rules are specified, so the controller and service remain scaffolds. Any follow-up creation, editing, publication, or authorization behavior belongs to a later workflow task.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `FollowUpTest.followUpHasExpectedJpaMapping()`.
- **Result:** `.\gradlew.bat --no-daemon --console=plain :Model:test --tests com.ubb.events.domain.FollowUpTest` — failed as expected. One test completed; `FollowUpTest > followUpHasExpectedJpaMapping()` failed with `AssertionFailedError` at `FollowUpTest.java:18`.
- **Failure reason:** The assertion that `FollowUp` is annotated with JPA `@Entity` failed because the pre-implementation compile stub had no entity mapping. This was a behavioral red for the missing planned mapping, not a compilation or setup failure. The sandbox invocation hung; the same command succeeded outside the sandbox.

### Green (after implementation)

- **Result:** Focused tests passed:
  - `.\gradlew.bat --no-daemon --console=plain :Model:test --tests com.ubb.events.domain.FollowUpTest` — `BUILD SUCCESSFUL`.
  - `.\gradlew.bat --no-daemon --console=plain :Persistence:test --tests com.ubb.events.persistence.FollowUpRepositoryTest` — `BUILD SUCCESSFUL`.
  - `.\gradlew.bat --no-daemon --console=plain :Business:test --tests com.ubb.events.implementation.FollowUpServiceStructureTest` — `BUILD SUCCESSFUL`.
  - `.\gradlew.bat --no-daemon --console=plain :RestServices:test --tests com.ubb.events.controllers.FollowUpControllerStructureTest` — `BUILD SUCCESSFUL`.
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain test` — `BUILD SUCCESSFUL` across Common, Model, Persistence, Business, and RestServices.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “T19 isn't ready. So we need to proceed just like we did with T21.” | Removed the T19 completion prerequisite and changed FollowUp to a required scalar `eventId` for now; deferred the Event association alignment to the T19 worker. | Proceed with the entity scaffold while the Event model is unavailable, matching the approved T21 workaround. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Follow `docs/entities/FollowUp.md` for `content` and `event_id`. Since T19 has not supplied `Event`, map `event_id` as a required scalar `Long eventId`, matching the approved T21 workaround. Keep the service and controller as empty scaffolds.
- **Alternatives considered:** A JPA `Event` association was deferred until T19 creates the Event model. The backlog names `summary` and `feedback` were not used because the user selected the entity specification as authoritative.
- **Limitations:** The scalar `eventId` does not provide an ORM association or generated foreign-key mapping. The T19 worker must update both FollowUp and Registration to use Event associations and update their tests.

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
