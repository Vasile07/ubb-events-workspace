# T3 - API conventions and error handling

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T3
- **Type:** architecture
- **Owner:** Irimies Vasile
- **Requirement(s):** R8, R9
- **Use case(s):** N/A — no use cases apply to this architecture task.
- **Workflow:** architecture
- **Drive doc / ADR:** N/A — no external Drive document or ADR applies; the required conventions update will be recorded in the workspace project context.
- **Repo(s):** backend
- **Branch:** `T3-api-conventions-and-error-handling`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/4
- **Tests:** `RestServices/src/test/java/com/ubb/events/errors/GlobalErrorHandlingApiTest.java`; `RestServices/src/test/java/com/ubb/events/BackendApiDocumentationTest.java`; `RestServices/src/test/java/com/ubb/events/errors/MockErrorController.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “API conventions and error handling” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** architecture. **Requirements:** R8, R9. **Depends on:** Backend skeleton.
- Generic exception types (business, not found, conflict, validation, forbidden).
- A global exception handler (`@RestControllerAdvice`) that maps them to HTTP statuses.
- One shared error response format (status, message, list of errors), also used for validation errors and unexpected errors (no stack traces).
- Bean Validation on request DTOs.
- URL, status code and DTO conventions, recorded in `context/project-context.md`.
- The error format appears in the generated OpenAPI document. No pagination.

Dependencies from the backlog: 2.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/UserController.java`, `OrganizationController.java`; `RestServices/src/test/java/com/ubb/events/BackendApiDocumentationTest.java`; `RestServices/build.gradle`; `docs/coding-conventions.md`.
- **Related completed tasks:** None found for this API/error-handling area.
- **Constraints / assumptions:** Backend uses Spring Boot 3.5.6, Java 21, Spring MVC, Bean Validation, springdoc-openapi 2.8.13, and a multi-module Gradle build. Controllers currently establish `/api/users` and `/api/organizations` base paths; the OpenAPI test checks `/api/health`. Existing project context specifies `/api/organizations/{orgCode}/...` for organization-scoped routes, non-paginated lists, shared error fields (`status`, `message`, `errors`), and no stack traces. T3's backlog requires these conventions to be recorded in `context/project-context.md`, which already contains a conventions section; planner should identify the specific additions or confirmation needed.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Define one API error contract and make it the boundary between controllers and clients. Add the five generic exception categories in the dedicated `Common` package `com.ubb.events.common.errors`, a `@RestControllerAdvice` that converts them and validation/unexpected failures into the shared `{status, message, errors}` response, and request DTO validation in the API layer. Record the endpoint, status-code, DTO, validation, and non-pagination conventions in `context/project-context.md`, and expose the error schema/statuses in springdoc OpenAPI. Use a test-only mock controller to exercise request DTO validation and the advice without adding or changing a product controller. For non-validation failures, `errors` will default to an empty list; structured details may be added in cases where useful. The proposed status mapping below is for user review.

| Exception / failure | Proposed HTTP status | Rationale |
|---|---:|---|
| `BusinessRuleException` | 400 Bad Request | Request is syntactically valid but violates an application rule. |
| Not found | 404 Not Found | Requested resource does not exist. |
| Conflict | 409 Conflict | Request conflicts with current resource state. |
| Validation | 400 Bad Request | Request DTO fails Bean Validation. |
| Forbidden | 403 Forbidden | Caller is not allowed to perform the action. |
| Any other unhandled exception | 500 Internal Server Error | Catch-all handling without a dedicated exception class; return a generic client-safe message. |

### Steps

