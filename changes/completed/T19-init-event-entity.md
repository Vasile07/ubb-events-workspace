# T19 - Init Event entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T19
- **Type:** entity
- **Owner:** Blaj Horatiu
- **Requirement(s):** R7
- **Use case(s):** UC-06 Create event; UC-07 Edit event and manage assigned tasks; UC-08 Publish event; UC-17 Delete event.
- **Workflow:** entity
- **Drive doc / ADR:** [Project requirements, use cases, architecture, ERD, API specification, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban)
- **Repo(s):** backend
- **Branch:** `T19-init-event-entity`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/15
- **Tests:** `EventTest`, `EventRepositoryTest`, `EventServiceStructureTest`, `EventControllerStructureTest` (new); `RegistrationTest`, `FollowUpTest`, `MessageTest`, `RegistrationRepositoryTest`, `FollowUpRepositoryTest`, `MessageRepositoryTest` (updated)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init Event entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/Event.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Database creation, Backend skeleton, API conventions and error handling, Init Organization entity, and Init FollowUp entity (T37; T37 must be completed first per the task sequencing decision).
- Entity scaffold for Event: start and end date and time, status, `deleted` flag. No creator.

Dependencies from the backlog: 1, 2, 3, 6.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/docs/coding-conventions.md` (JPA/Lombok/DI rules). Entity patterns: `Model/.../domain/Membership.java` (required `@ManyToOne` + `@JoinColumn`, `@Enumerated(EnumType.STRING)` with `MembershipStatus` enum), `Organization.java`. Temporary scalar `eventId` to replace: `Model/.../domain/Registration.java`, `FollowUp.java`, `Message.java`, with tests `RegistrationTest`, `FollowUpTest`, `MessageTest`, `RegistrationRepositoryTest`, `FollowUpRepositoryTest`, `MessageRepositoryTest`. Scaffold patterns: `Persistence/.../FollowUpRepository.java`, `Business/.../FollowUpService.java` + `implementation/FollowUpServiceImpl.java`, `RestServices/.../controllers/FollowUpController.java` and their `*StructureTest` classes. Persistence tests use H2 with `ddl-auto=create-drop` (`Persistence/src/test/resources/application-test.properties`).
- **Related completed tasks:** T1 (schema: `date`, `start_time`, nullable `end_time`, status DRAFT/PUBLISHED/CANCELED, no `deleted` or creator field), T6 (Organization), T21 (Registration), T37 (FollowUp), T40 (Message; also uses a temporary scalar `eventId`).
- **Spec:** `docs/entities/Event.md`. No `Category` entity exists in the backend and no task in `changes/` creates it, although `events.category_id` is a required foreign key to `categories`.
- **Constraints / assumptions:** T21 (Init Registration entity) and T37 (Init FollowUp entity) are sequenced before T19. T37 must be completed before T19 begins, even though this is an added task sequencing dependency beyond the original backlog dependencies. T21 and T37 use a required scalar `event_id` temporarily because the Event model is not yet available. When starting T19, its worker must inspect the Registration and FollowUp entities and their tests, then replace both temporary scalar mappings with required ORM associations to the Event model using `event_id`; update the corresponding tests and verify both relationships as part of T19.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the Event persistence scaffold from `docs/entities/Event.md` (user decision: Event.md is authoritative over the backlog wording "start and end date and time ... deleted flag"). Map `public.events` with identity `event_id`, a required `@ManyToOne Organization`, a required **scalar** `Long categoryId` (no Category entity exists yet; same temporary workaround as T21/T37/T40), `title`, `description`, `date`, `startTime`, nullable `endTime`, `location`, `capacity`, and a required `status` stored as a string enum `EventStatus { DRAFT, PUBLISHED, CANCELED }`. No `deleted` flag and no creator field.

Then replace the temporary scalar `eventId` in `Registration`, `FollowUp` and `Message` with a required `@ManyToOne Event event` joined on `event_id`, and update their tests (user decision: include Message too).

Add the empty repository, service interface/implementation and controller scaffold in the same style as FollowUp. No business rules, lifecycle transitions or authorization (those belong to T22 and later workflow tasks).

Code style follows the existing entities exactly: `@Entity` + `@Table(name, schema = "public")`, Lombok `@Getter @Setter @NoArgsConstructor @AllArgsConstructor`, `@Column(name = ..., nullable = ...)` on every field, enums with `@Enumerated(EnumType.STRING)` in their own one-line file like `MembershipStatus`, reflection-based mapping tests, `@DataJpaTest` repository tests, and `*StructureTest` classes for the service and controller.

Alternatives: a minimal `Category` entity was rejected because it is outside T19's scope. The backlog's `deleted` flag was rejected because it does not exist in the schema; `CANCELED` represents deletion. T22's backlog text still mentions a `deleted` flag, so the team should reconcile T22 with Event.md (worth recording in Drive/ADR).

### Steps

1. **Model:** add `EventStatus` enum (`DRAFT`, `PUBLISHED`, `CANCELED`).
2. **Model:** add `Event` entity:
   - `id` (`Long`, `event_id`, identity)
   - `organization` (`@ManyToOne(optional = false)`, `@JoinColumn(name = "organization_id", nullable = false)`)
   - `categoryId` (`Long`, `category_id`, not null), temporary scalar, see the risks below
   - `title`, `description` (`String`, not null)
   - `date` (`LocalDate`, not null), `startTime` (`LocalTime`, `start_time`, not null), `endTime` (`LocalTime`, `end_time`, nullable)
   - `location` (`String`, nullable), `capacity` (`Long`, nullable)
   - `status` (`EventStatus`, `@Enumerated(EnumType.STRING)`, not null)
3. **Model:** in `Registration`, `FollowUp`, `Message` replace `Long eventId` with `@ManyToOne(optional = false) @JoinColumn(name = "event_id", nullable = false) Event event`. Field order and other fields stay unchanged.
4. **Persistence:** add `EventRepository extends JpaRepository<Event, Long>` (`@Repository`, like `FollowUpRepository`).
5. **Business:** add empty `EventService` interface and `@Service EventServiceImpl`.
6. **API:** add `EventController` (`@RestController`, `@RequestMapping("/api/events")`, constructor-injected `EventService`, no handlers).
7. **Verification:** tests first, behavioral red, implement, focused green, then the full backend suite (`.\gradlew.bat test`). Record everything in the Test log.

### Files to create / modify

Backend (`repos/ubb-events-backend/`):

- Create `Model/src/main/java/com/ubb/events/domain/EventStatus.java`
- Create `Model/src/main/java/com/ubb/events/domain/Event.java`
- Modify `Model/src/main/java/com/ubb/events/domain/Registration.java`, `FollowUp.java`, `Message.java`
- Create `Model/src/test/java/com/ubb/events/domain/EventTest.java`
- Modify `Model/src/test/java/com/ubb/events/domain/RegistrationTest.java`, `FollowUpTest.java`, `MessageTest.java`
- Create `Persistence/src/main/java/com/ubb/events/persistence/EventRepository.java`
- Create `Persistence/src/test/java/com/ubb/events/persistence/EventRepositoryTest.java`
- Modify `Persistence/src/test/java/com/ubb/events/persistence/RegistrationRepositoryTest.java`, `FollowUpRepositoryTest.java`, `MessageRepositoryTest.java`
- Create `Business/src/main/java/com/ubb/events/EventService.java`, `Business/src/main/java/com/ubb/events/implementation/EventServiceImpl.java`
- Create `Business/src/test/java/com/ubb/events/implementation/EventServiceStructureTest.java`
- Create `RestServices/src/main/java/com/ubb/events/controllers/EventController.java`
- Create `RestServices/src/test/java/com/ubb/events/controllers/EventControllerStructureTest.java`

### Tests to write

Acceptance criteria → tests:

- **AC1: implemented according to the specification (Event.md):**
  - `EventTest.eventHasExpectedJpaMapping`: entity/table `public.events`, identity id, `organization` `@ManyToOne` on non-null `organization_id`, every column name, type and nullability from Event.md, status `EnumType.STRING`, and no `deleted` or creator field.
  - `EventTest.statusValuesMatchSpecification`: `EventStatus.values()` is exactly `DRAFT, PUBLISHED, CANCELED`.
  - `RegistrationTest`, `FollowUpTest`, `MessageTest`: the `event` field is `@ManyToOne` with a non-null `event_id` join column, and `eventId` no longer exists.
- **AC2: scaffold behavior and constraints are covered:**
  - `EventRepositoryTest.savesAndReloadsEventWithOrganizationAndNullableFields`: save with an Organization, null `endTime`/`location`/`capacity`, then reload and check the values.
  - `EventRepositoryTest.rejectsNullRequiredValues`: `DataIntegrityViolationException` for null organization, categoryId, title, description, date, startTime and status.
  - `RegistrationRepositoryTest`, `FollowUpRepositoryTest`, `MessageRepositoryTest`: save and reload with a persisted Event (checking `getEvent().getId()`), and reject a null event.
  - `EventServiceStructureTest`, `EventControllerStructureTest`: same assertions as the FollowUp structure tests.
- Expected red: the stubs (an empty `Event`/`EventStatus`, and the `event` field added without mapping annotations) compile, so the mapping and repository tests fail on assertions or integrity checks, not on compilation.

### Risks / open questions

- **Category (follow-up needed, do not forget):** `category_id` is mapped as a temporary scalar `Long categoryId` because no Category entity or task exists. There is no ORM association or generated FK until a future task adds the `Category` entity (`docs/entities/Category.md`) and replaces `categoryId` with `@ManyToOne Category category`. The team needs to create that task.
- **T22 mismatch:** T22's backlog text derives CANCELED from a `deleted` flag. Event.md and T1 store CANCELED in `status` with no flag. The team should reconcile this before T22 starts.
- **`date` column name:** `date` is an SQL keyword. If H2 (tests) rejects it, quote the column name in the mapping (`@Column(name = "\"date\"")`). This will be confirmed by `EventRepositoryTest` during red/green.
- Changing `Registration`/`FollowUp`/`Message` constructors (`Long` → `Event`) affects only their tests. No production code constructs them yet (checked by grep).

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventTest`, `RegistrationTest`, `FollowUpTest`, `MessageTest` (Model); `EventRepositoryTest`, `RegistrationRepositoryTest`, `FollowUpRepositoryTest`, `MessageRepositoryTest` (Persistence); `EventServiceStructureTest` (Business); `EventControllerStructureTest` (RestServices).
- **Stubs used:**
  - `Event` with only `@Entity` + `@Id`, all other fields `@Transient`, and no `@Table`.
  - The `event` field in `Registration`/`FollowUp`/`Message` marked `@Transient`, replacing `eventId`.
  - `EventStatus` with its three constants, which the other tests need in order to compile.
  - `EventRepository`.
  - `EventServiceImpl` without `@Service`, and `EventController` without `@RestController`/`@RequestMapping`.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Model:test --tests "com.ubb.events.domain.EventTest" --tests "com.ubb.events.domain.RegistrationTest" --tests "com.ubb.events.domain.FollowUpTest" --tests "com.ubb.events.domain.MessageTest" :Persistence:test --tests "com.ubb.events.persistence.EventRepositoryTest" --tests "com.ubb.events.persistence.RegistrationRepositoryTest" --tests "com.ubb.events.persistence.FollowUpRepositoryTest" --tests "com.ubb.events.persistence.MessageRepositoryTest" :Business:test --tests "com.ubb.events.implementation.EventServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.EventControllerStructureTest"`
- **Result:** `BUILD FAILED`, 15 of 16 tests failed and everything compiled.
  - Model: 4 of 5 failed. `eventHasExpectedJpaMapping` (`AssertionFailedError` at `EventTest.java:26`), plus the three `*HasExpectedJpaMapping` tests for Registration, FollowUp and Message.
  - Persistence: 9 of 9 failed. `AssertionFailedError` in every test except `savesAndReloadsEventWithOrganizationAndNullableFields`, which threw a `NullPointerException` at `EventRepositoryTest.java:37` because the reloaded organization is null.
  - `EventServiceStructureTest` and `EventControllerStructureTest`: `AssertionFailedError` on the missing annotations.
  - `statusValuesMatchSpecification` passed, because the enum constants had to exist for compilation.
- **Failure reason:** behavioral. All failures come from the missing planned mapping:
  - no `@Table` or `@Column` mapping on Event;
  - the event associations are not persisted, so the reloaded `getEvent()` is null and null required values are not rejected (no `DataIntegrityViolationException`);
  - the Spring stereotype annotations are missing.

  There were no compilation or setup errors.
- **Note:** a first red run used the existing reload pattern without clearing the persistence context. The save/reload tests passed then, because `findById` returned the cached instance. `TestEntityManager.clear()` was added before each reload so the tests check what is actually stored, and the red run above was repeated after that change.

### Green (after implementation)

- **Result:** same focused command as Red → `BUILD SUCCESSFUL`, all 16 focused tests pass. The `date` column name works on H2 without quoting.
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain clean test` → `BUILD SUCCESSFUL` across Common, Model, Persistence, Business and RestServices: 99 tests, 0 failures, 0 errors, 0 skipped (counted from `*/build/test-results/test/TEST-*.xml`).

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
  - `docs/entities/Event.md` is authoritative over the backlog wording, so there is no `deleted` flag and no creator field. `CANCELED` in `status` represents a canceled or deleted event.
  - `status` is a string-stored enum `EventStatus`, following the `MembershipStatus` pattern.
  - `date`, `start_time` and `end_time` map to `LocalDate`/`LocalTime`, matching the PostgreSQL `date` and `time without time zone` types.
  - Organization is a required `@ManyToOne`.
  - `Registration`, `FollowUp` and `Message` now hold a required `@ManyToOne Event` on `event_id` instead of the temporary scalar `eventId` left by T21, T37 and T40.
  - The service and controller (`/api/events`) are empty scaffolds with no endpoints, so no unprotected CRUD endpoint is exposed.
  - Repository tests clear the persistence context before reloading (`TestEntityManager.clear()`), so they check what is actually stored in the database rather than the first-level cache.
- **Alternatives considered:**
  - Backlog fields (start/end date-time and a `deleted` flag): rejected because they don't match the schema.
  - A minimal `Category` entity: rejected as outside T19's scope.
  - Keeping Message as a scalar: rejected by the user, so all three are now associations.
  - `@ManyToOne(fetch = LAZY)`: not used, because the existing associations use the default (eager), and the style was kept consistent.
- **Limitations:**
  - Only the entity scaffold exists. There are no business rules (lifecycle transitions, capacity) and no endpoints; T22 and the workflow tasks own those.
  - The T22 backlog text still assumes a `deleted` flag and needs reconciling with Event.md.
  - TODO (follow-up): `Event.categoryId` is a temporary scalar for `category_id`, with no Category entity, ORM association or FK mapping. A future task must add `Category` and replace the scalar with `@ManyToOne Category`.

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
