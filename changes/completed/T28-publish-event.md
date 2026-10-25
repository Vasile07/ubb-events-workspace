# T28 - Publish event

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T28
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** EVT-03, EVT-06, LIF-02
- **Use case(s):** UC-08
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T28-publish-event`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/28
- **Tests:** `ClockConfigTest` (new); `EventServiceTest`, `EventServiceStructureTest`, `EventControllerApiTest`, `EventControllerStructureTest` (updated)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Publish event” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** EVT-03, EVT-06, LIF-02. **Depends on:** Organization-scoped authorization, Event lifecycle model, Create an event with tasks.
- `PATCH /api/organizations/{organizationId}/events/{eventId}/publish`, for managers: validate the required details (including the end date and time), change the status from DRAFT to PUBLISHED, and return the updated event.
- Reject incomplete events with what to correct, and reject an event that is already published or canceled.

Dependencies from the backlog: 9, 22, 23.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code (backend, `master` after T24/T25):**
  - `RestServices/.../controllers/EventController.java` (`/api/organizations`, T24 GET handlers with `@RequireOrgPermission`), `Business/.../EventService.java` and `implementation/EventServiceImpl.java` (T24 `listOrganizationEvents`/`getOrganizationEvent`, `toResponse` mapping, `NotFoundException("Event not found")`), `Persistence/.../EventRepository.java` (`findByIdAndOrganizationId`), `Model/.../dto/EventResponse.java`, `domain/Event.java`/`EventStatus.java`.
  - Errors: `Common/.../errors/BusinessRuleException` and `ConflictException` (both can carry `List<ErrorDetail>`), `dto/ErrorDetail(field, message)`, and `GlobalExceptionHandler` (BusinessRule → 400, Conflict → 409, NotFound → 404, each with `errors`).
  - Manager-only pattern: `MembershipController` (`@RequireOrgPermission(EffectiveOrgRole.MANAGER)`, `PATCH`), and `MembershipServiceImpl` (`@Transactional`; "not pending" → `ConflictException` 409).
  - Tests: `EventControllerApiTest`, `EventServiceTest`, `EventControllerStructureTest` (T24).
- **Related tasks:** T9 (authorization), T19 (Event), T24 (view organization events, merged).
- **Use case:** UC-08 Publish event:
  - only a draft can be published;
  - publication requires complete required details;
  - missing or invalid information → "the system prevents publication and indicates what to correct";
  - the user lacks permission → denied.
- **Constraints / assumptions:**
  - **T22 (lifecycle) and T23 (create) are not implemented** (same user decision as T24–T27). The DRAFT → PUBLISHED transition and its checks are implemented in `EventServiceImpl`; T22 can later move them into a lifecycle component.
  - **Branching:** T28 branches from `master`, which already contains T24/T25. It does not need T27.
  - **Schema:** `events` has one `date` plus `start_time` and a nullable `end_time`. The "end date and time" from the backlog is therefore the `end_time` on the same `date`.
  - **Pre-existing failing tests on `master`:** 3 from T13/T14 (recorded in T24).

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add `PATCH /api/organizations/{organizationId}/events/{eventId}/publish` to `EventController`, for **managers** only (`@RequireOrgPermission(EffectiveOrgRole.MANAGER)`; a member gets 403 from the T9 interceptor).

`EventService.publishEvent(organizationId, eventId)` is `@Transactional` and works in this order:

1. Loads the event with `findByIdAndOrganizationId`. If it's missing or belongs to another organization: 404 "Event not found" (as in T24).
2. Checks the state:
   - `PUBLISHED` → **409** `ConflictException("Event is already published")`;
   - `CANCELED` → **409** `ConflictException("A canceled event cannot be published")`.

   This is the same style as membership's "only pending requests can be decided" → 409.
3. Validates the required details and collects **every** problem as an `ErrorDetail(field, message)`. If any exist: **400** `BusinessRuleException("Event is not ready to be published", errors)`, so the response tells the manager exactly what to correct.
4. Sets `status = PUBLISHED`, saves, and returns the updated `EventResponse` (200).

**Required for publication (proposed, open question 1):**
- `title` and `description` not blank;
- `date` and `startTime` present;
- `endTime` present and after `startTime`;
- `location` not blank;
- `capacity` present and > 0 (UC-06: "Capacity is a positive number").

**Alternatives considered:**
- 400 instead of 409 for already-published or canceled events: rejected, so that "wrong state" (409, consistent with memberships) stays distinguishable from "fix these fields" (400 with field errors).
- A separate lifecycle class now: deferred to T22, which owns it; T28 keeps one private check in the service.
- Rejecting events in the past (open question 2): UC-06 lists date rules as an open decision.

### Steps

1. **Business:** add `EventResponse publishEvent(Long organizationId, Long eventId)` to `EventService`. In `EventServiceImpl`, add `@Transactional publishEvent` with the lookup → state check → field validation → status change and `save` → `toResponse` (reusing T24's mapping).
2. **API:** add `publishEvent` to `EventController`: `@PatchMapping("/{organizationId}/events/{eventId}/publish")`, `@RequireOrgPermission(EffectiveOrgRole.MANAGER)`, and OpenAPI `@Operation` with `@ApiResponses` for 200, 400, 401, 403, 404 and 409.
3. **Structure test:** extend `EventControllerStructureTest` (publish handler: PATCH path, MANAGER) and `EventServiceStructureTest` (the `publishEvent` method).
4. **Verification:** tests first and a behavioral red, then implement and get the focused tests green. Then run the full suite with `--continue`; the only failures should be the 3 known pre-existing ones.

### Files to create / modify

Backend (`repos/ubb-events-backend/`, branch `T28-publish-event` from `master`):

- Modify `Business/src/main/java/com/ubb/events/EventService.java`, `Business/src/main/java/com/ubb/events/implementation/EventServiceImpl.java`
- Modify `Business/src/test/java/com/ubb/events/implementation/EventServiceTest.java`, `EventServiceStructureTest.java`
- Modify `RestServices/src/main/java/com/ubb/events/controllers/EventController.java`
- Modify `RestServices/src/test/java/com/ubb/events/controllers/EventControllerApiTest.java`, `EventControllerStructureTest.java`

No persistence or DTO changes are needed: the existing `save` and `EventResponse` are reused.

### Tests to write

Acceptance criteria → tests:

- **AC1: managers publish a complete draft, which becomes PUBLISHED and is returned:**
  - `EventServiceTest.publishesCompleteDraftAndReturnsUpdatedEvent`: the status becomes PUBLISHED, the event is saved, and the response maps it.
  - `EventControllerApiTest.managerPublishesDraft`: 200 with `status: "PUBLISHED"`.
- **AC2: rejections and authorization:**
  - `EventServiceTest.rejectsIncompleteDraftListingEveryFieldToCorrect`: null `endTime`, blank `location`, null `capacity` → `BusinessRuleException` with 3 `ErrorDetail` entries (`endTime`, `location`, `capacity`); nothing is saved.
  - `EventServiceTest.rejectsEndTimeNotAfterStartTime`: `endTime` equal to or before `startTime` → an error on `endTime`.
  - `EventServiceTest.rejectsNonPositiveCapacity`.
  - `EventServiceTest.rejectsBlankTitleOrDescription`.
  - `EventServiceTest.rejectsAlreadyPublishedEvent` and `rejectsCanceledEvent`: `ConflictException`; nothing is saved.
  - `EventServiceTest.throwsNotFoundForEventOfAnotherOrganization`.
  - `EventControllerApiTest.incompleteDraftReturnsBadRequestWithFieldErrors`: 400, `message`, and `errors[].field`/`errors[].message`.
  - `EventControllerApiTest.alreadyPublishedOrCanceledReturnsConflict`: 409.
  - `EventControllerApiTest.unknownEventReturnsNotFoundOnPublish`: 404.
  - `EventControllerApiTest.memberCannotPublish`: 403, checked at **MANAGER** level; the service is not called.
  - `EventControllerApiTest.unauthenticatedPublishIsRejected`: 401.
- **Structure:** the publish handler is `PATCH /{organizationId}/events/{eventId}/publish` with MANAGER; the service declares `publishEvent`.
- **Expected red:** the stub `publishEvent` returns `null` without loading or validating, and the controller handler has no `@PatchMapping`/`@RequireOrgPermission`. The failures will be assertions, not compilation errors.

### Risks / open questions

User decisions at Gate 1:

1. **Required fields for publishing:** title, description, date, start time, **end time (after the start)**, **location**, and **capacity > 0**. **Confirmed by the user.**
2. **Date in the past:** **changed by the user** ("check so that an event can't be created in the past"). T28 can only enforce this at **publication**, so publishing is rejected when `date` + `startTime` is not in the future. It becomes one more `ErrorDetail` on `date` in the 400 response. Enforcing it at **creation** belongs to T23 (Karina); this is noted in the T28 PR so that T23 adds the same check.
   - **Implementation detail:** the current time comes from a `java.time.Clock` bean (new `Common/.../configs/ClockConfig.java`, `Clock.system(ZoneId.of("Europe/Bucharest"))`, because event times are local times in Cluj). It is injected into `EventServiceImpl`, so the tests can use a fixed clock.
3. **Status codes:** already published or canceled → **409**; incomplete → **400** with the list of fields to correct; not found → 404; a member (not a manager) → 403. **Confirmed by the user.**
4. **Same-day events only:** the schema has a single `date`, so `endTime` must be after `startTime` on that day, and events past midnight can't be represented. This is a schema limitation outside T28. **Confirmed by the user.**

Plan additions from decision 2:
- **Files:** create `Common/src/main/java/com/ubb/events/common/configs/ClockConfig.java` and `Common/src/test/java/com/ubb/events/common/configs/ClockConfigTest.java`. `EventServiceImpl` gets `Clock` as an extra constructor parameter, so `EventServiceStructureTest` is updated.
- **Tests:**
  - `EventServiceTest.rejectsPublishingEventThatStartsInThePast`: an error on `date`, nothing saved.
  - `EventServiceTest.acceptsEventStartingLaterToday`: a boundary case with a fixed clock.
  - `ClockConfigTest`: the bean uses `Europe/Bucharest`.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `ClockConfigTest` (new), `EventServiceTest` (10 new), `EventServiceStructureTest` (updated), `EventControllerApiTest` (6 new), `EventControllerStructureTest` (1 new).
- **Stubs used:**
  - `ClockConfig` returning `Clock.systemUTC()`, the wrong zone;
  - `EventService.publishEvent` declared;
  - `EventServiceImpl(EventRepository, Clock)` with a `publishEvent` that returns `null` (no lookup, state check, validation or save, and no `@Transactional`);
  - an `EventController.publishEvent` handler without `@PatchMapping`/`@RequireOrgPermission`.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Common:test --tests "com.ubb.events.common.configs.ClockConfigTest" :Business:test --tests "com.ubb.events.implementation.EventServiceTest" --tests "com.ubb.events.implementation.EventServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.EventControllerApiTest" --tests "com.ubb.events.controllers.EventControllerStructureTest"`
