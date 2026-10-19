# T21 - Init Registration entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T21
- **Type:** entity
- **Owner:** Brudea Armand
- **Requirement(s):** R7
- **Use case(s):** UC-11 Register for event; UC-12 View event registrations; UC-18 Cancel event registration.
- **Workflow:** entity
- **Drive doc / ADR:** [Project requirements, use cases, architecture, ERD, API specification, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban)
- **Repo(s):** backend
- **Branch:** `T21-init-registration-entity`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/7
- **Tests:** `RegistrationTest`, `RegistrationRepositoryTest`, `RegistrationServiceStructureTest`, `RegistrationControllerStructureTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init Registration entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/Registration.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Init User entity, Init Event entity.
- Entity scaffold for Registration, with a status (REGISTERED or CANCELLED) and one row per user and event.

Dependencies from the backlog: 4, 19.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Membership.java` and `User.java` show current JPA entity conventions; `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` and its repository test show the persistence scaffold pattern. `repos/ubb-events-backend/AGENTS.md` defines backend module boundaries.
- **Related completed tasks:** No completed T19 or T20 entity task was found.
- **Constraints / assumptions:** Backend only. Per user clarification, follow `../../docs/entities/Registration.md` as the source of truth: implement only the specified `registration_id`, `event_id`, and `user_id` fields and do not add a status field or `(event_id, user_id)` unique constraint. This overrides the conflicting backlog description. Per the user's later direction, proceed while T19 is outstanding; represent `event_id` as a required scalar `Long` until the Event model is available, then align it to the Event association in T19 follow-up work.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the persistence scaffold described by `../../docs/entities/Registration.md` and align it with existing conventions. Map `public.registrations` with an identity-generated `registration_id`, a required scalar `event_id` (the Event model is not yet available), and a required `User` association mapped to `user_id`. Add the matching Spring Data repository, empty service contract/implementation, and constructor-injected scaffold controller used by the existing entity tasks. The entity specification is authoritative, so do not introduce a status field, enum, duplicate-prevention constraint, or registration business rule.

The conflicting backlog wording about status and one row per user/event is intentionally not implemented because the user confirmed `Registration.md` as the source of truth. The missing T19 dependency is deferred at the user's direction: `event_id` remains a scalar column in this task, without an ORM association or generated foreign-key enforcement. After T19 supplies `Event`, a follow-up should align the mapping to the relationship described by `Registration.md`.

### Steps

1. **Model layer:** add `Registration` with `registration_id` as a `Long` identity primary key, required scalar `eventId` mapped to `event_id`, and a non-null `@ManyToOne` association to `User` through `user_id`, following current Lombok/JPA conventions. The `Event` association is deferred until T19 supplies its model.
2. **Persistence layer:** add a `RegistrationRepository extends JpaRepository<Registration, Long>` and verify that the required event id and user association persist and reload through the existing test configuration.
3. **Business layer:** add an empty `RegistrationService` interface and `@Service` implementation, matching the current entity scaffold pattern and leaving business rules for workflow tasks.
4. **API layer:** add the constructor-injected `RegistrationController` scaffold at `/api/registrations`; expose no CRUD or workflow operations because this task does not specify an endpoint.
5. **Verification:** run the focused model, persistence, business-structure, and controller-structure tests first, then the affected backend test suite. Record the required red and green evidence in this task before implementation review.

### Files to create / modify

- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Registration.java`
- `../../repos/ubb-events-backend/Model/src/test/java/com/ubb/events/domain/RegistrationTest.java`
- `../../repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java`
- `../../repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence/RegistrationRepositoryTest.java`
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/RegistrationService.java`
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/RegistrationServiceImpl.java`
- `../../repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/RegistrationServiceStructureTest.java`
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/RegistrationController.java`
- `../../repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/RegistrationControllerStructureTest.java`

### Tests to write

