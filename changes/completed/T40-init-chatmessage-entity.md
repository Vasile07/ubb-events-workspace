# T40 - Init Message entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T40
- **Type:** entity
- **Owner:** Cernautan Teofan
- **Requirement(s):** R7
- **Use case(s):** N/A for now
- **Workflow:** entity
- **Drive doc / ADR:** N/A for now
- **Repo(s):** backend
- **Branch:** `T40-init-chatmessage-entity`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/9
- **Tests:** `MessageTest`, `MessageRepositoryTest`, `MessageServiceStructureTest`, `MessageControllerStructureTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init Message entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/Message.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Init User entity, Init Event entity.
- Entity scaffold for Message (event, author, text, time). There is no Conversation entity: the conversation of an event is the set of its messages.

Dependencies from the backlog: 4, 19.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `Model/src/main/java/com/ubb/events/domain/User.java`; `Model/src/main/java/com/ubb/events/domain/Registration.java` (similar mapping, scalar event id and User relation); `Persistence/src/main/java/com/ubb/events/persistence/UserRepository.java`; `Persistence/src/test/java/com/ubb/events/persistence/UserRepositoryTest.java`; `Persistence/src/test/java/com/ubb/events/persistence/RegistrationRepositoryTest.java`. No Event or Message entity exists in the backend checkout.
- **Related completed tasks:** `changes/completed/T4-init-user-entity.md` documents the User entity scaffold and its conventions. No matching completed Event entity task was found.
- **Constraints / assumptions:** Project context specifies Java 21, Spring Boot, JPA, Gradle wrapper, and layered architecture. The corrected specification is `docs/entities/Message.md`, which defines generated `message_id`, required `event_id`, required `sender_id`, and required `content`; it does not define a timestamp field. The backend checkout is clean on `master` and was fetched/fast-forward checked; no Event entity is currently present.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the Message persistence scaffold from `docs/entities/Message.md` across the existing backend layers. Add a JPA domain entity mapped to `public.messages`, with an identity-generated `message_id`, required scalar `event_id`, a required `User` association for `sender_id`, and required `content`. Follow the existing Registration scaffold conventions: repository and service/controller types remain structural and contain no message, conversation, authorization, or other business rules. Because the Event entity is not present, keep `event_id` as a required scalar and defer an Event association. Use the backlog name `Message` for Java types while preserving the database table/column names from the specification.

### Steps

1. **Model:** create the `Message` JPA entity in the Model module, mapped to `public.messages`, with identity-generated `Long id` at `message_id`, non-null `Long eventId` at `event_id`, non-null `User sender` via `@ManyToOne`/`@JoinColumn(name = "sender_id")`, and non-null `String content`. Do not add a timestamp or Conversation entity.
2. **Persistence:** create the `MessageRepository` Spring Data repository for `Message`, matching the existing repository package and annotation conventions.
3. **Business/service:** create an empty `MessageService` interface and `MessageServiceImpl` annotated with `@Service`, matching the structural scaffold used for Registration and leaving business rules for workflow tasks.
4. **API/controller:** create an empty `MessageController` annotated with `@RestController`, mapped to `/api/chat-messages`, and constructor-injected with `MessageService`; add no handlers or unprotected CRUD behavior.
5. **Verification:** write tests before implementation, run focused tests to obtain a behavioral red result, implement the scaffold, run focused tests to green, then run the affected backend test suite and record exact evidence in the Test log.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Message.java` (create).
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MessageRepository.java` (create).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MessageService.java` (create).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/MessageServiceImpl.java` (create).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MessageController.java` (create).
- Corresponding tests under `Model/src/test/java/com/ubb/events/domain/`, `Persistence/src/test/java/com/ubb/events/persistence/`, `Business/src/test/java/com/ubb/events/implementation/`, and `RestServices/src/test/java/com/ubb/events/controllers/`.

### Tests to write

