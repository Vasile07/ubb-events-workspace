# T29 - Discover events

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T29
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** DSC-01
- **Use case(s):** UC-09
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T29-discover-events`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/29 (depends on https://github.com/Vasile07/ubb-events-backend/pull/28)
- **Tests:** `DiscoverEventControllerApiTest`, `DiscoverEventControllerStructureTest` (new); `EventRepositoryTest`, `EventServiceTest`, `EventServiceStructureTest` (updated)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Discover events” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** DSC-01. **Depends on:** Publish event.
- `GET /api/events`: the list of events that are not drafts, with summary information. Completed events are included with the status COMPLETED, and deleted ones that had been published appear with the status CANCELED.
- "No results" returns an empty list. No pagination.

Dependencies from the backlog: 28.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code (backend):**
  - From T24/T28: `Persistence/.../EventRepository.java`, `Business/.../EventService.java` and `implementation/EventServiceImpl.java` (T28 added `publishEvent` and the `Clock` constructor parameter, on branch `T28-publish-event`, PR #28 open), `domain/Event.java`/`EventStatus.java`, `domain/Organization.java`.
  - `EventController` has root `/api/organizations`, so it can't host `/api/events`. `MyTaskController` (T27) is the pattern for a small controller outside the organization path.
  - `SecurityConfig`: `/api/**` requires authentication (401); only `/api/auth/**`, health and the API docs are public.
  - Test patterns: `EventControllerApiTest`, `EventServiceTest` (fixed `Clock`), `EventRepositoryTest`.
- **Related tasks:** T19 (Event), T24 (organization events, merged), T28 (publish, PR #28 open), T27 (`MyTaskController` pattern, PR #27 open). T30 (filters) will extend this endpoint.
- **Use case:** UC-09 Discover, search and filter events: "Only published events are returned (drafts and deleted events never)". UC-17: "A deleted event is no longer discoverable".
- **Constraints / assumptions (user decisions before planning):**
  - **Conflict between the backlog and the use cases:** the T29 backlog says canceled events that had been published appear as CANCELED, while UC-09/UC-17 say deleted events are never discoverable. The schema can't tell whether a canceled event was ever published. **User decision: return PUBLISHED events only**, following UC-09/UC-17. No schema change.
  - **COMPLETED:** **user decision: not derived yet**. Past published events keep their stored status `PUBLISHED`; deriving COMPLETED is left to T22.
  - **Sign-in:** **user decision: required**. This is the existing `/api/**` rule, with no `SecurityConfig` change.
  - **Response:** **user decision: summary information, chronological**: `id, title, date, startTime, endTime, location, status, organization {id, name}`, with no description, capacity or tasks.
  - **Branching:** the backend branch `T29-discover-events` is created from `T28-publish-event`, because both change `EventService`/`EventServiceImpl`/`EventRepository`. Merge order: #28 → T29, with merge commits.
  - **Pre-existing failing tests on `master`:** 3 from T13/T14.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Add `GET /api/events` for any signed-in user (401 without a token), outside the organization path. It returns **all PUBLISHED events of all organizations**, ordered chronologically by `date` and then `startTime`. "No results" is a 200 with an empty list, and there is no pagination (backlog).

Each item is an `EventSummaryResponse`:
- `{ id, title, date, startTime, endTime, location, status, organization: { id, name } }`;
- there is no description, capacity or tasks;
- in particular, tasks are **never** included, which satisfies T25's "never returned by the public event endpoints".

The handler lives in a new small `DiscoverEventController` (`@RequestMapping("/api/events")`), following T27's `MyTaskController`. The logic goes in `EventService.discoverEvents()`, and a new repository query `findAllByStatusOrderByDateAscStartTimeAsc(EventStatus)` filters in the database. T30 will add search and filter parameters to this same endpoint.

**Alternatives considered:**
- Including CANCELED events, or adding a "was published" column: rejected by the user, following UC-09/UC-17.
- Deriving COMPLETED now: rejected by the user, left to T22.
- A public (anonymous) endpoint: rejected by the user.
- Reusing `EventResponse`: rejected, because the backlog asks for summary information and the organization name is needed for display.
- Putting the handler in `EventController`: not possible, because its root is `/api/organizations`.

### Steps

1. **Model (DTOs):** `dto/EventSummaryResponse { id, title, date, startTime, endTime, location, status, organization }` and `dto/EventOrganizationSummaryResponse { id, name }`, both `final` classes with `@Getter @AllArgsConstructor @EqualsAndHashCode`.
2. **Persistence:** add `List<Event> findAllByStatusOrderByDateAscStartTimeAsc(EventStatus status)` to `EventRepository`.
3. **Business:** add `List<EventSummaryResponse> discoverEvents()` to `EventService`. It calls the query with `PUBLISHED` and maps the results.
4. **API:** new `DiscoverEventController` (`@RestController`, `@RequestMapping("/api/events")`, constructor-injected `EventService`) with `discoverEvents()` on `GET`, and OpenAPI `@Operation` with `@ApiResponses` for 200 and 401. There is no org-permission annotation, because the path is outside `/api/organizations`.
5. **Structure tests:** extend `EventServiceStructureTest` and add `DiscoverEventControllerStructureTest`.
6. **Verification:** tests first and a behavioral red, then implement and get the focused tests green. Then run the full suite with `--continue`; the only failures should be the 3 known pre-existing ones.

### Files to create / modify

Backend (`repos/ubb-events-backend/`, branch `T29-discover-events` from `T28-publish-event`):

- Create `Model/src/main/java/com/ubb/events/dto/EventSummaryResponse.java`, `EventOrganizationSummaryResponse.java`
- Modify `Persistence/src/main/java/com/ubb/events/persistence/EventRepository.java`
- Modify `Persistence/src/test/java/com/ubb/events/persistence/EventRepositoryTest.java`
- Modify `Business/src/main/java/com/ubb/events/EventService.java`, `Business/src/main/java/com/ubb/events/implementation/EventServiceImpl.java`
- Modify `Business/src/test/java/com/ubb/events/implementation/EventServiceTest.java`, `EventServiceStructureTest.java`
- Create `RestServices/src/main/java/com/ubb/events/controllers/DiscoverEventController.java`
- Create `RestServices/src/test/java/com/ubb/events/controllers/DiscoverEventControllerApiTest.java`, `DiscoverEventControllerStructureTest.java`

### Tests to write

Acceptance criteria → tests:

- **AC1: per the backlog and the user decisions (published events only, summary information, empty list for no results, no pagination):**
  - `EventRepositoryTest.findsOnlyPublishedEventsOfAllOrganizationsChronologically`: DRAFT and CANCELED are excluded, events from two organizations are included, and the order is by date and then start time.
  - `EventServiceTest.discoversPublishedEventsAsSummaries`: the mapping including the organization name, and the query is called with `PUBLISHED`.
  - `EventServiceTest.discoverReturnsEmptyListWhenNothingPublished`.
  - `DiscoverEventControllerApiTest.signedInUserDiscoversPublishedEvents`: 200 with the summary fields and `organization.name`; no `description`, `capacity` or `tasks`.
  - `DiscoverEventControllerApiTest.noResultsReturnsEmptyList`: 200 with `[]`.
  - `DiscoverEventControllerApiTest.doesNotRequireOrganizationMembership`: the authorization service is never called.
- **AC2: authentication:**
  - `DiscoverEventControllerApiTest.unauthenticatedRequestIsRejected`: 401, and the service is not called.
- **Structure:** `DiscoverEventControllerStructureTest` (root `/api/events`, a GET handler without org annotations); `EventServiceStructureTest` (`discoverEvents`).
- **Expected red:**
  - complete DTOs and the declared repository method;
  - `discoverEvents()` returns an empty list;
  - the controller exists at `/api/events`, but its handler has no `@GetMapping`.

  The failures will be assertions.

### Risks / open questions

1. **The backlog vs UC-09/UC-17:** resolved by the user (PUBLISHED only). The T29 backlog text still mentions COMPLETED and CANCELED; the team should update it or record the decision in Drive.
2. **COMPLETED:** past events are still returned, with status `PUBLISHED`. When T22 derives COMPLETED, this endpoint's mapping is the place to apply it.
3. **Volume:** there is no pagination (backlog), so every published event is returned at once. This is acceptable for a university project.
4. **Branch dependency:** T29 depends on #28 (T28) being merged first, with a merge commit.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `EventRepositoryTest` (1 new), `EventServiceTest` (2 new), `EventServiceStructureTest` (1 new), `DiscoverEventControllerApiTest` (new), `DiscoverEventControllerStructureTest` (new).
- **Stubs used:**
  - complete `EventSummaryResponse` and `EventOrganizationSummaryResponse` DTOs;
  - `EventRepository.findAllByStatusOrderByDateAscStartTimeAsc` declared;
  - `EventServiceImpl.discoverEvents()` returning `List.of()` without querying;
  - `DiscoverEventController` at `/api/events` with a `discoverEvents()` handler that has no `@GetMapping`.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Persistence:test --tests "com.ubb.events.persistence.EventRepositoryTest" :Business:test --tests "com.ubb.events.implementation.EventServiceTest" --tests "com.ubb.events.implementation.EventServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.DiscoverEventControllerApiTest" --tests "com.ubb.events.controllers.DiscoverEventControllerStructureTest"`
- **Result:** `BUILD FAILED`, everything compiled, and all 6 new behavior tests failed.
  - `EventServiceTest`:
    - `discoversPublishedEventsAsSummaries` (`AssertionFailedError` at `:67`, empty list);
    - `discoverReturnsEmptyListWhenNothingPublished` (Mockito `verify`: the repository was never queried).
  - `DiscoverEventControllerApiTest`: `signedInUserDiscoversPublishedEvents`, `noResultsReturnsEmptyList`, `doesNotRequireOrganizationMembership` (`AssertionError` on the status code, because `GET /api/events` is not mapped).
  - `DiscoverEventControllerStructureTest.discoverHandlerIsGetWithoutOrganizationAnnotations` (`:26`, no `@GetMapping`).
- **Passing at red, as expected:**
  - `EventRepositoryTest` (6 of 6, including the new `findsOnlyPublishedEventsOfAllOrganizationsChronologically`): Spring Data derives the query from the declared name;
  - `EventServiceStructureTest` (4 of 4): the stub declares `discoverEvents`;
  - the existing T24/T28 tests in `EventServiceTest`;
  - `DiscoverEventControllerStructureTest.controllerIsMappedOutsideOrganizationPathWithServiceDependency`: the stub already had the class structure;
  - `DiscoverEventControllerApiTest.unauthenticatedRequestIsRejected`: the existing security config returns 401.
- **Failure reason:** behavioral. The failures come from the missing planned functionality: published events are not queried or mapped, and the endpoint is not mapped. There were no compilation or setup errors.

### Green (after implementation)

- **Result:** the same focused command gives `BUILD SUCCESSFUL`, and all 32 tests in the focused classes pass (`EventRepositoryTest` 6, `EventServiceTest` 16, `EventServiceStructureTest` 4, `DiscoverEventControllerApiTest` 4, `DiscoverEventControllerStructureTest` 2).
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives `BUILD FAILED`: 216 tests, 3 failures, 0 errors, 0 skipped. The 3 failures are the **pre-existing** T13/T14 ones (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`). There are no new failures, and all T24/T28 tests still pass.

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

<!-- DRAFT written by the implementer. Verify and correct before using it as a study note. -->
- **Decisions:**
  - `GET /api/events` is for any signed-in user (401 without a token), outside the organization path, with no membership check.
  - It returns **only PUBLISHED events** of all organizations (user decision, following UC-09 "drafts and deleted events never" and UC-17 "a deleted event is no longer discoverable"). This resolves the conflict with the T29 backlog, which wanted previously published CANCELED events too; the schema can't tell whether a canceled event was ever published.
  - The filtering happens in the database (`findAllByStatusOrderByDateAscStartTimeAsc(PUBLISHED)`), chronologically by `date` and then `startTime`. There is no pagination, and an empty list means no results.
  - The response is an `EventSummaryResponse { id, title, date, startTime, endTime, location, status, organization { id, name } }` (user decision). There is no description, capacity or tasks; tasks are never exposed by the public endpoints (T25's rule).
  - COMPLETED is **not derived** (user decision): past published events keep the status `PUBLISHED` until T22.
  - A new `DiscoverEventController` (`/api/events`) follows T27's `MyTaskController`. The logic lives in `EventService.discoverEvents()`, so T30 can add search and filters to the same endpoint.
- **Alternatives considered:**
  - Including CANCELED events, or adding a "was published" column: rejected by the user.
  - Deriving COMPLETED now: rejected by the user.
  - An anonymous endpoint: rejected by the user.
  - Reusing `EventResponse`: rejected, because the backlog asks for summary information plus the organization name.
- **Limitations:**
  - Past events are still listed, as `PUBLISHED`, until T22 derives COMPLETED.
  - No pagination, as the backlog says.
  - The T29 backlog text still mentions COMPLETED and CANCELED; the team should align it with the user's decision.
  - The API tests mock the service; the query is covered by `EventRepositoryTest`.
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