- **Result:** `BUILD FAILED`, everything compiled, 18 of 34 tests failed.
  - `ClockConfigTest.exposesClockInClujLocalTime` (`AssertionFailedError`: zone `Z` instead of `Europe/Bucharest`).
  - `EventServiceStructureTest.publishIsTransactional` (no `@Transactional`).
  - `EventServiceTest`, all 10 publish tests failed:
    - `publishesCompleteDraftAndReturnsUpdatedEvent`, `rejectsIncompleteDraftListingEveryFieldToCorrect`, `rejectsEndTimeNotAfterStartTime`, `rejectsNonPositiveCapacity`, `rejectsBlankTitleOrDescription`, `rejectsPublishingEventThatStartsInThePast`, `rejectsAlreadyPublishedEvent`, `rejectsCanceledEvent`, `throwsNotFoundWhenPublishingEventOfAnotherOrganization` (`AssertionFailedError`: expected exceptions not thrown, or a null result);
    - `acceptsEventStartingLaterToday` (`NullPointerException` at `:147`, because the stub returned `null` instead of the published event).
  - `EventControllerApiTest`: `memberCannotPublish`, `managerPublishesDraft`, `incompleteDraftReturnsBadRequestWithFieldErrors`, `alreadyPublishedOrCanceledReturnsConflict`, `unknownEventReturnsNotFoundOnPublish` (`AssertionError` on the status code: no PATCH handler is mapped).
  - `EventControllerStructureTest.publishHandlerIsMappedForManagersOnly` (no `@PatchMapping`).
