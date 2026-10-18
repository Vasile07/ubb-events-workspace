# T4 - Init User entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T4
- **Type:** entity
- **Owner:** Kentsch Karina
- **Requirement(s):** R7
- **Use case(s):** 
- **Workflow:** entity
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T4-init-user-entity`
- **PR link(s):** 
- **Tests:** `Model/src/test/java/com/ubb/events/domain/UserTest.java`, `Persistence/src/test/java/com/ubb/events/persistence/UserRepositoryTest.java`, `Business/src/test/java/com/ubb/events/implementation/UserServiceImplTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init User entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/User.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Database creation, Backend skeleton, API conventions and error handling.
- Entity scaffold for User. The password is never exposed through the API.

Dependencies from the backlog: 1, 2, 3.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/User.java`, `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/UserRepository.java`, `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/UserService.java`, `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/UserServiceImpl.java`, and the RestServices health and empty User controller scaffolds.
- **Related completed tasks:** `T1-database-creation.md` provides the schema baseline; `docs/entities/User.md` is the field/relationship specification.
- **Constraints / assumptions:** Backend is Java 21/Spring Boot with JPA and Gradle wrapper. User maps to `public.users`: generated `bigint` id; non-null first/last name, unique non-null email, unique non-null password storage; password stored hashed and never exposed through API. Relationships: memberships, registrations, assigned tasks, and sent messages. Only an entity scaffold is in scope; no business rules.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the User scaffold from `../../docs/entities/User.md`: a JPA entity mapped to `public.users`, a persistence repository, a service interface and implementation, and an empty User controller scaffold. Defer all relationships and API endpoints to later tasks because their contracts or entity types are not yet defined. The password remains unavailable through the empty controller.

### Steps

1. **Model:** define the User JPA entity with generated `bigint` identifier, required first/last name, unique required email, and unique required password storage. Defer relationship mappings for entities that do not yet exist.
2. **Persistence:** add the User repository interface in the persistence layer, exposing only the basic data access needed by the scaffold.
3. **Business/service:** add the User service interface and a minimal implementation for basic repository delegation, without registration, hashing, authorization, or other business rules.
4. **API/controller:** add an empty `@RestController` scaffold mapped at `/api/users`. Add no endpoints or response DTO in this task.
5. **Verification:** entity, repository, and service tests cover mappings, persistence constraints, and service delegation. No endpoint-specific tests apply while the controller is empty. Record red/green evidence when the test environment is available.

### Files to create / modify

- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/User.java` (create).
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/...` (create the User repository in the existing persistence package convention).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/...` (create the User service interface and implementation in the existing business package convention).
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java` (create empty controller scaffold).
- Corresponding `src/test/java` files in Model, Persistence, Business, and RestServices modules, following package and class naming conventions.

### Tests to write

- Acceptance criterion 1: JPA mapping/repository integration tests verify User maps to `public.users`, generates its identifier, persists required scalar fields, and enforces required/unique constraints; a service test verifies delegation.
- Acceptance criterion 2: the User controller has no endpoints or response type in this task, so it cannot expose password data. No registration, password-hashing, or authorization rules are added.
- Edge cases: null required fields, duplicate email, and duplicate password storage through the configured test database.

### Risks / open questions

- The controller is an empty scaffold with no endpoints, as clarified by the user. Relationship mappings remain deferred until related entity types exist.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `UserTest`, `UserRepositoryTest`, and `UserServiceImplTest` were written first against compile stubs.
- **Result:** The focused Gradle invocation (`./gradlew.bat :Model:cleanTest :Model:test :Persistence:cleanTest :Persistence:test :Business:cleanTest :Business:test --rerun-tasks`) and a `--no-daemon` retry failed before test execution with `java.io.IOException: Unable to establish loopback connection`. The user separately reported `TestFrameworkNotAvailableException` when running `UserTest`; explicit JUnit Platform launcher dependencies have now been added, but not rerun.
- **Failure reason:** No behavioral red result could be observed. The earlier local attempts could not start Gradle's loopback connection; the user-reported JUnit framework error was addressed by adding the launcher dependency.

### Green (after implementation)

- **Result:** The user reports running the tests manually after the launcher dependency fix and says all passed. The command and output were not provided. The agent could not rerun them because Gradle's Windows selector fails to establish its loopback connection in this environment.
- **Full suite of the affected repo:** User reports all tests passed; not independently verified by the agent.
## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The controller contract and relationship mappings were unresolved. | Defer the controller. Ignore for now all entities that do not exists; relationship mappings are deferred. | As clarified by the user. |
| 2 | implementation | Constructors, getters, and setters were written manually. | Use Lombok for constructors, getter and setters on the User entity, and the service constructor. | As requested by the user. |
| 3 | implementation | The repository interface lacked the requested annotation, and the controller was deferred. | Add `@Repository` and create an empty User controller without endpoints. | As requested by the user. |
| 4 | implementation | Running `UserTest` fails with `TestFrameworkNotAvailableException`. | Add `junit-platform-launcher` as a test runtime dependency to every module using `useJUnitPlatform()`. | Gradle 9 requires the JUnit Platform launcher on the test runtime classpath. |
|   |                               |                |                              |     |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Added a JPA `User` entity mapped to `public.users`, with identity-generated `user_id`, required scalar columns, and unique constraints for email and password storage. Lombok generates entity constructors/accessors and the service constructor. Added a `JpaRepository`, a service delegating basic save, lookup, and list operations, and an empty `UserController` scaffold. Relationships and API endpoints remain deferred until later tasks.
- **Alternatives considered:** Relationship mappings and API endpoints were deferred because their entity types and contracts are not yet available.
- **Limitations:** Password hashing remains outside this entity scaffold. Verification is based on the user-reported manual test run; the agent test runner remains blocked by its Windows selector loopback failure.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green) — user-reported
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — `8ed1d74` (`T4: initialize user entity scaffold`) on `T4-init-user-entity`.
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``
