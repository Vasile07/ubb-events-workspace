# T2 - Backend skeleton

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T2
- **Type:** architecture
- **Owner:** Vasile Irimies
- **Requirement(s):** R8
- **Use case(s):** 
- **Workflow:** architecture
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T2-backend-skeleton`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/1
- **Tests:** `RestServices/src/test/java/com/ubb/events/controllers/HealthControllerTest.java`; `RestServices/src/test/java/com/ubb/events/BackendApplicationTest.java`; `RestServices/src/test/java/com/ubb/events/BackendApiDocumentationTest.java`; `RestServices/src/test/java/com/ubb/events/ProjectBuildConfigurationTest.java`; `RestServices/src/test/java/com/ubb/events/EnvConfigurationTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Backend skeleton” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** architecture. **Requirements:** R8. **Depends on:** Database creation.
- Gradle project with the layered packages (API, service, persistence).
- Lombok and JPA setup, datasource configuration, profiles.
- springdoc-openapi setup, so the API specification is generated from the code.
- Test setup uses an H2 in-memory database, working from the first task. The test profile must not require a separately installed or running database.

Dependencies from the backlog: 1.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.
- [ ] The test profile uses an H2 in-memory database and can run without an external database service; Supabase PostgreSQL configuration remains separate for application use.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/` exists but is empty (no commits or source files); it is on the user-authorized local `master` branch. The remote has no branch/commit to fetch. `context/project-context.md` defines the stack and layering conventions.
- **Related completed tasks:** None available; `changes/completed/` has no completed task files.
- **Constraints / assumptions:** `context/project-context.md` specifies Java 21, Spring Boot, Gradle wrapper, PostgreSQL/JPA, REST/OpenAPI, and JUnit 5. User-added requirements: use `rest-services`, `business`, `persistance` (as named by the user), `domain`, and `common/{configs,security,utils}`; add a `HealthController`; configure Swagger and Supabase PostgreSQL connectivity. Supabase connection string, username, and password will be supplied later and must be provided as environment values rather than committed secrets. T2 is architecture type.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Create and configure a Gradle multi-project build containing exactly five Java subprojects/modules named and located `Business`, `RestServices`, `Persistence`, `Model`, and `Common`. Each subproject owns its own `build.gradle`, Java 21/toolchain, repositories, dependency management, dependencies, and JUnit configuration; the root build only coordinates shared plugin versions. Map the business service layer to `Business`, controllers and `HealthController` to `RestServices`, repositories/database access to `Persistence`, models and request/response DTOs to `Model`, and configs/security/utils to `Common`. Use the confirmed base package `com.ubb.events`, the confirmed health contract `GET /api/health` returning `{ "status": "UP" }`, and the confirmed Supabase environment variables. Keep T2 limited to project initialization/configuration and the health endpoint; do not add event features, domain rules, or JWT behavior. Database credentials will be read from environment variables and an example configuration will contain placeholders only.

Use a conventional Spring Boot source layout and Gradle wrapper. Configure the confirmed module dependency graph and Swagger endpoints. Use H2 as the test-profile database: it is a real in-memory database, fast, and requires no installation. Record the H2 test choice and environment variable contract as architecture decisions, including the known limitation that H2 behavior can differ from the production PostgreSQL database.

### Steps

1. **Repository/build foundation:** create the Gradle wrapper and root settings for five Java subprojects named exactly `Business`, `RestServices`, `Persistence`, `Model`, and `Common`. Keep the root build limited to shared plugin/version coordination and put each project's Java 21/toolchain, repositories, dependency management, dependencies, and JUnit 5 setup in its own `build.gradle`. Configure Lombok, Spring Web, validation, JPA, PostgreSQL, springdoc-openapi, and JUnit 5 dependencies. Add the runnable Spring Boot entry point in the approved module and a minimal test profile.
2. **Module structure and dependencies:** create the five Java Gradle project source trees and map responsibilities strictly. Apply the confirmed project dependencies: `Model` has none; `Common` depends on `Model`; `Persistence` depends on `Model` and `Common`; `Business` depends on `Model`, `Persistence`, and `Common`; `RestServices` depends on `Business`, `Model`, and `Common`. Use `com.ubb.events` for Business interfaces, `com.ubb.events.implementation` for Business implementations, `com.ubb.events.domain` for Model entities, `com.ubb.events.dto` for Model DTOs, and `com.ubb.events.controllers` for RestServices controllers.
3. **API layer:** implement `HealthController` in `RestServices` as a documented `GET /api/health` endpoint returning `{ "status": "UP" }` with appropriate HTTP response documentation. It must contain no business logic or direct persistence access.
4. **Configuration:** configure application profiles in `RestServices/src/main/resources/application.properties` and, if needed, `application-test.properties`. Configure PostgreSQL/JPA datasource properties using `SUPABASE_JDBC_URL`, `SUPABASE_DB_USERNAME`, and `SUPABASE_DB_PASSWORD`. The user will supply the values later through JDBC configuration. Add a placeholder-only `.env.example` or equivalent example configuration, and ensure no real credentials or `.env` files are committed. Configure springdoc-openapi/Swagger UI at `/v3/api-docs` and `/swagger-ui.html`.
   The runtime properties optionally import `.env[.properties]` from the backend root, and `RestServices` configures `bootRun` to use that root as its working directory.
5. **Cross-cutting boundaries:** add the initial configs, security, and utils packages in `Common`, limited to project initialization/configuration. Do not implement JWT authentication, event features, or business rules in T2.
6. **Database test infrastructure:** configure H2 as the test-profile database with schema generation and test connection properties isolated from development/Supabase credentials. Record that the backlog requirement was changed to H2-only testing and that PostgreSQL compatibility remains a known limitation.
7. **Verification and traceability:** run the required red-green TDD sequence, then the complete backend test suite and build through `gradlew`. Record exact commands/results in the Test log and fill the task's test metadata after implementation.

### Files to create / modify

- `repos/ubb-events-backend/settings.gradle` and root `build.gradle` (or the chosen Gradle Kotlin/Groovy equivalents), Gradle wrapper files, and project-level build metadata.
- `repos/ubb-events-backend/Business/`, `RestServices/`, `Persistence/`, `Model/`, and `Common/` Gradle project directories, each with its own `build.gradle`, Java source/test layout, toolchain, dependency management, dependencies, and matching build identifier.
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/HealthController.java` and the approved runnable Spring Boot entry point.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/...` for service interfaces and `.../implementation/...` for implementations; `Persistence/src/main/java/<base-package>/...` for repositories; `Model/src/main/java/com/ubb/events/domain/...` for entities and `.../dto/...` for DTOs; and `Common/src/main/java/<base-package>/...` for configs/security/utils.
- `repos/ubb-events-backend/RestServices/src/main/resources/application.properties` and, if needed, `application-test.properties` for default/test/profile configuration, with environment-variable placeholders only.
- `repos/ubb-events-backend/.env.example` if needed for the documented Supabase variable contract; never create or commit `.env` or real credentials.
- Optional local `repos/ubb-events-backend/.env` beside `settings.gradle` / `.env.example`; it is ignored and must contain only user-supplied local values.
- `repos/ubb-events-backend/<module>/src/test/java/<base-package>/...` for module-specific tests and H2 test-profile configuration.
- `repos/ubb-events-backend/README.md` only if needed to document local startup, Swagger URLs, required environment variables, and test prerequisites; no workspace-level documentation changes.
- `repos/ubb-events-backend/AGENTS.md` for backend-specific guidance subordinate to workspace instructions, and `repos/ubb-events-backend/docs/coding-conventions.md` for JPA, Lombok, SOLID, OOP, dependency injection, and Java naming conventions.

### Tests to write

Write tests before the implementation and map them to the acceptance criteria as follows:

- **Backlog skeleton/build criterion:** a Spring application context smoke test and a Gradle compilation/test execution check prove the application starts with the test profile and required dependencies resolve.
- **Layer/module criterion:** Gradle project evaluation and compile-time/module-location verification prove that `Business`, `RestServices`, `Persistence`, `Model`, and `Common` exist with the requested responsibilities; no controller test may access a repository directly.
- **HealthController criterion (user addition):** `HealthControllerTest`/MockMvc verifies `GET /api/health` returns `{ "status": "UP" }`, the expected status/content type, and documented response shape.
- **Swagger criterion (user addition):** an API-level test requests `/v3/api-docs` and verifies `HealthController` is described; a focused configuration check verifies Swagger UI is enabled at `/swagger-ui.html`.
- **Datasource/profile criterion:** an H2 integration test starts the application with the test profile and verifies a JPA datasource can connect. The test documentation records that H2 is real in-memory testing and may differ from production PostgreSQL. A configuration test verifies Supabase values are sourced from environment properties and no credentials are committed.
- **Module dependency criterion:** Gradle project evaluation verifies the exact five projects and the confirmed dependency graph, with no reverse or undeclared project dependencies.
- **Project build ownership criterion:** `ProjectBuildConfigurationTest` verifies that every Gradle subproject has its own `build.gradle` and declares the Java library plugin.
- **Environment configuration criterion:** `EnvConfigurationTest` verifies optional `.env` import and that `bootRun` uses the backend root working directory.
- Run the focused tests after writing stubs and confirm a behavioral red result, then implement and confirm green, followed by `gradlew clean test` and `gradlew build`.

### Risks / open questions

- Confirmed choices: the five Gradle subprojects span the backend; their names and dependency graph are fixed as described above; base package `com.ubb.events`; `GET /api/health` returns `{ "status": "UP" }`; Swagger/OpenAPI uses `/v3/api-docs` and `/swagger-ui.html`; Supabase values use `SUPABASE_JDBC_URL`, `SUPABASE_DB_USERNAME`, and `SUPABASE_DB_PASSWORD`; values will be supplied later through JDBC configuration. No further approval is needed on these points.
- H2-only testing is confirmed. It is a real in-memory database, fast, and requires no installation, but it can differ from production PostgreSQL; this compatibility limitation must remain visible in the task’s design notes and test evidence.
- “Connection via JDBC” does not define the exact Supabase JDBC URL/driver/parameters yet. The user will supply the values later; implementation must keep the confirmed environment-variable contract and avoid inventing URL details.
- The backend remote and local `master` have no commits. The user approved proceeding from this empty `master` state; create `T2-backend-skeleton` from that state and make the first project commit only on the task branch.
- T2 is typed as architecture while the backlog also requires a runnable scaffold and tests. The implementation should remain limited to infrastructure and the health endpoint; event/domain business rules, JWT enforcement, and database entities belong to later tasks.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `HealthControllerTest.healthReturnsUp`.
- **Result:** `gradlew.bat --no-daemon :RestServices:test --tests com.ubb.events.controllers.HealthControllerTest --rerun-tasks` failed with one test failure.
- **Failure reason:** The test expectation remained the approved `{"status":"UP"}` while the controller was temporarily changed to return `{"status":"DOWN"}`. The test compiled and failed on the behavioral assertion (`HealthControllerTest > healthReturnsUp`), confirming the red step; the controller was restored afterward.
- **Tests:** `ProjectBuildConfigurationTest.eachGradleProjectOwnsItsBuildConfiguration`.
- **Result:** `gradlew.bat --no-daemon :RestServices:test --tests com.ubb.events.ProjectBuildConfigurationTest --rerun-tasks` initially failed with one assertion because the five project-owned `build.gradle` files did not yet exist.
- **Failure reason:** The test compiled and failed on the missing per-project build configuration, confirming the red step before the build split.
- **Tests:** `EnvConfigurationTest.optionalEnvFileIsImportedFromBackendRoot`.
- **Result:** `gradlew.bat --no-daemon :RestServices:test --tests com.ubb.events.EnvConfigurationTest --rerun-tasks` initially failed with an assertion because the optional `.env` import and root `bootRun` working directory were not configured.
- **Failure reason:** The test compiled and failed on the missing configuration declarations, confirming the red step before implementation.

### Green (after implementation)

- **Result:** After adding optional `.env` loading and configuring `bootRun` at the backend root, `gradlew.bat --no-daemon clean test` completed successfully. `HealthControllerTest`, `BackendApplicationTest` (Spring context with H2 test profile), `BackendApiDocumentationTest` (OpenAPI includes `/api/health`), `ProjectBuildConfigurationTest`, and `EnvConfigurationTest` passed.
- **Full suite of the affected repo:** A final `gradlew.bat --no-daemon build` completed successfully; all five modules evaluated and `RestServices` produced `ubb-events-backend.jar`.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | the names/project organization in the plan | use their five capitalized CamelCase names, each a Java Gradle project: `Business`, `RestServices`, `Persistence`, `Model`, and `Common` | that is what they want |
| 2 | plan | the plan still asked for decisions that were already clarified and did not reflect the exact dependencies and scope | use base package `com.ubb.events`, the specified health and Swagger endpoints, the specified Supabase environment variables, the exact five-project dependency graph, and limit T2 to initialize/configure the project and add `HealthController` | those are the user's confirmed choices; database values will be supplied later |
| 3 | plan | the test database requirement in the backlog | change the backlog requirement to H2 only and use H2 for the test profile | that is what they want |
| 4 | plan | the resource configuration format | use `application.properties` at resources | that is what they want |
| 5 | implementation | the package structure and repository guidance needed to match the requested conventions | move controllers to `com.ubb.events.controllers`; use `com.ubb.events` for Business interfaces, `com.ubb.events.implementation` for implementations, `com.ubb.events.domain` and `com.ubb.events.dto` in Model; add backend `README.md`, `AGENTS.md`, and `docs/coding-conventions.md` covering the requested Java, JPA, Lombok, SOLID, dependency injection, and OOP conventions | these are the user's requested implementation review changes |
| 6 | implementation | each Gradle project must own its own build.gradle and setup/dependencies; the existing root build centralized subprojects config | move plugins, Java 21/toolchain, repositories, dependency management, dependencies, and test setup into `Business/build.gradle`, `RestServices/build.gradle`, `Persistence/build.gradle`, `Model/build.gradle`, and `Common/build.gradle`; keep root build for shared plugin/version coordination and add build-ownership verification | this is what the user requested |
| 7 | implementation | `.env` should be loaded automatically from the backend root when running the application | add optional `spring.config.import=optional:file:.env[.properties]`, configure `bootRun` to use the backend root, and document copying `.env.example` to `.env` beside `settings.gradle` | this is what the user requested |
| 8 | implementation | the coding conventions should be clearly referenced for agents working in the backend | add an explicit instruction in the backend `AGENTS.md` to read and follow `docs/coding-conventions.md` before changing backend code | make the repository guidance discoverable and applicable to implementers and other agents |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use a five-project Gradle build with the approved names and dependency graph. `RestServices` owns the Spring Boot entry point and controller. Runtime PostgreSQL/Supabase configuration is environment-based through `SUPABASE_JDBC_URL`, `SUPABASE_DB_USERNAME`, and `SUPABASE_DB_PASSWORD`; the test profile uses an H2 in-memory database through `application-test.properties`. OpenAPI is generated by springdoc and exposed at `/v3/api-docs` and `/swagger-ui.html`.
- **Alternatives considered:** A single Gradle project with packages was not selected because the user requested five Gradle subprojects. PostgreSQL/Testcontainers was not selected because the user explicitly changed the backlog to H2-only testing.
- **Limitations:** Supabase values are placeholders until supplied by the user. H2 is intentionally used only for tests and can differ from production PostgreSQL. JWT/security behavior, event features, entities, and business rules are outside T2.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`
