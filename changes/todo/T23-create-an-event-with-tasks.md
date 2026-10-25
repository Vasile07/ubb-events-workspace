# T23 - Create an event with tasks

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T23
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):** EVT-01, EVT-06, ORGZ-01, LIF-01
- **Use case(s):** UC-06
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T23-create-an-event-with-tasks`
- **PR link(s):** 
- **Tests:** (test classes/files, filled at the end)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Create an event with tasks” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** EVT-01, EVT-06, ORGZ-01, LIF-01. **Depends on:** Init Membership entity, Organization-scoped authorization, Init Event entity, Init EventTask entity, Event lifecycle model.
- `POST /api/organizations/{organizationId}/events`, for managers: create an event as a draft with validation (title, description, start and end date and time with the end after the start, location, capacity, registration information). Returns 201 with the event id and the draft status.
- Create written tasks on the event and assign them to members, rejecting a non-member of the organizing organization.

Dependencies from the backlog: 7, 9, 19, 20, 22.

**Validation rules moved here from T28 (publish event).** Publishing only checks that the event starts in the future, so creating an event must enforce these (decided by Blaj Horatiu in T28, Human Corrections #2):

- `title` is required and not blank.
- `description` is required and not blank.
- `date` and `startTime` are required, and the start (`date` + `startTime`) must be **in the future**, measured with the injected `java.time.Clock` (`ClockConfig`, Europe/Bucharest). Never call `LocalDateTime.now()` without the clock.
- `endTime` is required and must be **after `startTime`** (same day; events can't cross midnight, because the schema has a single `date`).
- `location` is required and not blank.
- `capacity` is required and must be **> 0**.
- Collect **every** problem as an `ErrorDetail(field, message)` and throw `BusinessRuleException("…", errors)` → 400, so the manager sees everything to correct at once. Use the field names `title`, `description`, `date`, `startTime`, `endTime`, `location`, `capacity`, the same names T28 used.
- Put the checks in a reusable private method in `EventServiceImpl` (for example `validateEventDetails(...)`), because T26 (edit) must apply exactly the same rules.

**Alignment with the work already done (T19, T20, T24, T25, T27–T30):**

- **Entities:**
  - `Event` (T19): organization `@ManyToOne`, `categoryId` (a temporary scalar `Long`; there is no Category entity yet), `title`, `description`, `date`, `startTime`, `endTime`, `location`, `capacity`, and `status` (`EventStatus`: DRAFT, PUBLISHED, CANCELED). There is **no** `deleted` flag and **no** creator field; CANCELED means canceled or deleted.
  - `EventTask` (T20): `description` (required), `assignedUser` (`@ManyToOne User`, nullable), `event` (required). There is no title, due date or status.
- **Where to put the code:**
  - `POST "/{organizationId}/events"` goes into the existing `EventController` (root `/api/organizations`), with `@RequireOrgPermission(EffectiveOrgRole.MANAGER)` and OpenAPI `@Operation` + `@ApiResponses` (201, 400, 401, 403).
  - The logic goes into `EventService`/`EventServiceImpl` (`@Transactional`).
  - Return **201** with the existing `EventResponse` DTO; the status is always **DRAFT**.
- **Tasks created with the event:** each assignee is optional. If an assignee is given, it must be an **ACTIVE member** of the event's organization (`MembershipRepository.findByUserIdAndOrganizationId` + `MembershipStatus.ACTIVE`). Otherwise return 400 with an `ErrorDetail` and save nothing (the whole creation is one transaction). Reuse `EventTaskRepository` and the `TaskAssigneeResponse` style (`{id, firstName, lastName}`, never email or password).
- **Errors:** validation → `BusinessRuleException` (400, with an `ErrorDetail` list); wrong state → `ConflictException` (409); unknown or foreign event → `NotFoundException("Event not found")` (404). Authorization stays in the T9 interceptor; the JWT carries only the user id.
- **Never expose tasks** on public endpoints (`GET /api/events` from T29/T30).
- **Code style:**
  - DTOs are `final` classes with Lombok `@Getter @AllArgsConstructor @EqualsAndHashCode` (no records); use constructor injection.
  - Tests come first (TDD):
    - Mockito service tests with a fixed `Clock` (`Clock.fixed(..., ZoneId.of("Europe/Bucharest"))`);
    - `@DataJpaTest` repository tests that call `TestEntityManager.clear()` before reloading;
    - MockMvc API tests with a real JWT (`JwtTokenService.issue(userId)`) and `@MockitoBean OrganizationAuthorizationService`;
    - update the `EventControllerStructureTest`/`EventServiceStructureTest` classes when signatures change.
- **Branching:** branch `T23-create-an-event-with-tasks` from `master` **after** backend PRs #28 → #29 → #30 are merged, because #28 brings the `Clock` bean and they all change `EventService`/`EventController`. If they're not merged yet, branch from `T30-filter-events` and ask the user. Merge with "Create a merge commit", never squash.
- **Full suite:** `.\gradlew.bat --no-daemon --console=plain --continue clean test`. There are 3 **pre-existing** failures on `master` that are not T23's (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`).
- **Keep it consistent for T22 and T26:**
  - Leave the DRAFT → PUBLISHED rule in `publishEvent` (T28).
  - An event counts as "ended" when `date` + `endTime` (or `startTime` if missing) is not after now (T30 ordering); T22 should derive COMPLETED with the same definition.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** 
- **Related completed tasks:** 
- **Constraints / assumptions:** 

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

### Steps

1. 

### Files to create / modify

### Tests to write

### Risks / open questions

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** 
- **Result:** 
- **Failure reason:** 

### Green (after implementation)

- **Result:** 
- **Full suite of the affected repo:** 

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

- **Decisions:** 
- **Alternatives considered:** 
- **Limitations:** 

## Status

- [ ] Task read and context gathered
- [ ] Plan written
- [ ] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [ ] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [ ] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`