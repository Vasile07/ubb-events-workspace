# T44 - Init Announcement entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T44
- **Type:** entity
- **Owner:** Cernautan Teofan
- **Requirement(s):** R7
- **Use case(s):**
- **Workflow:** entity
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T44-init-announcement-entity`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/17
- **Tests:** `AnnouncementTest`, `AnnouncementRepositoryTest`, `AnnouncementServiceStructureTest`, `AnnouncementControllerStructureTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init Announcement entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/Announcement.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Init User entity, Init Event entity.
- Entity scaffold for Announcement (event, title, and content). Team clarification: the database has no author or date/time fields.

Dependencies from the backlog: 4, 19.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Event.java`, `Message.java`, and `User.java`; scaffold patterns in `Persistence/.../MessageRepository.java`, `Business/.../MessageService.java`, `Business/.../implementation/MessageServiceImpl.java`, and `RestServices/.../controllers/MessageController.java`; existing mapping tests in `Model/src/test/java/com/ubb/events/domain/MessageTest.java`.
- **Related completed tasks:** `T4-init-user-entity.md`; no completed T19 task file was found. The current backend master includes Event and User entities.
- **Constraints / assumptions:** Backend repo was clean on updated `master` (`3c2a576`) before creating `T44-init-announcement-entity` after Gate 1 approval. `docs/entities/Announcement.md` maps only `announcement_id`, `event_id`, `title`, and `content`. The team confirmed that the database has no author or date/time fields, so those properties are excluded.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the backend persistence scaffold following the existing `Message`/`FollowUp` patterns: a JPA `Announcement` entity for the documented `public.announcements` columns and an `AnnouncementRepository` extending `JpaRepository`. Add empty `AnnouncementService`/`AnnouncementServiceImpl` types and a dependency-injected controller with the task's basic announcement route, without introducing workflow business rules or CRUD handlers. Team clarification confirms the database has no author or date/time fields, so the entity will contain only the documented announcement columns.

The alternative is to add author/date-time properties despite their absence from the database schema; those properties are removed to keep the entity aligned with the documented table.

### Steps

1. **Persistence/model:** Keep `Announcement` limited to generated identity `id`, mandatory `Event event` relation through `event_id`, and mandatory `title`/`content` columns matching `docs/entities/Announcement.md`; remove the provisional author/date-time properties.
2. **Persistence:** Add `AnnouncementRepository` as a Spring `@Repository` extending `JpaRepository<Announcement, Long>`; do not add query methods or business rules.
3. **Service:** Add an empty `AnnouncementService` interface and `@Service` `AnnouncementServiceImpl`, matching the existing scaffold structure and leaving lifecycle/authorization behavior to workflow tasks.
4. **API:** Add a basic `AnnouncementController` with `@RestController`, the announcement root mapping, and constructor injection of `AnnouncementService`; expose no handlers until an API workflow specifies request/response contracts and authorization.
5. **Verification:** Write and run mapping, repository constraint, service-structure, and controller-structure tests before implementation, record a behavioral red result, then implement and run the focused tests followed by the backend suite.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Announcement.java` (create)
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/AnnouncementRepository.java` (create)
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/AnnouncementService.java` (create)
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/AnnouncementServiceImpl.java` (create)
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/AnnouncementController.java` (create)
- Corresponding `src/test/java` files in Model, Persistence, Business, and RestServices for the scaffold described below (create).

### Tests to write

- `AnnouncementTest`: verifies `@Entity`, `public.announcements`, identity-generated `announcement_id`, non-null `event`/`title`/`content`, the `Event` `@ManyToOne`/`event_id` mapping, that no `eventId` scalar replaces the relationship, and that no author/date-time properties remain. This covers the entity portion of acceptance criterion 1.
- `AnnouncementRepositoryTest` (`@DataJpaTest`): saves and reloads an announcement with its event, title, and content, and rejects null required values. This covers persistence behavior and applicable validation/error cases for criteria 1 and 2.
- `AnnouncementServiceStructureTest`: verifies the service implementation is annotated with `@Service`, implements the interface, and exposes no operations. No business-rule or authorization tests apply to this entity scaffold.
- `AnnouncementControllerStructureTest`: verifies `@RestController`, the selected announcement root route, constructor injection, and no unapproved handlers. API authorization/error behavior is deferred because no endpoint contract exists yet.
- Run focused tests first, then the complete backend Gradle test suite; record exact commands and outcomes in the Test log.

