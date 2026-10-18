# T6 - Init Organization entity

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T6
- **Type:** entity
- **Owner:** Irimies Vasile
- **Requirement(s):** R7
- **Use case(s):** N/A
- **Workflow:** entity
- **Drive doc / ADR:** N/A
- **Repo(s):** backend
- **Branch:** `T6-init-organization-entity`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/3
- **Tests:** `OrganizationTest`, `OrganizationRepositoryTest`, `OrganizationServiceStructureTest`, `OrganizationControllerStructureTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Init Organization entity” from the backlog. This task addresses the backend scope and user need described below.

Read `/docs/entities/Organization.md` for this entity's fields, types, constraints, and relationships.

Backlog specification:

- **Type:** entity. **Requirements:** R7. **Depends on:** Database creation, Backend skeleton, API conventions and error handling.
- Entity scaffold for Organization, using `organization_id` as its identifier. There is no `orgCode`. Organizations are read-only through the API (they are created in SQL).

Dependencies from the backlog: 1, 2, 3.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/User.java` and `Model/src/test/java/com/ubb/events/domain/UserTest.java` show the existing JPA entity mapping and unit-test pattern; `Persistence/src/main/java/com/ubb/events/persistence/UserRepository.java` shows the repository pattern; `RestServices/src/main/java/com/ubb/events/controllers/UserController.java` is the existing empty controller scaffold.
- **Related completed tasks:** `T4-init-user-entity.md` records the preceding entity scaffold workflow and implementation pattern; `docs/entities/Organization.md` is the Organization field and relationship specification.
- **Constraints / assumptions:** Backend is Java 21/Spring Boot with JPA, split into Model, Persistence, Business, and RestServices modules; tests use JUnit 5 and the Gradle wrapper. Organization fields in the entity specification are `organization_id`, `name`, nullable `description`, and nullable `logo_url`. As clarified by the user, `organization_id` is the identifier and there is no `orgCode`. Organizations are created through SQL and are read-only through the API; the exact read endpoint contract is not present here.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the Organization persistence scaffold in the backend following the existing User entity structure and the fields in `docs/entities/Organization.md`. Add a JPA entity mapped to `public.organizations`, a repository, empty service interface and implementation class definitions, and a controller scaffold at `/api/organizations` that injects the service. `organization_id` is the generated identifier; no `orgCode` field or mapping is introduced. Relationships to memberships and events, business rules, and API operations remain outside this entity task.

The service interface and implementation intentionally contain no operations yet; they provide only the class definitions and annotations needed for later work. The controller also contains no handler methods or endpoints because organizations are seeded through SQL and are read-only through the API.

### Steps

1. **Model:** add the `Organization` JPA entity in the domain package, mapped to `public.organizations`, with an identity-generated `Long organization_id` primary key, required `name`, and nullable `description` and `logo_url`. Do not add `orgCode` or relationship mappings whose entity contracts are not part of this task.
2. **Persistence:** add the `OrganizationRepository` in the persistence package using the established Spring Data repository convention and `Organization`/`Long` types.
3. **Business/service:** add an empty `OrganizationService` interface and an annotated `OrganizationServiceImpl` class implementing it. Do not add service methods, repository delegation, create/update/delete operations, validation, authorization, or other business rules.
4. **API:** add an `OrganizationController` annotated as a REST controller, mapped to `/api/organizations`, with a constructor-injected `OrganizationService`. Define no request-mapping methods, endpoints, DTOs, or response handling.
5. **Verification:** write the focused tests first, run them to capture the required red result, implement the scaffold, then run the focused and affected-repository suites and record the red/green evidence in the task file.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Organization.java` (create).
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/OrganizationRepository.java` (create).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/OrganizationService.java` (create).
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/OrganizationServiceImpl.java` (create).
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/OrganizationController.java` (create).
- Corresponding tests under `Model`, `Persistence`, `Business`, and `RestServices` `src/test/java` packages; no frontend files.

### Tests to write

- **Acceptance criterion 1 (backlog implementation):** `OrganizationTest` verifies the entity annotations/table, generated `organization_id`, required `name`, and nullable `description`/`logo_url`; `OrganizationRepositoryTest` verifies persistence and retrieval using the configured test database; a service structure test verifies the empty `OrganizationService` interface and annotated implementation; a controller structure test verifies the `/api/organizations` root mapping and service constructor while confirming no handler methods are declared.
- **Acceptance criterion 2 (relevant behavior and error cases):** persistence tests cover a missing required `name` and normal null optional fields. Service and controller have no operations to exercise, so no service delegation, authorization, or endpoint error tests apply; the tests verify that no methods/endpoints were introduced.
- Tests must not introduce `orgCode`, write operations, relationship behavior, or endpoint contracts that are outside this task.

### Risks / open questions

- The exact read API contract is unspecified; the plan therefore provides only the controller class, root path, and service dependency, with no endpoints.
- The database schema is supplied by the Database creation task; repository integration tests depend on that schema and its PostgreSQL test configuration.
- Membership and event relationships are deferred until their entity and workflow tasks define the owning side and behavior.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** Added `OrganizationTest`, `OrganizationRepositoryTest`, `OrganizationServiceStructureTest`, and `OrganizationControllerStructureTest` before the final annotations and mappings were implemented.
- **Result:** Focused Gradle run reached `:Model:test` and reported `OrganizationTest > organizationHasExpectedJpaMappingAndScalarFields() FAILED` (1 of 2 tests failed).
- **Failure reason:** The initial compile stubs intentionally lacked the required JPA and Spring annotations/mappings, so the structural assertions failed. The first wrapper attempt was blocked by access to the Gradle cache lock; the rerun with elevated access executed the tests successfully.

### Green (after implementation)

- **Result:** Focused tests passed for all four modules with `BUILD SUCCESSFUL` (20 actionable tasks; Model, Persistence, Business, and RestServices Organization tests all green).
- **Full suite of the affected repo:** `.\gradlew.bat test` passed with `BUILD SUCCESSFUL` (20 actionable tasks; 4 executed, 16 up-to-date).

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The task said the Organization entity has a unique `orgCode`. | Replace `orgCode` with `organization_id`; there will be no `orgCode`. | As clarified by the user. |
| 2 | plan | The controller scope was unspecified. | Define only the controller class, root path, and service dependency; include no functions or endpoints. | As clarified by the user. |
| 3 | plan | The service interface and implementation were planned with read methods and repository delegation. | Keep the service interface and implementation as empty class definitions with annotations only; remove service operations and delegation. | As clarified by the user. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Map `Organization` to `public.organizations` with identity-generated `organization_id`, required `name`, and nullable `description`/`logo_url`. Keep the service interface and implementation empty except for their type/`@Service` scaffolding. Keep the controller at `/api/organizations` with constructor-injected `OrganizationService` and no handlers.
- **Alternatives considered:** Adding service operations, repository delegation, relationships, or read endpoints was considered out of scope because the approved plan explicitly defers those contracts and organizations are seeded through SQL.
- **Limitations:** No API endpoint is available yet; the controller is intentionally a structural scaffold. Relationships to memberships/events and business rules remain for later tasks. Repository tests use the configured H2 test profile rather than a production PostgreSQL instance.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — `fa89250` (`T6: initialize organization entity scaffold`) on `T6-init-organization-entity`.
- [x] PR created — https://github.com/Vasile07/ubb-events-backend/pull/3
- [x] PR reviewed — approved by user
- [x] **Gate 3:** merged by user — PR #3 merged into `master`.
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
