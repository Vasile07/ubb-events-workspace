# T7 - Init Membership entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T7
- **Type:** entity
- **Owner:** Kentsch Karina
- **Requirement(s):** R7
- **Use case(s):** 
- **Workflow:** entity
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T7-init-membership-entity`
- **PR link(s):** 
- **Tests:** `Model/src/test/java/com/ubb/events/domain/MembershipTest.java`; `Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java`; `Business/src/test/java/com/ubb/events/implementation/MembershipServiceStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerStructureTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init Membership entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/Membership.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Init User entity, Init Organization entity.
- Entity scaffold for Membership: user, organization, role (MEMBER or MANAGER), active status.

Dependencies from the backlog: 4, 6.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/User.java` and `Organization.java` show the existing JPA entity style; `Persistence/src/main/java/com/ubb/events/persistence/UserRepository.java`, `Business/src/main/java/com/ubb/events/UserService.java` and `implementation/UserServiceImpl.java`, and `RestServices/src/main/java/com/ubb/events/controllers/UserController.java` show the current scaffold pattern. Related tests are `Model/src/test/java/com/ubb/events/domain/OrganizationTest.java` and `Persistence/src/test/java/com/ubb/events/persistence/OrganizationRepositoryTest.java`.
- **Related completed tasks:** `T4-init-user-entity.md`. T6 is present in the backend `master` history, but no completed T6 task file was found.
- **Constraints / assumptions:** Implement both documented status values, `PENDING` and `ACTIVE`, as clarified by the user. Preserve the entity specification's nullability and lack of a uniqueness constraint on `(user_id, organization_id)`. Backend `master` is clean and synchronized with `origin/master` at `05026c4`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the Membership persistence scaffold from `../../docs/entities/Membership.md` across the backend layers. Add a JPA entity mapped to `public.memberships`, with identity-generated `membership_id`, required `user` and `organization` many-to-one relationships, and nullable string-enum columns for `role` (`MEMBER`, `MANAGER`) and `status` (`PENDING`, `ACTIVE`). Add the repository, minimal business service abstraction/implementation, and an empty controller scaffold following the existing User and Organization structure. Keep business transitions, authorization, approval behavior, and API operations out of this entity task. Do not add a uniqueness constraint for `(user_id, organization_id)` because the supplied schema does not define one.

The enum values are represented with `EnumType.STRING` so the persisted values match the entity specification and remain readable. A numeric enum mapping was considered but rejected because it would make the schema dependent on enum declaration order.

### Steps

1. **Model:** add the Membership entity in `Model`, plus `MembershipRole` and `MembershipStatus` enums if needed by the model convention. Map the generated `membership_id` and the `user`/`organization` associations to non-null `user_id` and `organization_id` join columns. Map nullable `role` and `status` with `@Enumerated(EnumType.STRING)` and the documented column names. Do not add a composite unique constraint or membership lifecycle rules.
2. **Persistence:** add a `MembershipRepository` extending `JpaRepository<Membership, Long>` and annotated according to the existing repository convention. Keep it limited to database access; no approval, authorization, or duplicate-membership rules are added.
3. **Business/service:** add the Membership service interface and Spring service implementation required by the entity scaffold, keeping them minimal and free of business behavior. Use constructor injection if the implementation has a repository dependency.
4. **API/controller:** add an empty `MembershipController` scaffold under `RestServices`, mapped consistently to `/api/memberships`, with no CRUD endpoints or authorization behavior because no endpoint contract is specified for T7.
5. **Verification:** write model mapping and enum-value tests, repository integration tests for generated IDs, required foreign-key associations, nullable role/status, and persistence of both statuses, plus service/controller structure tests where applicable. Run the focused tests and then the wider backend suite after implementation; record red and green results in the task file.

### Files to create / modify

- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Membership.java` (create).
- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/MembershipRole.java` (create, if enums are kept as separate model types).
- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/MembershipStatus.java` (create, if enums are kept as separate model types).
- `../../repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` (create).
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MembershipService.java` and `implementation/MembershipServiceImpl.java` (create).
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` (create).
- Corresponding `src/test/java` files in Model, Persistence, Business, and RestServices for the scaffold and its persistence mappings.

### Tests to write

- Acceptance criterion 1: a model test verifies the JPA table/schema, identity-generated ID, non-null `user` and `organization` join columns, nullable role/status columns, `EnumType.STRING` mapping, and the complete role/status value sets; a repository test saves and reloads a Membership linked to persisted User and Organization records, including `PENDING` and `ACTIVE` statuses, and verifies generated IDs and nullable fields.
- Acceptance criterion 2: repository tests verify null `user` and null `organization` are rejected by persistence; model/service/controller structure tests verify the repository and scaffold wiring. No authorization, approval transition, or endpoint error tests apply because those behaviors are outside T7 and no endpoints are specified.
- Edge cases: each nullable `role`/`status` combination can be persisted, both role values persist as their names, both status values persist as their names, and multiple rows for the same user/organization pair remain allowed because no uniqueness constraint is specified.

### Risks / open questions

- The entity specification gives relationship cardinality and column constraints but no API contract. The plan therefore uses an empty `/api/memberships` controller, consistent with the existing entity scaffolds; later workflow tasks must define and secure membership operations.
- The exact database schema used by repository tests must contain the referenced `users`, `organizations`, and `memberships` tables. Tests should use the existing test profile/schema setup rather than introduce a task-local schema or migration.
- The service has no business rules to implement yet; approval transitions and organization-role authorization remain deferred to workflow tasks.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** The four focused Membership test classes listed in Metadata were written before the implementation, with compile stubs added for the new types.
- **Result:** The focused Gradle invocation could not reach test execution in this environment. Using the direct JDK path and a workspace-local `GRADLE_USER_HOME`, `.\gradlew.bat --no-daemon :Model:test --tests com.ubb.events.domain.MembershipTest --rerun-tasks --console=plain` failed while resolving the wrapper distribution with `java.net.UnknownHostException: services.gradle.org`.
- **Failure reason:** No valid red test result was obtainable because the Gradle 9.8 distribution is not cached and the sandbox has no network resolution. The earlier default Java shim also stalled; the direct JDK invocation exposed the network blocker.

### Green (after implementation)

- **Result:** The user reports that the tests are passing. I could not independently verify that result: with Gradle 9.8 downloaded, the focused command still failed because Gradle could not connect to its localhost daemon. The exact command and output from the user's test run were not provided.
- **Full suite of the affected repo:** The user reports the tests pass; the agent could not run the full suite because of the same Gradle daemon connection error. `git diff --check` passes.

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

- **Decisions:** Membership uses a generated identity `membership_id`, required many-to-one associations to User and Organization, and nullable string enums for role and status. Both documented status values (`PENDING`, `ACTIVE`) are represented. The controller and service remain empty scaffolds because T7 specifies no operations or lifecycle rules.
- **Alternatives considered:** Numeric enum persistence was rejected because it depends on enum declaration order. A composite uniqueness constraint was rejected because the supplied entity specification does not define one.
- **Limitations:** Gradle tests could not be executed in this environment because the configured Gradle 9.8 distribution was not cached and `services.gradle.org` could not be resolved. The focused and full-suite results must be rerun in an environment with the wrapper distribution available.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green; user-reported)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``
