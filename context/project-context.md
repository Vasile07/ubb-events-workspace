# Project Context

Small, stable context shared by all agents. Keep it short. Full documentation lives in Google Drive (written manually by the team). Update this file only when a decision actually changes, and tell the user when you think it is outdated.

Values marked `TODO` must be filled in by the team. Agents must not guess them: if a needed value is `TODO`, ask the user.

## 1. Project

Enterprise event management system for UBB (Babeș-Bolyai University) student organizations. It covers the event lifecycle: creation and planning, publication, participant registration, execution (attendance) and post-event activities. Main areas: event management, organizations and users, authentication/authorization, registration with business rules (capacity, registration status), search, persistence, APIs, UI, testing and deployment.

User roles (confirm with the team): `TODO` (e.g. participant/student, organizer, organization admin, platform admin).

## 2. Stack

| Layer | Technology |
|-------|------------|
| Backend language / framework | Java 21 Spring boot |
| Frontend framework | React |
| Database | PostgreSQL |
| ORM / data access | JPA |
| Authentication / authorization | TODO |
| API style | REST + OpenAPI |
| Backend test framework | JUnit 5 (Jupiter) |
| Frontend test framework | TODO |
| Build / package manager (backend) | Gradle, always through the wrapper (`gradlew`) |
| Deployment | TODO (e.g. Docker Compose, cloud provider) |

## 3. Architecture and layering rules

Multi-layer architecture. Concerns must stay separated:

- **Presentation (frontend):** UI only. No business rules. Talks to the backend through the API.
- **API / controllers:** request validation, mapping, status codes. No business logic and no direct database access.
- **Business / application logic (services):** all business rules (capacity, registration status, permissions, lifecycle transitions).
- **Persistence (repositories / data access):** database access only.
- **External interfaces:** isolated behind their own module (e.g. email, external APIs).

Layering rules specific to the chosen stack: `TODO`

## 4. Conventions

- **Naming (code):** TODO (classes, methods, files, DB tables/columns)
- **API conventions:** TODO (URL style, error response format, pagination, versioning)
- **Error handling:** TODO (exception types, error format returned to the client)
- **Code style / formatter / linter:** TODO
- **Tests:** unit tests for business logic, TDD order from `AGENTS.md`. Backend tests are JUnit 5, in `src/test/java`, mirroring the package of the class under test, and named `<ClassName>Test`.
- **Configuration and secrets:** never commit secrets or `.env`. Use environment variables. Keep an example file (e.g. `.env.example`) without real values.

## 5. Git and branching

- Branch name: `<task-id>-<short-description>` (e.g. `T12-event-registration`).
- Default branch: `master` (both repos).
- One PR per repo per task. PR title starts with the task id.
- Commits happen inside the specific repo folder, not at the workspace root.
- Commit message format: TODO (e.g. `<task-id>: short imperative summary`)

## 6. Repositories

Repository URLs and clone instructions live in `repos/README.md` (single source of truth). Local paths:

- Backend: `repos/ubb-events-backend/`
- Frontend: `repos/ubb-events-frontend/`

## 7. How to run and test

Backend (Java 21, Gradle, JUnit 5). Always use the Gradle wrapper from the backend repo root (`repos/ubb-events-backend/`), never a globally installed `gradle`, so everyone builds with the same version.

```
# Windows (cmd / PowerShell)
.\gradlew.bat build             # compile everything and run all tests
.\gradlew.bat run               # run the application (main class configured in build.gradle)
.\gradlew.bat test              # run all tests
.\gradlew.bat test --tests "com.example.ClassTest"           # run one test class
.\gradlew.bat test --tests "com.example.ClassTest.methodName" # run one test method
.\gradlew.bat clean test        # clean first, forces a full re-run of the tests

# Linux / macOS
./gradlew build
./gradlew run
./gradlew test
./gradlew test --tests "com.example.ClassTest"
./gradlew clean test
```

Notes for the TDD steps:

- Gradle skips tests that are up to date. To see a red or green result again without changing code, use `clean test` or add `--rerun-tasks`.
- Test reports are in `build/reports/tests/test/index.html` (open in a browser). Raw results are in `build/test-results/test/`.
- A compilation error is **not** a valid red result. The tests must compile and then fail on behavior.
- Requires JDK 21 (`java -version`).

Frontend: run `TODO`, test `TODO`

Local environment (database, services): `TODO`

## 8. Where things are documented

- Requirements, use cases, architecture, ERD, API specification, ADRs: [Google Drive doc](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban)
- Task evidence (plans, test log, corrections): `changes/` in this workspace.