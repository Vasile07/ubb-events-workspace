# T30 - Filter events

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T30
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** DSC-02, DSC-05
- **Use case(s):** UC-09
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T30-filter-events`
- **PR link(s):** https://github.com/Vasile07/ubb-events-backend/pull/30 (depends on https://github.com/Vasile07/ubb-events-backend/pull/28 and https://github.com/Vasile07/ubb-events-backend/pull/29)
- **Tests:** `EventRepositoryTest`, `EventServiceTest`, `EventServiceStructureTest`, `DiscoverEventControllerApiTest`, `DiscoverEventControllerStructureTest` (updated)

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Filter events” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** DSC-02, DSC-05. **Depends on:** Discover events.
- Extend the list endpoint with a search term (event title or organization name), an organization filter and a date range filter (start and end date).
- Removing the criteria returns the full list.

Dependencies from the backlog: 29.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code (backend, branch `T29-discover-events`, PR #29 open):**
  - `RestServices/.../controllers/DiscoverEventController.java` (`GET /api/events`);
  - `Business/.../EventService.discoverEvents()` with `EventServiceImpl.toSummaryResponse`;
  - `Persistence/.../EventRepository.findAllByStatusOrderByDateAscStartTimeAsc`;
  - DTOs `EventSummaryResponse` and `EventOrganizationSummaryResponse`;
  - their tests (`DiscoverEventControllerApiTest`, `EventServiceTest`, `EventRepositoryTest`).
  - Optional query parameter pattern: `MembershipController.listMembershipRequests` (`@RequestParam(required = false)`). Invalid parameter types give 400 "Request validation failed" with the field name, through `GlobalExceptionHandler` (`MethodArgumentTypeMismatchException`).
- **Related tasks:** T29 (discover events, PR #29 open), T28 (publish, PR #28 open).
- **Use case:** UC-09 Discover, search and filter events:
  - "Search matches the event title or the organization name";
  - "Filters: organizing organization and date";
  - "A. No events match: the system displays an appropriate message" (an empty list);
  - "B. The user removes the criteria or filters: the results update accordingly".
- **Constraints / assumptions:**
  - **Branching:** the backend branch `T30-filter-events` is created from `T29-discover-events`, because T30 extends the same endpoint. Merge order: #28 → #29 → T30, with merge commits.
  - The T29 decisions still apply: **PUBLISHED events only**, COMPLETED not derived, sign-in required, summary fields, chronological order.
  - **Pre-existing failing tests on `master`:** 3 from T13/T14.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Extend `GET /api/events` with **optional** query parameters, combined with AND:

| Parameter | Type | Meaning |
|---|---|---|
| `search` | text | Case-insensitive **substring** match on the event title **or** the organization name. It's trimmed first, and blank means not set. |
| `organizationId` | number | Only events of this organization. An unknown id gives an empty list. |
| `from` | date `YYYY-MM-DD` | Only events on or after this date (inclusive). |
| `to` | date `YYYY-MM-DD` | Only events on or before this date (inclusive). |

- With no parameters the result is exactly T29's full list: removing the criteria restores the list (UC-09 B).
- `from` later than `to` gives **400** `BusinessRuleException("Invalid date range", [ErrorDetail("from", "From date must not be after the to date")])`.
- A wrongly typed value (`organizationId=abc`, `from=2026-13-01`) gives **400** "Request validation failed" with the field name, through the existing handler.

**Implementation:**
- **Filtering in the database:** `EventRepository` also extends `JpaSpecificationExecutor<Event>`. A small `EventSpecifications` class in Persistence builds one `Specification` per criterion (`isPublished`, `matchesSearch`, `inOrganization`, `onOrAfter`, `onOrBefore`).
- **The service** (`discoverEvents(EventDiscoveryFilter)`) validates the date range, combines only the criteria that were given, and calls `findAll(spec, Sort.by("date", "startTime"))`.
- **Wildcards:** `%` and `_` in the search text are escaped, so they're matched literally.
- **The controller** binds the 4 `@RequestParam(required = false)` values (with `@DateTimeFormat(iso = DATE)`) into a new `EventDiscoveryFilter` DTO.

**Alternatives considered:**
- One JPQL `@Query` with "`:param is null or …`": rejected, because null `LocalDate` parameters are fragile in Hibernate 6/H2 and it gets harder to read.
- Filtering in memory in the service: rejected, because loading every published event and then filtering doesn't scale, and filtering is a persistence concern.
- Derived query method names: there are too many combinations.

### Steps

1. **Model:** `dto/EventDiscoveryFilter { search, organizationId, from, to }`, a `final` class with `@Getter @AllArgsConstructor @EqualsAndHashCode`.
2. **Persistence:**
   - `EventRepository extends JpaRepository<Event, Long>, JpaSpecificationExecutor<Event>`;
   - new `EventSpecifications` with the 5 static specifications;
   - the T29 query `findAllByStatusOrderByDateAscStartTimeAsc` is no longer needed by the service, so it's removed together with its repository test. The same behavior is now covered by the specification tests.
3. **Business:** `EventService.discoverEvents()` becomes `discoverEvents(EventDiscoveryFilter filter)`. It does the range validation (400), composes the specifications, sorts with `Sort.by("date", "startTime")`, and reuses the `toSummaryResponse` mapping.
4. **API:** `DiscoverEventController.discoverEvents(...)` gets the 4 optional `@RequestParam`s, builds the filter, and has updated OpenAPI annotations (200, 400, 401).
5. **T29 tests:** the T29 tests change because the signature changes: the service is now called with an empty filter, and the behavior is unchanged.
6. **Verification:** tests first and a behavioral red, then implement and get the focused tests green. Then run the full suite with `--continue`; the only failures should be the 3 known pre-existing ones.

### Files to create / modify

Backend (`repos/ubb-events-backend/`, branch `T30-filter-events` from `T29-discover-events`):

- Create `Model/src/main/java/com/ubb/events/dto/EventDiscoveryFilter.java`
- Create `Persistence/src/main/java/com/ubb/events/persistence/EventSpecifications.java`
- Modify `Persistence/src/main/java/com/ubb/events/persistence/EventRepository.java`
- Modify `Persistence/src/test/java/com/ubb/events/persistence/EventRepositoryTest.java` (the T29 query test is replaced by specification tests)
- Modify `Business/src/main/java/com/ubb/events/EventService.java`, `Business/src/main/java/com/ubb/events/implementation/EventServiceImpl.java`
- Modify `Business/src/test/java/com/ubb/events/implementation/EventServiceTest.java`, `EventServiceStructureTest.java`
- Modify `RestServices/src/main/java/com/ubb/events/controllers/DiscoverEventController.java`
- Modify `RestServices/src/test/java/com/ubb/events/controllers/DiscoverEventControllerApiTest.java`, `DiscoverEventControllerStructureTest.java`

### Tests to write

Acceptance criteria → tests:

- **AC1: search, organization filter, date range; removing the criteria returns the full list:**
  - `EventRepositoryTest` (real H2, specifications):
    - `withoutCriteriaReturnsAllPublishedEventsChronologically`: drafts and canceled events are excluded, and the order is kept (replaces the T29 test);
    - `searchMatchesEventTitleCaseInsensitively`;
    - `searchMatchesOrganizationName`;
    - `searchTreatsWildcardsLiterally`;
    - `filtersByOrganization`;
    - `filtersByInclusiveDateRange` (the `from`/`to` boundaries are included, and each bound can be used alone);
    - `combinesAllCriteria`.
  - `EventServiceTest`:
    - `discoverWithEmptyFilterReturnsAllPublishedSummaries` (the T29 behavior);
    - `discoverPassesCriteriaAndSortsChronologically`;
    - `blankSearchIsIgnored`.
  - `DiscoverEventControllerApiTest`:
    - `bindsQueryParametersIntoFilter`: `?search=hack&organizationId=7&from=2026-11-01&to=2026-11-30` reaches the service as the matching `EventDiscoveryFilter`;
    - `withoutParametersUsesEmptyFilter`;
    - `noMatchesReturnsEmptyList`.
- **AC2: invalid input and authentication:**
  - `EventServiceTest.rejectsFromAfterTo`: `BusinessRuleException` with a field error on `from`; the repository is not queried.
  - `DiscoverEventControllerApiTest.invalidDateRangeReturnsBadRequest`.
  - `DiscoverEventControllerApiTest.malformedParameterReturnsBadRequest`: `from=not-a-date` and `organizationId=abc` give 400 "Request validation failed".
  - `DiscoverEventControllerApiTest.unauthenticatedRequestIsRejected` (kept from T29).
- **Expected red:**
  - the DTO, the `EventSpecifications` signatures and the `JpaSpecificationExecutor` exist, but the specification methods return `null`, so they add no restriction;
  - the service ignores the filter and returns an empty list;
  - the controller doesn't bind the parameters.

  The failures will be assertions, not compilation errors.

### Risks / open questions

User answers before Gate 1:

1. **Parameter names:** `search`, `organizationId`, `from`, `to`. **Confirmed by the user.**
2. **Search:** matches **both** the event title and the organization name (title OR organization name), as a case-insensitive substring, trimmed, and ignored when blank. **Confirmed by the user ("both").**
3. **Date range:** inclusive on both ends, on the event `date`. **Either bound can be used on its own (confirmed by the user).** `from` > `to` → 400. **Confirmed by the user.**
4. **Unknown `organizationId`:** an empty list (200), not 404. **Confirmed by the user.**
5. **The T29 repository query** (`findAllByStatusOrderByDateAscStartTimeAsc`): the user asked why it becomes unused. Answer: with Specifications, `discoverEvents` always calls `findAll(isPublished() + optional criteria, Sort.by("date", "startTime"))`. `isPublished()` and the sort replace exactly what the T29 query did, so nothing calls the old method and keeping it would be dead code. Keeping it as a "no filters" shortcut would mean two code paths and duplicate tests. Its behavior stays covered by `withoutCriteriaReturnsAllPublishedEventsChronologically`. **User decision: remove it.**

**Plan amendment after Gate 2 (review change, Human Corrections #2): upcoming first, then completed**

A request from a teammate (Vasile Irimieș), relayed and approved by the user: "sort by date, first the upcoming ones sorted by date, then the completed ones sorted by date". The user's decisions:
1. **Completed events are ordered most recent first** (descending). Upcoming events are ascending (the next one first).
2. **"Completed" means** `date` + `endTime` is not after now (or `date` + `startTime` when `endTime` is missing). An event that is running counts as upcoming until its end. This is the same definition T22 should use when it derives COMPLETED. The status label is unchanged (still `PUBLISHED`) until T22.
3. **It is done in T30**, as a review change on the open backend PR #30.

- **Approach:** the database query (filters plus `Sort.by("date", "startTime")`) stays as it is. `EventServiceImpl.discoverEvents` then splits the mapped result with the injected `Clock` (from T28) into upcoming (kept in ascending order) and completed (reversed, so most recent first), and returns upcoming followed by completed. This is done in Java rather than with an SQL `ORDER BY CASE`, because "ended" depends on the current time (testable through `Clock`), the list isn't paginated, and a `CASE` ordering inside Specifications would be fragile.
- **Files:** `Business/.../implementation/EventServiceImpl.java` and `Business/src/test/.../implementation/EventServiceTest.java`.
- **Tests, written first:**
  - `EventServiceTest.ordersUpcomingFirstThenCompletedMostRecentFirst`, with the fixed clock 2026-11-01 10:00 Cluj: an event running now and a future event first (ascending), then an event that ended earlier today, then past events (descending).
  - `EventServiceTest.usesStartTimeWhenEndTimeIsMissing`.
  - `EventServiceTest.eventEndingExactlyNowIsCompleted`.
- **Gate 2** needs to be confirmed again for this change before it's committed.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:**
  - `EventRepositoryTest`: 8 specification tests; they replace T29's `findsOnlyPublishedEventsOfAllOrganizationsChronologically`, whose query was removed (user decision).
  - `EventServiceTest`: 4 discovery tests; they replace T29's 2.
  - `EventServiceStructureTest` (updated signature).
  - `DiscoverEventControllerApiTest` (rewritten for the filter).
  - `DiscoverEventControllerStructureTest` (updated).
- **Stubs used:**
  - complete `EventDiscoveryFilter`;
  - `EventRepository extends JpaSpecificationExecutor<Event>`, with the T29 query removed;
  - `EventSpecifications` with all methods (`isPublished`, `matchesSearch`, `inOrganization`, `onOrAfter`, `onOrBefore`, `forDiscovery`) returning `null`, which means no restriction;
  - `EventServiceImpl.discoverEvents(EventDiscoveryFilter)` returning `List.of()` without validating or querying;
  - `DiscoverEventController.discoverEvents(search, organizationId, from, to)` without `@RequestParam` annotations, always passing an empty filter.
- **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Persistence:test --tests "com.ubb.events.persistence.EventRepositoryTest" :Business:test --tests "com.ubb.events.implementation.EventServiceTest" --tests "com.ubb.events.implementation.EventServiceStructureTest" :RestServices:test --tests "com.ubb.events.controllers.DiscoverEventControllerApiTest" --tests "com.ubb.events.controllers.DiscoverEventControllerStructureTest"`
- **Result:** `BUILD FAILED`, everything compiled, 16 of 44 tests failed.
  - `EventRepositoryTest`, 8 of 8 new failed (`AssertionFailedError`: drafts, canceled events, other organizations and out-of-range dates were all returned, and the order was wrong, because the null specifications restrict nothing):
    - `withoutCriteriaReturnsAllPublishedEventsChronologically`, `searchMatchesEventTitleCaseInsensitively`, `searchMatchesOrganizationName`, `searchTreatsWildcardsLiterally`;
    - `filtersByOrganization`, `filtersByInclusiveDateRange`, `combinesAllCriteria`, `discoveryFilterCombinesGivenCriteriaAndIgnoresBlankSearch`.
  - `EventServiceTest`, 4 of 4 new failed:
    - `discoversPublishedEventsAsSummariesSortedChronologically` (empty list);
    - `discoverReturnsEmptyListWhenNothingMatches` and `acceptsSingleDayAndOpenEndedRanges` (Mockito `verify`: the repository was never queried);
    - `rejectsFromAfterTo` (no exception).
  - `DiscoverEventControllerApiTest`: `bindsQueryParametersIntoFilter`, `noMatchesReturnsEmptyList`, `invalidDateRangeReturnsBadRequest` (the parameters were not bound into the filter).
  - `DiscoverEventControllerStructureTest.discoverHandlerIsGetWithOptionalFilterParametersAndNoOrganizationAnnotations` (no `@RequestParam`).
