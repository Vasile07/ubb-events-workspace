# T31 - Inspect event

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T31
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** DSC-03, DSC-04, EVT-04
- **Use case(s):** UC-10
- **Workflow:** workflow
- **Drive doc / ADR:** https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban
- **Repo(s):** backend
- **Branch:** `T31-inspect-event`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/33
- **Tests:** `EventServiceTest`, `RegistrationRepositoryTest`, `DiscoverEventControllerApiTest`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Inspect event” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** DSC-03, DSC-04, EVT-04. **Depends on:** Init Registration entity, Publish event.
- `GET /api/events/{eventId}`: title, description, organization, start and end date and time, location, registration information, status, whether registration is available, and the user's `registrationStatus` (NOT_REGISTERED, REGISTERED or CANCELLED).
- No task assignments and no participant lists in this response.
- Draft events and unknown ids return 404 for normal users. A deleted event is returned with the status CANCELED.

Dependencies from the backlog: 21, 28.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/DiscoverEventController.java` and `Business/.../EventService.java` provide the authenticated `/api/events` endpoint and service patterns; `Model/.../domain/Event.java`, `Persistence/.../EventRepository.java`, and event API/service tests cover event access and mapping. `Model/.../domain/Registration.java` currently has only `id`, `event`, and `user`; `RegistrationRepository` is empty, and `RegistrationService`/implementation are scaffolds.
- **Related completed tasks:** T21 (Init Registration entity), T28 (Publish event), T29 (Discover events), and T30 (Filter events).
- **Constraints / assumptions:** Backend only; task type is `workflow`, branch is `T31-inspect-event`, and backend `master` is clean and up to date. Project context requires an API-level test, service-owned business rules, and OpenAPI summaries/response codes. Per the user's clarification, Registration rows represent accepted registrations only; inspection should fetch `registration_id` and `user_id` from that table. When an event is cancelled, its status changes while its registrations remain. The response can therefore derive the requesting user's status from registration row presence and the event's status.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add a signed-in, non-organization-scoped `GET /api/events/{eventId}` vertical path. The controller will delegate to `EventService`; the service will load an event, enforce normal-user visibility, load registrations for the event and authenticated user, and map a detail DTO that contains the event's descriptive fields, organization summary, registration capacity/count, availability, lifecycle status, and the user's registration status. Registration rows represent accepted registrations; a matching row means `REGISTERED`. If the event is `CANCELED`, retained rows make the user's status `CANCELLED`; otherwise an absent row is `NOT_REGISTERED`. Draft and unknown events return 404 to normal users, while canceled events remain inspectable and are returned with `CANCELED`.

Keep task assignments and participant lists out of the DTO and query path. Registration availability is derived from the event lifecycle and remaining capacity (published, not canceled, and registered count below capacity); the exact JSON names for the registration-information/count fields should be confirmed during review because the backlog names the information but does not define a schema. This keeps lifecycle and registration rules in the service and HTTP mapping/OpenAPI metadata in the controller.

### Steps

1. **Tests first (Model/Business/API):** define the detail response contract and service/controller tests for published, draft, canceled, unknown, unauthenticated, registered, and unregistered cases; assert no task or participant fields are serialized and assert registration availability at capacity boundaries.
2. **Persistence:** extend `RegistrationRepository` with queries for registrations by event and by event/user (or equivalent projections) so the service can count accepted registrations and identify the authenticated user's row without loading participant lists.
3. **Business:** add an event-detail response DTO and `EventService.inspectEvent(eventId, userId)` (names aligned with repository conventions). Load the event, return `NotFoundException` for missing or `DRAFT` events, count registrations, resolve `registrationStatus` from event status plus row presence, and compute `registrationAvailable` from published lifecycle and remaining capacity. Preserve `CANCELED` events and their retained registrations.
4. **API/security:** add a `GET /api/events/{eventId}` handler, use the existing JWT-authenticated user id, document 200/401/404 responses with OpenAPI annotations, and leave organization membership checks out of this public event-detail route.
5. **Verification:** run focused tests after the required behavioral red result, then the affected backend suite; record exact commands and outcomes in T31's Test log.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/EventDetailsResponse.java` (new detail DTO and registration-status representation, or the project-approved equivalent)
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/RegistrationRepository.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/EventService.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/EventServiceImpl.java`
- `repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/EventServiceTest.java`
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/DiscoverEventController.java` (detail mapping alongside `/api/events`)
- `repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/DiscoverEventControllerApiTest.java`
- `repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/DiscoverEventControllerStructureTest.java` (if endpoint structure assertions require it)
- Related DTO/repository tests under `Model/src/test/java` and `Persistence/src/test/java` as needed for the chosen response and query mappings.

