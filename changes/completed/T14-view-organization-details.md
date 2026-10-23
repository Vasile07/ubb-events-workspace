# T14 - View organization details

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T14
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):** ORG-06
- **Use case(s):** UC-04
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T14-view-organization-details`
- **PR link(s):** 
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/OrganizationServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerApiTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “View organization details” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-06. **Depends on:** Init Organization entity, Init Membership entity, Request organization membership.
- `GET /api/organizations/{organizationId}` for any signed-in user, marked `@NoOrgCheck`: the public details of one organization and the user's status (MEMBER, PENDING or NONE), so the user can decide before requesting.
- No members, requests or management data.

Dependencies from the backlog: 6, 7, 12.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/OrganizationController.java` currently exposes only the organization list; `Business/src/main/java/com/ubb/events/OrganizationService.java` and `implementation/OrganizationServiceImpl.java` provide that workflow, with `OrganizationRepository` and `MembershipRepository` for reads. `OrganizationResponse` is the existing list DTO; T14 must use a details-specific response (or an equivalent projection) containing public fields and status only, without role or management data. `OrganizationAuthorizationInterceptor` permits `@NoOrgCheck` endpoints without organization membership authorization.
- **Related completed tasks:** `T13-view-organizations.md` implements the adjacent list endpoint and documents the public response shape and membership status mapping. `T6-init-organization-entity.md`, `T7-init-membership-entity.md`, and `T9-organization-scoped-authorization.md` are referenced by T13 as relevant entity and authorization context.
- **Constraints / assumptions:** Backend is Java 21 / Spring Boot with Gradle wrapper; use the established JWT/security setup and add an API-level test per `../../context/project-context.md`. T14 specifies a details endpoint for any signed-in user, with no private membership/request/management data and no role field in the details response. The user resolved the dependency mismatch by selecting the current `Membership` model as the source of MEMBER/PENDING/NONE status; do not introduce or depend on a separate `MembershipRequest` entity for this task.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the organization-details read as a small vertical backend workflow with a details-specific response containing only `organizationId`, `name`, `description`, and the computed membership status. The controller will expose `GET /api/organizations/{organizationId}`, require authentication, mark the method with `@NoOrgCheck`, and delegate identity and organization lookup to the service. The service will load one organization and the signed-in user's `Membership` for that organization, mapping `ACTIVE` to `MEMBER`, `PENDING` to `PENDING`, and absent, declined, or otherwise non-active/non-pending membership to `NONE`. No role, member, request, logo, or other management data is returned. A missing organization is handled as the existing not-found error through global error handling.

The selected status source is the current `Membership.status` value, so this task will not introduce or depend on a separate `MembershipRequest` entity. Listing behavior remains owned by T13, and no role, member, request, management, pagination, or organization-scoped permission data is added.

### Steps

1. **Persistence:** verify and use `OrganizationRepository.findById` and `MembershipRepository.findByUserIdAndOrganizationId`; add only a narrowly scoped query if the existing methods cannot support the details read. Keep repositories limited to persistence access.
2. **Business/service:** add a details operation to `OrganizationService` and implement it in `OrganizationServiceImpl`. Load the requested organization, raise the project-standard not-found exception when absent, and combine it with the authenticated user's membership. Map membership status only: no membership or declined/null state becomes `NONE`, active membership becomes `MEMBER`, and pending membership becomes `PENDING`. Do not carry T13's role field into the details response.
3. **API/controller/security:** add `GET /api/organizations/{organizationId}` to `OrganizationController`, extract the authenticated user id using the established security setup, annotate the handler with `@NoOrgCheck`, and document success, authentication, and not-found responses with OpenAPI annotations. Return the details-specific public response; do not expose role, logo, members, requests, or management fields.
4. **Verification:** write focused service and API tests first, run them to obtain a behavioral red result, then implement and rerun for green. Finish with the affected backend test suite and record exact commands and outcomes in this task's Test log.

### Files to create / modify

- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/OrganizationService.java` (modify: add organization-details operation).
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/OrganizationServiceImpl.java` (modify: single-organization lookup and Membership status mapping).
- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto/OrganizationDetailsResponse.java` (create, or use an equivalent details projection if the established DTO conventions require it: public fields plus status only).
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/OrganizationController.java` (modify: authenticated `GET /api/organizations/{organizationId}` handler, `@NoOrgCheck`, and OpenAPI responses).
- `../../repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/OrganizationServiceTest.java` (modify or extend: details status mapping, not-found, and membership edge cases).
- `../../repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerStructureTest.java` (modify or extend: details handler and authorization annotation).
- `../../repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerApiTest.java` (modify or extend: authenticated response, unauthenticated rejection, not-found/error mapping, and private-field exclusion).

