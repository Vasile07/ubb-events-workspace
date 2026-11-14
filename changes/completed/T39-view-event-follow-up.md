# T39 - View event follow-up

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T39
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** LIF-05
- **Use case(s):** none (supports UC-13)
- **Workflow:** workflow
- **Drive doc / ADR:** https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban
- **Repo(s):** backend
- **Branch:** `T39-view-event-follow-up`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/41
- **Tests:** `FollowUpRepositoryTest`, `FollowUpServiceStructureTest`, `FollowUpControllerApiTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View event follow-up” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** LIF-05. **Use case:** none, supports UC-13. **Depends on:** Organization-scoped authorization, Init FollowUp entity, Create event follow-up.
- `GET /api/organizations/{organizationId}/events/{eventId}/follow-up`: the saved follow-up, for members and managers of the organization. Returns 404 when there is none.

Dependencies from the backlog: 9, 37, 38.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/FollowUpController.java` and `Business/src/main/java/com/ubb/events/implementation/FollowUpServiceImpl.java` implement the T38 POST flow; `Model/src/main/java/com/ubb/events/dto/FollowUpResponse.java` is the response shape; `Persistence/src/main/java/com/ubb/events/persistence/FollowUpRepository.java` currently has standard JPA methods; `Persistence/src/main/java/com/ubb/events/persistence/EventRepository.java` provides organization-scoped event lookup; `RestServices/src/main/java/com/ubb/events/security/RequireOrgPermission.java` and `OrganizationAuthorizationInterceptor.java` implement membership role authorization.
- **Related completed tasks:** T37 (Init FollowUp entity) and T38 (Create event follow-up) define and implement the same backend resource.
- **Constraints / assumptions:** Backend only; `FollowUp.event` is a required Event relationship and `FollowUpResponse` contains `id`, `eventId`, and `content`. The backlog explicitly requires member and manager access and 404 when no follow-up exists. The backend repo was clean on `master`; during branch preparation, a fresh fetch showed `origin/master` had advanced to `92e58dd`, and the clean local `master` was fast-forwarded to it. The task branch was created from `92e58dd`.
## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add the read path to the existing FollowUp vertical slice. Keep organization membership authorization at the controller boundary with the existing default `@RequireOrgPermission` policy (members and managers), keep lookup and not-found behavior in the service, and use a repository query that scopes the follow-up to both event and organization. Return the existing `FollowUpResponse` so the GET contract stays consistent with T38. Do not add a second DTO or expose persistence objects.

### Steps

1. **Persistence:** add a repository query for a follow-up belonging to the requested event and organization, using the existing `FollowUp.event.organization` relationship; ensure the absence case can be represented as `Optional`.
2. **Business:** add `getFollowUp(organizationId, eventId)` to `FollowUpService` and implement it in `FollowUpServiceImpl`. Load through the organization-scoped repository query, throw `NotFoundException` with a client-safe message when no follow-up exists, and map the entity to `FollowUpResponse`.
3. **API:** add `GET /api/organizations/{organizationId}/events/{eventId}/follow-up` to `FollowUpController`, annotate it with the existing default `@RequireOrgPermission` (member or manager), document 200/401/403/404 responses, and return HTTP 200 with the mapped response.
4. **Verification:** write focused service and controller/API tests first, obtain a behavioral red result, then run focused module tests and the full backend Gradle suite. Record exact commands and outcomes in the task test log.

### Files to create / modify

- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/FollowUpRepository.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/FollowUpService.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/FollowUpServiceImpl.java`
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/FollowUpController.java`
- Corresponding tests under `Persistence/src/test`, `Business/src/test`, and `RestServices/src/test`.

### Tests to write

- **Backlog implementation:** a service test retrieves an existing follow-up for the requested organization and event and verifies all `FollowUpResponse` fields; a controller/API test verifies the exact GET path and 200 response payload.
- **Behavior, authorization, and errors:** service tests return `NotFoundException` when no follow-up exists and when the event/organization scope does not match. API tests verify unauthenticated requests return 401, a member can retrieve the follow-up, a manager can retrieve it, a non-member receives 403, and a missing follow-up is mapped to the shared 404 error response without invoking persistence outside the service.
- Include a repository integration test for the organization/event lookup query if the existing persistence test setup supports it.