### Tests to write

- **Acceptance criterion: endpoint implements the backlog contract:** API test for an authenticated user receiving title, description, organization, date/time, location, registration information, status, availability, and `registrationStatus`; assert assignments, task data, and participant lists are absent.
- **Acceptance criterion: visibility and lifecycle rules:** API/service tests for unauthenticated 401, draft 404, unknown 404, published 200, and canceled 200 with `CANCELED`; canceled events retain registrations while the response resolves a registered user's status as `CANCELLED`.
- **Registration mapping and availability:** service/repository tests for a matching registration (`REGISTERED`), no matching row (`NOT_REGISTERED`), capacity reached (`registrationAvailable=false`), remaining capacity (`true`), and no participant-list loading.
- Write these tests before implementation and confirm a behavioral failure caused by the missing inspection path, then implement and rerun them to green. Include at least one API-level test as required by project context.

### Risks / open questions

- The backlog does not define the exact JSON shape or names for “registration information” and “whether registration is available.” The proposed mapping uses event capacity, accepted-registration count, and a boolean `registrationAvailable`; confirm/refine field names during Gate 1.
- The current authentication/interceptor conventions must expose the authenticated user id to the controller/service. Reuse the existing JWT security context; do not add organization authorization to this route.
- Event cancellation and registration cancellation are distinct: this plan follows the user's clarification that canceling an event changes only the event status and keeps registration rows. A later registration workflow may need its own status/history decision.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventServiceTest.inspectsPublishedEventWithRegistrationDetails` (focused behavioral red; API test added in `DiscoverEventControllerApiTest.inspectsEventForAuthenticatedUser`).
- **Result:** `.\gradlew.bat --offline --no-daemon :Business:test --tests "com.ubb.events.implementation.EventServiceTest.inspectsPublishedEventWithRegistrationDetails" --rerun-tasks --console=plain` — one test executed and failed with `UnsupportedOperationException: T31 not implemented`.
- **Failure reason:** The test compiled and reached the new service method; the failure demonstrates the planned inspection behavior is missing. Earlier attempts were setup/diagnostic retries and are not used as red evidence.

### Green (after implementation)

- **Result:** `.\gradlew.bat --offline --no-daemon :Business:test --tests "com.ubb.events.implementation.EventServiceTest.inspectsPublishedEventWithRegistrationDetails" --rerun-tasks --console=plain` — one test executed and passed (`BUILD SUCCESSFUL`). The user also reports running the app and tests and that everything seemed fine; exact commands and output were not provided.
- **Full suite of the affected repo:** Not run by the agent. The elevated full-suite command was rejected; the sandboxed Gradle invocation stalled before producing output or a test report. The user's report does not specify whether the full suite was included.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “The status field in Registration is outdated. In this project, the Registration db table holds only registrations that were accepted, or better said REGISTERED. So when we inspect an event let's fetch registration_id and user_id from registration table” | Replaced the assumption that T31 can read a stored Registration status with the clarified rule that a row represents an accepted registration and inspection should fetch registration/user IDs. | User clarified the current Registration data model. |
| 2 | plan | “In case of a cancelled event, only the event's status changes, but the registrations for that event remain in the table” | Updated the cancellation assumption: retained registration rows must be considered together with the event status when producing the inspection response. | User clarified cancellation behavior. |
| 3 | implementation | “I saw you used records instead of class for the newly created EventDetailsResponse dto, I believe there is a rule that restricts you from using records for dtos. Please confirm this rule and change the dto to be a class” | Converted `EventDetailsResponse` to a class and replaced record-style accessors in its tests. | `docs/coding-conventions.md` says to avoid Java records and use classes and interfaces for application types and DTOs. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Added an authenticated `GET /api/events/{eventId}` outside the organization-scoped route. Draft and unknown events return 404; canceled events remain inspectable. A registration row represents an accepted registration; for the current user, a row maps to `REGISTERED` on a non-canceled event and `CANCELLED` on a canceled event, while no row maps to `NOT_REGISTERED`. Registration availability is true only for published events with remaining capacity. The response omits task assignments and participant lists. `EventDetailsResponse` is an immutable class, following the backend coding convention.
- **Alternatives considered:** A registration status field was not added because the existing registration table stores accepted registrations only; event cancellation is represented by event status and retained rows. Task assignments and participant lists were omitted to keep the response within the task contract. A record DTO was replaced with a class to follow `docs/coding-conventions.md`.
- **Limitations:** The backlog does not define the JSON shape for registration information; this implementation exposes capacity, `registrationCount`, and `registrationAvailable`. The user reports running the app and tests successfully, but exact commands and output were not supplied.

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