1. Confirm the error contract and convention wording against Requirements R8/R9: field types for `status`, `message`, and `errors`; safe structured error details; URL and DTO naming rules. The status mapping table below is approved.
2. Add the shared error response DTO/schema under `Model/src/main/java/com/ubb/events/dto`, including OpenAPI-visible descriptions and no stack-trace fields.
3. Add generic `BusinessRuleException`, not-found, conflict, validation, and forbidden exception types in `Common/src/main/java/com/ubb/events/common/errors`, with constructors that preserve a client-safe message and optional structured validation errors.
4. Add a global `@RestControllerAdvice` in `RestServices` mapping those exceptions, Bean Validation failures, malformed requests where applicable, and any other unhandled exception to the agreed statuses and the same error shape. The 500 response is a catch-all with a generic client-safe message; do not create a dedicated exception class for it. Include details in `errors` only when useful to frontend/debugging and safe to disclose, never implementation or sensitive business details. Ensure logs may retain diagnostics while responses never expose stack traces or internal details.
5. Apply Bean Validation to a request DTO exercised through a test-only mock controller, so validation and exception advice can be tested without adding or changing a product controller. Do not move business validation into controllers.
6. Document the shared error schema and representative error status codes in the generated OpenAPI document; preserve the existing health/API-doc behavior.
7. Update `context/project-context.md` with the confirmed URL, status-code, DTO, validation, error-shape, OpenAPI, and no-pagination conventions.
8. Add the backend-specific error contract to `repos/ubb-events-backend/docs/api-error-handling.md` so backend contributors can find the package layout, status mapping, response shape, and disclosure rules in the backend repo.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/...`: shared error response DTO/schema.
- `repos/ubb-events-backend/Common/src/main/java/com/ubb/events/common/errors/...`: generic exception types.
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/...`: global `@RestControllerAdvice`, exception-to-status mapping, validation/unexpected error conversion, and OpenAPI annotations.
- `repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/...`: test-only mock controller and API tests for the advice, validation, and generated OpenAPI error schema; no product controller changes or additions.
- `repos/ubb-events-backend/docs/api-error-handling.md`: backend-specific error response and exception conventions.
- `context/project-context.md`: confirmed API, DTO, status-code, validation, error-response, and documentation conventions.
- `changes/active/T3-api-conventions-and-error-handling.md`: test evidence, design decisions, and final metadata (implementation phase).

### Tests to write

- An API test using a test-only mock controller for each exception category, asserting its agreed HTTP status and shared `status`, `message`, and `errors` JSON fields.
- A validation test sending an invalid request DTO through the test-only mock controller and asserting the validation status, field-level errors, and absence of stack traces.
- An unexpected-exception test asserting the generic status/message shape and that exception details/stack traces are not returned.
- A test for a valid request proving Bean Validation permits the request to reach the test-only mock controller.
- An OpenAPI test asserting the generated document contains the shared error schema and documented error responses, while retaining the existing health endpoint assertion.
- Tests must be written before implementation and mapped to both acceptance criteria; the implementer will record the compiling red result and the green result plus the wider backend suite in `Test log`.

### Risks / open questions

- The proposed test matrix covers each mapped exception category, validation, the catch-all 500 behavior, a valid request, and OpenAPI output; it is considered sufficient for the current acceptance criteria.
- The HTTP status mapping is approved by the user.
- The generic business exception is named `BusinessRuleException`, as confirmed by the user.
- Exception ownership is `Common/src/main/java/com/ubb/events/common/errors`; the shared error DTO belongs in `Model/src/main/java/com/ubb/events/dto`, per user direction.
- For non-validation responses, `errors` defaults to an empty list. Include details only when relevant to frontend/debugging and safe to disclose; omit implementation details and sensitive business logic. The catch-all 500 response uses a generic message.
- The current controllers contain no request DTO endpoints. Validation coverage will use a test-only mock controller, with no product controller added or changed.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `GlobalErrorHandlingApiTest` (exception mappings, validation, safe 500 response, valid request) and the shared error-schema assertion in `BackendApiDocumentationTest`.
- **Result:** The tests were written before the implementation. The initial red run could not start because Gradle was denied access to its user-level lock file, and the local distribution fallback failed to initialize its native-platform DLL. On 2026-10-19, a delayed behavioral red check ran with `GlobalExceptionHandler` temporarily removed from source and restored immediately afterward. The API test class completed 8 tests; 7 failed as expected because exception mapping and validation advice were absent, while the valid-request test passed.
- **Failure reason:** Without the global handler, domain exceptions escaped as servlet errors and Bean Validation did not return the shared error response. Because of the initial Gradle startup failure, this behavioral red check was delayed until after implementation; it confirms the tests detect the missing behavior but does not reproduce the original pre-implementation red timing.