- **Passing at red, as expected:**
  - the existing T24/T28 tests in these classes;
  - `EventServiceStructureTest` (the signature exists);
  - in `DiscoverEventControllerApiTest`:
    - `unauthenticatedRequestIsRejected` (security);
    - `withoutParametersUsesEmptyFilterAndReturnsSummaries` and `doesNotRequireOrganizationMembership` (the stub always passes the empty filter, which is the correct value without parameters);
    - `malformedParameterReturnsBadRequest`: Spring's implicit binding of simple-typed parameters already rejects `abc`/`not-a-date` with 400.
- **Failure reason:** behavioral. The failures come from the missing planned functionality: no specifications, no filter composition, no date-range validation, no chronological sort, and no query parameter binding. There were no compilation or setup errors.
- **Deviation from the plan (recorded before implementing; reverted at review, see Human Corrections #1):** the criteria composition was first placed in `EventSpecifications.forDiscovery(filter)` (Persistence) instead of the service, so that it could be verified against the real H2 database. At review the user asked to keep it in the service, as planned.

### Green (after implementation)

- **Result:** the same focused command gives `BUILD SUCCESSFUL`, and all 44 tests in the focused classes pass (`EventRepositoryTest` 13, `EventServiceTest` 18, `EventServiceStructureTest` 4, `DiscoverEventControllerApiTest` 7, `DiscoverEventControllerStructureTest` 2).
- **Full suite of the affected repo:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives `BUILD FAILED`: 228 tests, 3 failures, 0 errors, 0 skipped. The 3 failures are the **pre-existing** T13/T14 ones (`OrganizationServiceStructureTest` ×2, `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`). There are no new failures; the T24/T28/T29 behavior still passes.

### Review change (Human Corrections #1: composition moved back to the service)

- **Change:**
  - `EventSpecifications.forDiscovery` was removed, and `EventServiceImpl.discoverySpecification(filter)` (private) now composes `isPublished()` with the given criteria, skipping a blank search and null values.
  - The repository test `discoveryFilterCombinesGivenCriteriaAndIgnoresBlankSearch` was replaced by `searchIsTrimmed`, because trimming stays inside `matchesSearch`.
  - Three service tests were added that verify the composition with `Mockito.mockStatic(EventSpecifications.class, CALLS_REAL_METHODS)`: `combinesPublishedWithEveryGivenCriterion`, `emptyFilterOnlyRequiresPublishedAndBlankSearchIsIgnored` and `openEndedRangeOnlyAddsTheGivenBound`.
- **TDD note:** this is a behavior-preserving restructuring requested at review, not new functionality. The new service tests were not run red first: while `forDiscovery` existed, the static mock also intercepted its internal calls to the same specifications, so they would have passed against either structure. This is recorded as a deviation, not as red evidence.
- **Result:** the focused command gives `BUILD SUCCESSFUL`, and all 47 tests in the focused classes pass (`EventRepositoryTest` 13, `EventServiceTest` 21, `EventServiceStructureTest` 4, `DiscoverEventControllerApiTest` 7, `DiscoverEventControllerStructureTest` 2).
- **Full suite:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives 231 tests, 3 failures, 0 errors. Only the 3 pre-existing T13/T14 failures remain.

### Review change (Human Corrections #2: upcoming first, then completed most recent first)

- **Red (tests written before changing the code; the existing purely chronological implementation served as the baseline):**
  - **Tests:** `EventServiceTest.ordersUpcomingFirstThenCompletedMostRecentFirst`, `usesStartTimeWhenEndTimeIsMissing`, `eventEndingExactlyNowIsCompleted`, with the fixed clock 2026-11-01 10:00 Europe/Bucharest.
  - **Command:** `.\gradlew.bat --no-daemon --console=plain --continue :Business:test --tests "com.ubb.events.implementation.EventServiceTest"`
  - **Result:** `BUILD FAILED`, everything compiled, and the 3 new tests failed on 24 tests completed:
    - `ordersUpcomingFirstThenCompletedMostRecentFirst`: `AssertionFailedError: expected: <[4, 5, 6, 3, 2, 1]> but was: <[1, 2, 3, 4, 5, 6]>`;
    - `usesStartTimeWhenEndTimeIsMissing`: `expected: <[2, 1]> but was: <[1, 2]>`;
    - `eventEndingExactlyNowIsCompleted`: `expected: <[2, 1]> but was: <[1, 2]>`.

    The other 21 tests passed.
  - **Failure reason:** behavioral. The service returned the plain chronological order, without splitting the results into upcoming and completed.
- **Implementation:** `EventServiceImpl.discoverEvents` keeps the database query and sort. It then splits the events with `hasEnded(event, LocalDateTime.now(clock))`, where "ended" means `date` + (`endTime`, or `startTime` if missing) is not after now. Upcoming events keep the ascending order, completed events are reversed (most recent first), and the result is upcoming followed by completed.
- **Green:** the focused command gives `BUILD SUCCESSFUL`, and all 50 tests in the focused classes pass (`EventRepositoryTest` 13, `EventServiceTest` 24, `EventServiceStructureTest` 4, `DiscoverEventControllerApiTest` 7, `DiscoverEventControllerStructureTest` 2).
- **Full suite:** `.\gradlew.bat --no-daemon --console=plain --continue clean test` gives 234 tests, 3 failures, 0 errors. Only the 3 pre-existing T13/T14 failures remain.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | implementation | The implementation moved the criteria composition into `EventSpecifications.forDiscovery(filter)` in Persistence, a deviation from the approved plan where the service combines the criteria. | The user said "keep it in the service". `forDiscovery` was removed; `EventServiceImpl` now composes `isPublished()` with the given criteria (skipping a blank search and null values) and calls `findAll(spec, sort)`. The composition is unit-tested in `EventServiceTest` with `Mockito.mockStatic(EventSpecifications.class)`, verifying which specifications are requested. | Follow the approved plan: business decisions about which criteria apply stay in the service layer. |
| 2 | implementation (review change) | The discovery list was purely chronological, so past events appeared before upcoming ones. A teammate asked: "sort by date, first the upcoming ones sorted by date, then the completed ones sorted by date". | The user approved it in T30: upcoming events first (by date ascending), then completed events **most recent first** ("1 desc"). "Completed" means the end (`date` + `endTime`, or `startTime` if missing) is not after now ("2 ok"). Implemented in the service with the injected `Clock`. | Participants should see what is coming next before past events. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

<!-- DRAFT written by the implementer. Verify and correct before using it as a study note. -->
- **Decisions:**
  - `GET /api/events` gets optional `search`, `organizationId`, `from` and `to` parameters (user decision), combined with AND. With no parameters the result is T29's full list, which covers UC-09 B ("removing the criteria updates the results").
  - `search` matches **both** the event title and the organization name (user decision): a case-insensitive substring, trimmed, and ignored when blank. `%`, `_` and `\` are escaped, so they're matched literally.
  - `from`/`to` are inclusive bounds on the event `date`, and **each can be used on its own** (user decision). `from` > `to` gives **400** "Invalid date range" with a field error on `from` (user decision). A malformed value gives 400 "Request validation failed" through the existing handler.
  - An unknown `organizationId` gives an empty list, because it's a filter, not a lookup (user decision).
  - The filtering is done in the database with **JPA Specifications** (`EventRepository extends JpaSpecificationExecutor<Event>`): one small `Specification` per criterion in `EventSpecifications` (Persistence). **The service** (`EventServiceImpl.discoverySpecification`) decides which criteria apply: it always includes `isPublished()` and adds the others only when given. It also validates the range, sorts with `Sort.by("date", "startTime")` and maps to the T29 summary DTO.
  - T29's `findAllByStatusOrderByDateAscStartTimeAsc` was **removed** (user decision), because `isPublished()` plus the sort replace it exactly. Its behavior is covered by `withoutCriteriaReturnsAllPublishedEventsChronologically`.
  - The composition was briefly moved to Persistence (`forDiscovery`) for database-level testing; the user had it moved back to the service (Human Corrections #1). It is unit-tested with `Mockito.mockStatic` on `EventSpecifications`, and each specification is tested against H2 in `EventRepositoryTest`.
- **Alternatives considered:**
  - A single JPQL query with "`:param is null or …`": rejected (fragile null date parameters, harder to read).
  - In-memory filtering: rejected (it loads everything).
  - Derived query names: too many combinations.
  - Keeping the T29 query as a no-filter shortcut: rejected by the user (two code paths).
  - 404 for an unknown organization: rejected by the user.
  - **Order (review change, Human Corrections #2):** upcoming events first (date and start time ascending), then completed events most recent first. An event counts as completed when `date` + `endTime` (or `startTime` if missing) is not after now (injected `Clock`, Europe/Bucharest). An event that is running counts as upcoming. The split is done in Java after the database query, because "ended" depends on the current time and there's no pagination. **T22 should use the same "ended" definition** when it derives COMPLETED; the status label stays `PUBLISHED` until then.
- **Limitations:**
  - The order isn't purely chronological anymore, so a client that wants a different order must sort itself. The split happens in memory, which is fine without pagination but would need to move into SQL if pagination is added.
  - The search is a plain substring `LIKE`; there is no accent folding (for example, "Cluj" vs "Clúj") and no full-text ranking.
  - There is no pagination (backlog).
  - The date filter uses the event `date` only, not the times.
  - The full suite still has the 3 pre-existing T13/T14 failures.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user (including review change #2)
- [x] **Gate 2:** approved for commit and push (confirmed again for review change #2)
- [x] Committed and pushed
- [x] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [x] Moved to `changes/completed/` (at Gate 2, by user preference; PR review and Gate 3 still pending)