### Tests to write

- **Acceptance criterion 1 — backlog workflow:** an API-level test for authenticated `GET /api/organizations/{organizationId}` returns the requested organization's `organizationId`, `name`, `description`, and computed status, and verifies that role, member, request, logo, and management fields are absent.
- **Acceptance criterion 2 — behavior, validation, authorization, and errors:** tests verify an unauthenticated request is rejected by the real JWT security setup, an unknown organization returns the standard 404 error response, and the endpoint is annotated/configured with `@NoOrgCheck` so organization membership authorization is not required.
- **Membership status source and mapping:** service tests cover no membership and declined/null membership as `NONE`, active membership as `MEMBER`, and pending membership as `PENDING`, using the current `Membership.status` value. Include a test that the details DTO contains no role or other persistence/private fields.
- Write all tests before implementation and require a behavioral failure caused by the missing details operation; then implement, rerun focused tests, and run the wider affected-repo suite.

### Risks / open questions

- The user resolved the dependency mismatch by selecting the current `Membership` model as the status source. If later work introduces `MembershipRequest`, this endpoint should be revisited deliberately rather than silently changing its contract.
- Confirm the existing global not-found exception and API test fixture names while implementing; use those established mechanisms instead of adding endpoint-specific error handling.
- `MembershipRepository.findByUserIdAndOrganizationId` is optional and may return a single historical/declined record; the service should apply the existing T13 status mapping consistently and document any schema limitation discovered during implementation.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `Business/src/test/java/com/ubb/events/implementation/OrganizationServiceTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerStructureTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerApiTest.java`
- **Result:** Tests did not execute. With JDK 21 selected explicitly, the focused command `$env:JAVA_HOME = 'C:\Program Files\Java\jdk-21.0.11'; .\gradlew.bat :Business:test --tests "com.ubb.events.implementation.OrganizationServiceTest" :RestServices:test --tests "com.ubb.events.controllers.OrganizationControllerStructureTest" --tests "com.ubb.events.controllers.OrganizationControllerApiTest" --no-daemon --console=plain` first failed in the sandbox with `FileNotFoundException ... gradle-9.8.0-bin.zip.lck (Access is denied)`. Retrying that command with elevated access reached Gradle but failed before task execution with `java.io.IOException: Unable to establish loopback connection`. Retrying with `-Dorg.gradle.jvmargs=` and then with `JAVA_TOOL_OPTIONS=-Djava.net.preferIPv4Stack=true` produced the same loopback error. Explicit JDK 21 `java.exe -version` succeeded and reported `21.0.11`.
- **Failure reason:** These are cache/Gradle daemon environment failures before test execution, not behavioral failures. No valid red result was obtained, so `Tests written and confirmed failing (red)` remains unchecked. At the user's explicit request, the service behavior was implemented despite the blocked red run; this is a TDD deviation. The tests have not been rerun after implementation.

### Green (after implementation)

- **Result:** The user reports manually running the T14 tests and seeing them pass. The command and output were not provided to the agent, so this result is user-reported and not independently verified.
- **Full suite of the affected repo:** Not confirmed; the user did not specify whether the full backend suite was included.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | implementation | The service method is still a stub, and there are no repository changes; it should call a repository method that retrieves the required database data. | Implemented the method using `OrganizationRepository.findById` and `MembershipRepository.findByUserIdAndOrganizationId`; no repository source change was needed because both reads already exist. | The user asked for the service implementation, and the existing repository methods retrieve the required organization and membership data. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** (Draft) `getOrganizationDetails` reads the organization by id and the signed-in user's membership through the existing repositories. It returns only public organization fields and a status computed from membership status. It does not expose a role or add a redundant repository query.
- **Alternatives considered:** 
- **Limitations:** (Draft) Focused tests and the backend suite have not run. Gradle fails before test execution with `Unable to establish loopback connection`; the required pre-implementation behavioral red was not obtained, and implementation proceeded at the user's explicit request as a TDD deviation.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green) — user-reported; command/output not provided
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — `2d7f0fd` (`T14: Add organization details endpoint`), branch `T14-view-organization-details`
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``