- **Passing at red, as expected:**
  - all existing T24 tests in these classes;
  - `EventControllerApiTest.unauthenticatedPublishIsRejected`: the security config already returns 401.
- **Failure reason:** behavioral. The failures come from the missing planned functionality: no lookup, no state transition checks, no field or date validation, no status change, no `@Transactional`, the wrong clock zone, and an unmapped handler. There were no compilation or setup errors. The NPE is a direct consequence of the missing return value, not a test defect.

### Green (after implementation)

- **Result:** the same focused command gives `BUILD SUCCESSFUL`, and all 34 tests in the focused classes pass (`ClockConfigTest` 1, `EventServiceTest` 14, `EventServiceStructureTest` 3, `EventControllerApiTest` 12, `EventControllerStructureTest` 4).
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives `BUILD FAILED`: 206 tests, 3 failures, 0 errors, 0 skipped. The 3 failures are the **pre-existing** T13/T14 ones (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`). There are no new failures. The Spring context starts with the new `Clock` bean, so all `@SpringBootTest` classes pass.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | The plan proposed not checking whether the event date is in the past ("UC-06 leaves date rules open"). | The user asked: "check so that an event can't be created in the past". T28 now rejects publishing an event whose date and start time are not in the future (400, error on `date`), using an injected `Clock` (Europe/Bucharest). Enforcement at creation is noted for T23. | Events in the past must not be accepted. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

<!-- DRAFT written by the implementer. Verify and correct before using it as a study note. -->
- **Decisions:**
  - `PATCH /api/organizations/{organizationId}/events/{eventId}/publish` is for **managers only**: `@RequireOrgPermission(MANAGER)`, checked by the T9 interceptor against the current membership in the database. The JWT only carries the user id.
  - `EventService.publishEvent` is `@Transactional` and works in this order:
    1. 404 if the event isn't in this organization;
    2. state check: PUBLISHED or CANCELED → **409** (`ConflictException`, consistent with memberships);
    3. field validation that collects **every** problem as an `ErrorDetail(field, message)` → **400** "Event is not ready to be published" with the full list (UC-08: "indicates what to correct");
    4. DRAFT → PUBLISHED, saved and returned as `EventResponse`.
  - Required for publication (user decision): title and description not blank, date, start time, end time after the start time, location not blank, capacity > 0.
  - **No events in the past** (Human Correction #1): the start (`date` + `startTime`) must be strictly in the future, measured with an injected `java.time.Clock` (`ClockConfig`, `Europe/Bucharest`, because event times are Cluj local times). The tests use a fixed clock for deterministic boundary cases (one minute later is accepted; earlier today and "exactly now" are rejected).
  - The transition rule lives in `EventServiceImpl` because T22 (lifecycle) doesn't exist yet; T22 can extract it.
- **Alternatives considered:**
  - 400 for wrong-state events: rejected, so "fix these fields" (400) stays distinguishable from "not possible in this state" (409).
  - Stopping at the first invalid field: rejected, because the manager should see everything to correct at once.
  - `LocalDateTime.now()` without a clock: rejected, because it isn't testable and depends on the server time zone (UTC in Docker).
  - Not checking past dates: changed by the user.
- **Limitations:**
  - The past-date rule is only enforced at **publication**. Creating or editing a draft in the past is still possible until T23 (create) and T26 (edit) add the same check; this is noted for Karina in the PR.
  - Events can't cross midnight, because the schema has a single `date` (user-confirmed).
  - The API tests mock the service, as in the existing `*ControllerApiTest` classes; the rules are covered by `EventServiceTest`.
  - The full suite still has the 3 pre-existing T13/T14 failures.

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
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [x] Moved to `changes/completed/` (at Gate 2, by user preference; PR review and Gate 3 still pending)