- Acceptance criterion 1: `MessageTest` verifies the entity annotations, table/column mappings, identity id generation metadata, required scalar fields, and `sender` User association; `MessageRepositoryTest` saves and reloads a message with generated id, event id, sender, and content.
- Acceptance criterion 2: `MessageRepositoryTest` verifies null `event_id`, null `sender_id`, and null `content` are rejected by the database; service and controller structure tests verify `@Service`/`@RestController`, the root mapping, constructor dependency, and absence of handlers. No authorization or workflow rule is introduced by this entity scaffold, so no additional business-rule tests apply.
- Edge cases: persistence tests cover each required field independently and verify that a persisted sender is reloaded through the User relationship. The repository test uses the existing test profile and real database integration setup.

### Risks / open questions

- The agreed backlog and database naming is `Message`; Java types use `Message` and the table/column names remain `messages`/`message_id`.
- `event_id` remains a scalar until the Event entity task exists; a later task may replace it with a relationship without changing this task's required database column.
- The specification contains no timestamp, so no time field is planned.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Model/src/test/java/com/ubb/events/domain/MessageTest.java`; focused Gradle command also selected the planned Persistence, Business, and RestServices tests.
- **Result:** `./gradlew :Model:test --tests com.ubb.events.domain.MessageTest :Persistence:test --tests com.ubb.events.persistence.MessageRepositoryTest :Business:test --tests com.ubb.events.implementation.MessageServiceStructureTest :RestServices:test --tests com.ubb.events.controllers.MessageControllerStructureTest` failed in `:Model:test`: `MessageTest > messageHasExpectedJpaMapping()` failed at line 20 with `AssertionFailedError`.
- **Failure reason:** The test reached the entity mapping assertions and found the stub `Message` class was missing the required JPA `@Entity` mapping. This is a behavioral failure caused by the planned entity functionality being absent; Gradle compilation and test setup succeeded.

### Green (after implementation)

- **Result:** Focused command passed: `./gradlew :Model:test --tests com.ubb.events.domain.MessageTest :Persistence:test --tests com.ubb.events.persistence.MessageRepositoryTest :Business:test --tests com.ubb.events.implementation.MessageServiceStructureTest :RestServices:test --tests com.ubb.events.controllers.MessageControllerStructureTest` (`BUILD SUCCESSFUL`, 20 actionable tasks). The entity mapping, repository persistence and null constraint checks, service structure, and controller structure tests all passed.
- **Full suite of the affected repo:** `./gradlew test` passed in `repos/ubb-events-backend` (`BUILD SUCCESSFUL`, 22 actionable tasks; 6 executed, 16 up-to-date).

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | the task wrote wrong the specification file, instead of ChatMessage.md its Message.md | use Message.md | The user corrected the specification file path. |
| 2 | plan | make the Message be class. The team agreed on making in the backlog to be "Message" instead of "ChatMessage" to be consistent with the database. | Use `Message` for the Java entity class and related Java types. | To be consistent with the database. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Added a JPA `Message` entity mapped to `public.messages` with identity-generated `message_id`, required scalar `event_id`, required `User` sender relation on `sender_id`, and required `content`. Added structural repository, service, implementation, and controller types. The controller uses the approved `/api/chat-messages` root mapping and has no handlers.
- **Alternatives considered:** An Event association was not added because the Event entity is not present in this backend checkout; `eventId` remains a scalar as approved. A timestamp and Conversation model were not added because they are absent from `Message.md` and the approved plan.
- **Limitations:** This task provides persistence and structural API scaffolding only. Message business rules, authorization, handlers, and Event relationship behavior remain for later workflow/entity tasks.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — `4756460` (`T40: initialize message entity scaffold`) on `T40-init-chatmessage-entity`.
- [x] PR created — [#9](https://github.com/Vasile07/ubb-events-backend/pull/9).
- [x] PR reviewed by teammates — user confirmed.
- [x] **Gate 3:** merged to `master` in PR #9; user confirmed.
- [x] Metadata complete (traceability) — user confirmed `N/A for now` for Use case(s) and Drive doc / ADR.
- [x] Moved to `changes/completed/`