### Green (after implementation)

- **Result:** After restoring the handler, `GlobalErrorHandlingApiTest` and `BackendApiDocumentationTest` passed with `.\gradlew.bat :RestServices:test --tests com.ubb.events.errors.GlobalErrorHandlingApiTest --tests com.ubb.events.BackendApiDocumentationTest --rerun-tasks` (13 actionable tasks executed).
- **Full suite of the affected repo:** `.\gradlew.bat test` passed after restoration (20 actionable tasks; 1 executed, 19 up-to-date).

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | Exception ownership was not specified; testing validation could have implied a new product controller; the status mapping needed review. | Set exception ownership to the dedicated `Common` package `errors`, use a test-only mock controller without changing or adding a product controller, and added a proposed mapping table for review. | The user specified these choices and requested a reviewable mapping table. |
| 2 | plan | A separate unexpected-error exception class was not needed, and the generic business exception could have a clearer name. | Removed the dedicated unexpected-error class from the plan; any other unhandled exception maps through the catch-all handler to 500. Proposed `BusinessRuleException` as the business exception name. | The user requested catch-all handling for 500 errors and a better business exception name. |
| 3 | plan | The status mappings, error DTO location, safe optional details, and 500 message needed confirmation. | Recorded the approved status mapping; placed the shared DTO in `Model`'s dedicated `dto` package; limited optional `errors` details to relevant, safe-to-disclose information; set a generic client-safe 500 message. | The user confirmed these contract decisions. |
| 4 | plan | Whether T3 should update `context/project-context.md` was unclear after marking Drive doc / ADR as N/A. | Kept the project-context update in scope and clarified that Drive doc / ADR is N/A because no external Drive document or ADR applies. | The user confirmed the project context must be updated so future sessions use the agreed error conventions. |
| 5 | implementation | The error information targets the backend only. | Added the backend-specific error contract to `repos/ubb-events-backend/docs/api-error-handling.md` and linked to it from the workspace project context. | The user requested that backend-targeted information also live in the backend repo's `docs` folder. |
| 6 | implementation | `ErrorDetail` should be a class, not a record; avoid records and use interfaces and classes. | Converted the error DTOs and test-only validation DTO to classes, and added the no-records rule to backend coding conventions. | The user requested a backend-wide convention against records. |
| 7 | implementation | ErrorDetail and ErrorResponse should use Lombok for constructors and getters. | Replaced their explicit constructors and getters with Lombok `@AllArgsConstructor` and `@Getter`. | The user requested that the DTOs follow the Lombok coding convention. |
|   |                               |                |                              |     |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Keep exception classes in `Common` under `com.ubb.events.common.errors`; keep the shared `ErrorResponse` and `ErrorDetail` classes in `Model` under `com.ubb.events.dto` with Lombok-generated constructors/getters; centralize mappings in a `@RestControllerAdvice`; use a test-only controller for validation coverage; return a generic 500 message and empty `errors` list for unhandled failures; document backend API error conventions in `docs/api-error-handling.md` and prohibit Java records in favor of classes and interfaces.
- **Alternatives considered:** A dedicated unexpected-error exception was rejected in favor of catch-all advice handling. Product-controller changes were avoided by importing a test-only controller. Structured error details remain opt-in and safe-to-disclose.
- **Limitations:** The behavioral red check was delayed until after implementation because Gradle could not start during the initial red attempt. The later controlled red run verified test sensitivity; all targeted and full-suite green tests passed after restoring the handler.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red; delayed check, see Test log)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