- **Acceptance criterion: implementation follows the entity specification:** `RegistrationTest` checks the entity/table mapping, identity generation, required `event_id` column and `user_id` association, and confirms that no status field or uniqueness constraint is part of the mapping.
- **Acceptance criterion: relevant scaffold behavior is covered:** `RegistrationRepositoryTest` saves and reloads a registration with a required event id and persisted `User` reference and rejects null required values; structure tests verify the repository, service, implementation, and controller annotations/injection. No capacity, status-transition, duplicate-registration, or authorization tests are added because those rules are absent from `Registration.md` and belong to later workflow tasks.
- Write these tests before implementation and confirm a behavioral red result caused by the missing Registration scaffold, then implement and rerun them to green.

### Risks / open questions

- T19 (Init Event entity) remains a dependency. Per the user's instruction, T21 proceeds before T19 by mapping `event_id` as a required scalar `Long`; this omits an ORM association and generated foreign-key enforcement until Event is available. Reconcile the model relationship after T19 is completed.
- The backlog and downstream workflow descriptions still mention registration status and uniqueness, while `Registration.md` explicitly omits both. This plan follows the user-confirmed entity specification; adding those fields or constraints later requires a separate approved schema/task decision.
- No endpoint behavior is specified for this entity task, so the controller remains an empty scaffold consistent with Membership. Workflow tasks own registration operations, validation, authorization, and error handling.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `RegistrationTest`, `RegistrationRepositoryTest`, `RegistrationServiceStructureTest`, `RegistrationControllerStructureTest`.
- **Result:** No valid pre-implementation behavioral red was captured. The implementer reported that Gradle and `java -version` hung without output and terminated after repeated 30-second waits; the exact Gradle invocation has not yet been recovered. The implementation was then written before the setup problem was resolved.
- **Failure reason:** The reported failure was a toolchain startup hang, not a test failure demonstrating missing behavior. This is a TDD deviation; the red status remains unchecked. The focused tests run after implementation are green evidence only and do not satisfy the red step.

### Green (after implementation)

- **Result:** Focused tests passed after implementation:
  - `.\gradlew.bat :Model:test --tests "com.ubb.events.domain.RegistrationTest" --console=plain` — `BUILD SUCCESSFUL`.
  - `.\gradlew.bat :Persistence:test --tests "com.ubb.events.persistence.RegistrationRepositoryTest" --console=plain` — `BUILD SUCCESSFUL`.
  - `.\gradlew.bat :Business:test --tests "com.ubb.events.implementation.RegistrationServiceStructureTest" --console=plain` — `BUILD SUCCESSFUL`.
  - `.\gradlew.bat :RestServices:test --tests "com.ubb.events.controllers.RegistrationControllerStructureTest" --console=plain` — `BUILD SUCCESSFUL`.
- **Full suite of the affected repo:** `.\gradlew.bat test --console=plain` — `BUILD SUCCESSFUL` across Common, Model, Persistence, Business, and RestServices.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “Please ignore this dependency for now, proceed with the implementation, review and refine the plan if necessary. Also specify this dependency in both T19 and T21 files please” | Deferred T19 for this implementation and changed `event_id` to a required scalar field until the Event model exists; added the dependency note to T19 and T21. | User requested implementation proceed despite the dependency. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Followed `Registration.md` as authoritative. Added identity-generated `registration_id`, required scalar `event_id` pending T19's Event model, and required User association through `user_id`. Kept service and controller as empty scaffolds.
- **Alternatives considered:** A JPA Event association was deferred because T19 has not supplied the Event class. Status, enum, duplicate-prevention constraint, and workflow endpoints were excluded because they are absent from `Registration.md`.
- **Limitations:** The scalar `eventId` does not yet provide ORM relationship or generated foreign-key enforcement; reconcile it after T19 supplies the Event model. Focused and full tests pass after implementation, but the required pre-implementation behavioral red was not captured and remains a recorded TDD deviation.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