### Risks / open questions

- The exact Spring Data method name depends on the current `FollowUp` and `Event` property names; use the existing `event.organization.id` relationship rather than adding a new database field.
- The task specifies one saved follow-up and does not define duplicate rows. If the repository returns multiple rows, preserve the existing data model assumption and do not invent a duplicate-resolution rule.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `FollowUpRepositoryTest.findsFollowUpOnlyForItsEventAndOrganization`, `FollowUpServiceStructureTest.retrievesFollowUpForOrganizationAndEvent`, `FollowUpServiceStructureTest.missingFollowUpIsNotFoundForRequestedOrganizationAndEvent`, and the four GET API behavior tests.
- **Result:** Before production implementation, the commands `.\gradlew.bat --no-daemon --console=plain :Persistence:test --tests com.ubb.events.persistence.FollowUpRepositoryTest`, `.\gradlew.bat --no-daemon --console=plain :Business:test --tests com.ubb.events.implementation.FollowUpServiceStructureTest`, and `.\gradlew.bat --no-daemon --console=plain :RestServices:test --tests com.ubb.events.controllers.FollowUpControllerApiTest` each ran. Persistence reported 3 tests, 1 failed (`findsFollowUpOnlyForItsEventAndOrganization`); Business reported 10 tests, 2 failed (`retrievesFollowUpForOrganizationAndEvent`, `missingFollowUpIsNotFoundForRequestedOrganizationAndEvent`); API reported 9 tests, 4 failed (`memberCanViewFollowUp`, `managerCanViewFollowUp`, `nonMemberCannotViewFollowUp`, `missingFollowUpReturnsSharedNotFoundResponse`). `unauthenticatedViewRequestIsRejected` passed.
- **Failure reason:** The lookup and service tests reached the temporary compile stubs and failed with `UnsupportedOperationException`; authenticated GET API assertions expected 200/403/404 but received 500 because the GET mapping was not yet present. These demonstrate the missing planned repository query, service behavior, and endpoint rather than compilation or setup problems.
### Green (after implementation)

- **Result:** Focused suites passed:
  - `.\gradlew.bat --no-daemon --console=plain :Persistence:test --tests com.ubb.events.persistence.FollowUpRepositoryTest` — `BUILD SUCCESSFUL`.
  - `.\gradlew.bat --no-daemon --console=plain :Business:test --tests com.ubb.events.implementation.FollowUpServiceStructureTest` — `BUILD SUCCESSFUL`.
  - `.\gradlew.bat --no-daemon --console=plain :RestServices:test --tests com.ubb.events.controllers.FollowUpControllerApiTest` — `BUILD SUCCESSFUL`.
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain test` failed on two existing `OrganizationServiceStructureTest` assertions (`serviceInterfaceIsEmpty`, `implementationIsAnnotatedAndImplementsServiceWithoutOperations`). `.\gradlew.bat --no-daemon --console=plain test --continue` ran the remaining modules: Common, Model, and Persistence passed; RestServices reported 156 tests with one unrelated failure, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization` (`OrganizationController.list lacks org authorization`). These tests concern unchanged organization code; all T39 focused suites passed.
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

- **Decisions:** Use a Spring Data derived query through `FollowUp.event.id` and `FollowUp.event.organization.id` so retrieval is constrained to both requested identifiers; map to the existing `FollowUpResponse` and throw `NotFoundException` when absent. Authorize the GET endpoint at MEMBER level, which admits members and managers through `hasAtLeast`.
- **Alternatives considered:** Reusing `FollowUpResponse` avoids duplicating the API shape; no direct entity response is exposed.
- **Limitations:** The data model does not define uniqueness for follow-ups per event. The query returns an Optional and assumes at most one matching row; duplicate resolution is unspecified. The full backend suite retains unrelated failures in organization structure/authorization tests.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed (`56ecac5`; `T39-view-event-follow-up` pushed to `origin`)
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
