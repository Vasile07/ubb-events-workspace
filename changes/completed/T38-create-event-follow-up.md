# T38 - Create event follow-up

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T38
- **Type:** workflow
- **Owner:** Brudea Armand
- **Requirement(s):** LIF-04, LIF-05, ORGZ-06
- **Use case(s):** UC-13
- **Workflow:** workflow
- **Drive doc / ADR:** [Project requirements, use cases, architecture, ERD, and ADRs](https://docs.google.com/document/d/1yxuTw802yzVg7W2Efn1kozgppFnyh9umw7WgCmRutJ8/edit?tab=t.j04pz2q0fhhi#heading=h.3se70sg4uban) — UC-13 and requirements LIF-04, LIF-05, ORGZ-06.
- **Repo(s):** backend
- **Branch:** `T38-create-event-follow-up`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/39 (reviewed and merged)
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/FollowUpServiceStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/FollowUpControllerStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/FollowUpControllerApiTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Create event follow-up” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** LIF-04, LIF-05, ORGZ-06. **Depends on:** Organization-scoped authorization, Event lifecycle model, Init FollowUp entity.
- `POST /api/organizations/{organizationId}/events/{eventId}/follow-up`: any manager of the organization can create it, only when the event is completed (end date and time has passed) and not canceled. Deviates from LIF-05 and ORGZ-06, which say the creator only.
- Reject with the reason when the event is not completed or required information is missing.

Dependencies from the backlog: 9, 22, 37.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [x] The task is implemented according to the backlog specification above.
- [x] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/FollowUp.java` and `Persistence/src/main/java/com/ubb/events/persistence/FollowUpRepository.java` provide the persisted follow-up scaffold; `Business/src/main/java/com/ubb/events/FollowUpService.java` and `implementation/FollowUpServiceImpl.java`, plus `RestServices/src/main/java/com/ubb/events/controllers/FollowUpController.java`, are empty workflow scaffolds. `Model/.../domain/Event.java` stores date, nullable end time, and status; `Persistence/.../EventRepository.java` has organization-scoped event lookup. `RestServices/.../security/RequireOrgPermission.java` and `controllers/EventController.java` show manager authorization and API conventions. `Business/.../implementation/RegistrationServiceImpl.java` shows event lookup, error, and time handling patterns.
- **Related completed tasks:** T9 (Organization-scoped authorization) and T37 (Init FollowUp entity) are in `changes/completed/`.
- **Constraints / assumptions:** Backend only, Java 21/Spring Boot/JPA/JUnit 5/Gradle wrapper, layered architecture, and `master` are set in `context/project-context.md`. Backend repo `repos/ubb-events-backend` was clean on `master`; fetch/pull succeeded and it was up to date with `origin/master`. Use the existing Event model; do not depend on T22 or a `deleted` flag. `FollowUp.content` must be non-null and non-blank. Only managers of the organization may create follow-ups.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the follow-up creation path vertically through the existing backend layers. The controller will expose the organization-scoped POST endpoint and require `MANAGER`; the service will load the event within the organization, validate non-null/non-blank content and lifecycle, create the existing `FollowUp` entity, and return a response DTO. Use the existing `EventStatus.CANCELED` representation and consider an event completed only when its event date and `endTime` are before the current date/time. Preserve the shared error response shape.

### Steps

1. **Model/API contract:** define the minimal request and response DTOs for follow-up content and the created follow-up identifier/event reference; enforce required, non-blank content at the request boundary and preserve the shared error shape.
2. **Persistence:** extend the follow-up repository only as needed for the workflow (for example, an organization/event lookup if duplicate or ownership checks require it); retain event loading through an organization-scoped event query so an event from another organization is not used.
3. **Business layer:** implement `FollowUpService.createFollowUp(organizationId, eventId, request)`. Load the event or return not found, reject canceled events, reject events whose event date/`endTime` is not before now (including a missing end time), validate required content, create and save `FollowUp`, and map the saved entity to the response. Use an injectable clock or equivalent deterministic time seam so completion tests are stable.
4. **API layer:** add `POST /api/organizations/{organizationId}/events/{eventId}/follow-up` to `FollowUpController`, annotate it with `@RequireOrgPermission(EffectiveOrgRole.MANAGER)`, document success and error responses, and return HTTP 201 with the created follow-up.
5. **Verification:** write tests before implementation, obtain a behavioral red result, then run focused module tests and the full backend Gradle suite; record exact commands and outcomes in this task file.

### Files to create / modify

- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/CreateFollowUpRequest.java`
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/FollowUpResponse.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/FollowUpService.java`
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/FollowUpServiceImpl.java`
- `repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/FollowUpController.java`
- `repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/FollowUpRepository.java` (only if a workflow query is required)
- Corresponding tests under `Model/src/test`, `Business/src/test`, `Persistence/src/test`, and `RestServices/src/test`.

### Tests to write

- **Acceptance criterion: implementation follows the backlog:** a service test creates a follow-up for a completed, non-canceled event in the requested organization and verifies the persisted entity and response; a controller/API test verifies the exact POST path, 201 response, and manager-only organization annotation.
- **Acceptance criterion: behavior, validation, authorization, and error cases:** tests cover end time equal to or after now, missing end time, a canceled event, null/blank content, an unknown event, an event belonging to another organization, an organization manager, and a non-manager caller. The API test also verifies the shared error response/status for representative failures.
- Use a fixed clock/time seam in service tests and an integration/API test with the real security and error handling setup where the existing test infrastructure supports it.

### Risks / open questions

- The current organization interceptor supports the required manager-only authorization through `@RequireOrgPermission(EffectiveOrgRole.MANAGER)`.
- The user confirmed `FollowUp.content` is required and must be non-null and non-blank.
- The user confirmed only organization managers may create follow-ups. The event must be completed based on its `endTime` being earlier than the current time; no T22/deleted flag behavior applies.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `FollowUpServiceStructureTest` (8 tests) and `FollowUpControllerStructureTest`/`FollowUpControllerApiTest`.
- **Result:** `.\gradlew.bat --no-daemon --console=plain :Business:test --tests com.ubb.events.implementation.FollowUpServiceStructureTest` ran 8 tests; 7 failed behaviorally with `UnsupportedOperationException` from the compile-only service stub (`createsFollowUpOnlyForCompletedEvent`, `rejectsCanceledEventAndInvalidContent`, `rejectsUnknownOrWrongOrganizationEvent`, `rejectsEndTimeExactlyAtCurrentTime`, `rejectsEventThatHasNotEnded`, `rejectsNullContent`, `rejectsMissingEndTime`); the structure test passed. `.\gradlew.bat --no-daemon --console=plain :RestServices:test --tests com.ubb.events.controllers.FollowUpControllerStructureTest --tests com.ubb.events.controllers.FollowUpControllerApiTest` completed `BUILD SUCCESSFUL` with the service mocked, covering endpoint/security/payload behavior.
- **Failure reason:** The Business failures were behavioral failures caused by the missing follow-up creation implementation, not compilation or test setup failures. RestServices API tests passed independently because the service was mocked.

### Green (after implementation)

- **Result:** Focused tests passed: `.\gradlew.bat --no-daemon --console=plain :Business:test --tests com.ubb.events.implementation.FollowUpServiceStructureTest` — `BUILD SUCCESSFUL`; `.\gradlew.bat --no-daemon --console=plain :RestServices:test --tests com.ubb.events.controllers.FollowUpControllerStructureTest --tests com.ubb.events.controllers.FollowUpControllerApiTest` — `BUILD SUCCESSFUL`.
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain test` completed 115 tests but failed 2 unchanged existing tests in `OrganizationServiceStructureTest`: `serviceInterfaceIsEmpty()` (line 14) and `implementationIsAnnotatedAndImplementsServiceWithoutOperations()` (line 21). These failures concern the pre-existing `OrganizationService` methods and files, which T38 does not modify; T38 focused suites pass.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “Forget about T22 and the deleted flag. We are using the already existing model” | Removed T22 and the deleted flag from the plan; use the current event model and cancellation status. | Follow the existing model. |
| 2 | plan | “The 'content' field inside the FollowUp class is the content of that event's follow-up. This field should not be null or blank” | Made FollowUp content explicitly required and non-blank in the API/service plan and tests. | Enforce the specified required field. |
| 3 | plan | “Either a manager or the event's creator can create a follow-up” and “a follow-up for an event can be created if the event is completed (end_time < current time)” | Updated authorization and completion rules to allow managers or creators, and require end time to be before now. | Apply the clarified creation policy and lifecycle condition. |
| 4 | plan | “I made a mistake when I told you that the event's creator can also create follow-ups. Follow the task description and only allow managers of the organization to create follow-ups” | Removed the creator exception; require manager authorization for the endpoint and update the authorization tests. | Follow the task description's manager-only rule. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Expose the workflow at the organization-scoped POST endpoint and enforce manager authorization through the existing interceptor annotation. Keep business rules in `FollowUpServiceImpl`: content must be non-blank, the event must belong to the organization, must not be canceled, and its combined date/end time must be strictly before the injected clock time. Persist using the existing `FollowUp` entity and return a response DTO.
- **Alternatives considered:** Creator authorization and T22/deleted-flag lifecycle behavior were excluded after the user clarified manager-only authorization and use of the existing event model.
- **Limitations:** The full backend suite reports two failures in unchanged `OrganizationServiceStructureTest` assertions; T38 does not modify those files. Missing event end times are treated as incomplete and return a conflict. No duplicate-follow-up rule was specified or added.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed (`2e8e2e1`; `T38-create-event-follow-up` pushed to `origin`)
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