### Risks / open questions

- The original backlog mentioned author and date/time, but the team confirmed the database has no such fields. The implementation follows the entity specification and excludes these properties.
- The controller route is not specified by the task. Use the existing singular resource scaffold convention selected during implementation and update it if the API specification supplies a different path; do not add handlers or authorization annotations without a workflow requirement.
- No business behavior, validation beyond database nullability, or authorization is defined for announcements. Workflow tasks must add those rules and replace the scaffold endpoints before exposing CRUD operations.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `AnnouncementTest`, `AnnouncementRepositoryTest`, `AnnouncementServiceStructureTest`, `AnnouncementControllerStructureTest`.
- **Command:** `./gradlew.bat :Model:test --tests "com.ubb.events.domain.AnnouncementTest" :Persistence:test --tests "com.ubb.events.persistence.AnnouncementRepositoryTest" :Business:test --tests "com.ubb.events.implementation.AnnouncementServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.AnnouncementControllerStructureTest"`
- **Result:** `:Model:test` reached the test phase and failed behaviorally: `AnnouncementTest > mapsAnnouncementColumnsAndRequiredRelationships() FAILED` with `AssertionFailedError` at `AnnouncementTest.java:15`.
- **Failure reason:** The provisional compile stubs lacked the planned JPA entity/table mapping, demonstrating missing Announcement persistence behavior. Gradle execution initially required approved access to the user-level wrapper cache; the rerun completed and produced the behavioral failure.

- **Update (remove author/date-time):** Changed `AnnouncementTest` to assert that `author` and `dateTime` fields are absent while the implementation still contained them. Command: `./gradlew.bat :Model:test --tests "com.ubb.events.domain.AnnouncementTest"`. Result: failed behaviorally in `AnnouncementTest > mapsAnnouncementColumnsAndRequiredRelationships()` with `AssertionFailedError` at `AnnouncementTest.java:44`; the failure demonstrated that the provisional fields were still present.

### Green (after implementation)

- **Result:** Focused command `./gradlew.bat :Model:test --tests "com.ubb.events.domain.AnnouncementTest" :Persistence:test --tests "com.ubb.events.persistence.AnnouncementRepositoryTest" :Business:test --tests "com.ubb.events.implementation.AnnouncementServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.AnnouncementControllerStructureTest"` completed successfully after structural assertion fixes (20 actionable tasks).
- **Full suite of the affected repo:** `./gradlew.bat test` completed successfully after structural assertion fixes (22 actionable tasks).

- **Update (remove author/date-time):** Focused command `./gradlew.bat :Model:test --tests "com.ubb.events.domain.AnnouncementTest" :Persistence:test --tests "com.ubb.events.persistence.AnnouncementRepositoryTest" :Business:test --tests "com.ubb.events.implementation.AnnouncementServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.AnnouncementControllerStructureTest"` completed successfully (20 actionable tasks). Full backend command `./gradlew.bat test` completed successfully (22 actionable tasks).

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | `docs/entities/Announcement.md` does not include the author and date/time mentioned in the backlog. | Include author and date/time as provisional additions without definitive fields or mappings. | “adding them without definitive fields or mappings for now for the possibility of easily removing it later if necessary.” |
| 2 | implementation | The team says the database has no author and datetime. | Remove the author and date/time properties from the Announcement entity. | “talked right now with the team and they say the data base has no author and datetime. We can remove them.” |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** The entity uses only the documented `event`, `title`, and `content` persistence mappings. Author/date-time properties were removed after team clarification that the database has no such fields. The empty service and constructor-injected controller at `/api/announcements` remain scaffolds.
- **Alternatives considered:** Keeping non-persistent author/date-time placeholders was rejected after the team confirmed those fields do not exist in the database. CRUD handlers and business rules remain deferred to workflow tasks.
- **Limitations:** The controller has no handlers or authorization because this task defines only the scaffold.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — `aa008d2` (initial scaffold) and `d753646` (remove absent author/date-time fields) on `T44-init-announcement-entity`.
- [x] PR created — `https://github.com/Vasile07/ubb-events-backend/pull/17`.
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability; user confirmed Use Case(s) and Drive doc/ADR may remain blank)
- [x] Moved to `changes/completed/`
