# T16 - Manage organization membership requests

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T16
- **Type:** workflow
- **Owner:** Kentsch Karina
- **Requirement(s):** ORG-03, ORG-05
- **Use case(s):** UC-05
- **Workflow:** workflow
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** backend
- **Branch:** `T16-manage-organization-membership-requests`
- **PR link(s):** 
- **Tests:** `Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java`; `Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java`; `RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerStructureTest.java`

## Description

<!-- What needs to be done and why. Which user need does it address? -->

Create the backend task “Manage organization membership requests” from the backlog. This task addresses the backend scope and user need described below.

Backlog specification:

- **Type:** workflow. **Requirements:** ORG-03, ORG-05. **Depends on:** Init Membership entity, Organization-scoped authorization, Request organization membership.
- `PATCH /api/organizations/{organizationId}/membership-requests/{requestId}` with the new status (accepted or declined), for managers only; successful decisions return HTTP 200 with the resulting status (`ACTIVE` for acceptance, `DECLINED` for deletion).
- Accepting promotes the pending Membership row to `ACTIVE` with the `MEMBER` role in the same transaction as the decision.
- Declining deletes the pending Membership row.
- Only pending requests can be decided, and an already processed request shows its current status.
- No reviewer or decision-time audit record is required.

Dependencies from the backlog: 7, 9, 12.

## Acceptance criteria

<!-- Verifiable statements. Each one should be testable. -->

- [ ] The task is implemented according to the backlog specification above.
- [ ] Relevant behavior, validation, authorization, and error cases specified above are covered by tests as applicable.

<!-- Proposed from the backlog; confirm or edit during task planning. -->

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java`, `Business/src/main/java/com/ubb/events/MembershipService.java`, `Business/src/main/java/com/ubb/events/implementation/MembershipServiceImpl.java`, `Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java`, `Model/src/main/java/com/ubb/events/domain/Membership.java`, `MembershipStatus.java`, and corresponding API/service/repository tests. Current T15 listing endpoint already exposes organization membership requests and filters by status.
- **Related completed tasks:** `T7-init-membership-entity.md` establishes the Membership entity and repository; `T9-organization-scoped-authorization.md` establishes manager authorization; `T15-view-organization-membership-requests.md` implements the adjacent GET workflow and request DTO.
- **Constraints / assumptions:** Backend only; preserve the controller/service/repository layering, shared error handling, and organization-scoped authorization through the T9 interceptor. Acceptance criteria refer to the supplied backlog contract. Confirm dependencies are present in backend `master`; repository is clean on `master` and synchronized with `origin/master`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the decision workflow as a vertical backend change over the Membership row used by T12/T15 for a request. Add the organization-scoped PATCH handler to the existing `MembershipController`, require `MANAGER` authorization through the T9 interceptor, validate an `accepted` or `declined` decision, and delegate all state changes to `MembershipService`. In one transaction, an accepted pending request is promoted to an active `MEMBER`; a declined pending request is deleted. A request that is already active is not changed and is returned with its current status, while a missing or deleted request is reported through the established not-found error. Keep persistence access in `MembershipRepository`, business rules in the service, and shared error mapping in the existing handler. No reviewer or decision-time audit fields are added, per the clarified task decision.

The current model represents a request as a `Membership` row, so acceptance updates that row rather than creating a second row for the same user and organization. This preserves the existing T12/T15 data model and makes the decision atomic with the membership transition.

### Steps

1. **Model/API contract:** add a small request DTO or enum for the two wire decisions (`accepted` and `declined`) without changing persisted `MembershipStatus` (`PENDING`/`ACTIVE`). Add a small decision result DTO with `ACTIVE` for an accepted/already-active request and `DECLINED` for a declined request; the declined outcome is returned but not persisted.
2. **Persistence:** extend `MembershipRepository` with a lookup scoped by request id and organization id, and retain the existing status finder. Use `delete` for declined requests. Keep repository methods limited to data access.
3. **Business:** add a transactional service operation accepting `organizationId`, `requestId`, the decision, and the authenticated reviewer id only as an authorization-context parameter if needed by the service signature. Load the row in the requested organization; reject a missing row with the shared not-found error. For `PENDING`, set role `MEMBER` and status `ACTIVE` for acceptance, or delete the row for decline. For an already `ACTIVE` row, return its current state without applying a second decision. Do not add reviewer/time persistence or duplicate authorization checks.
4. **API:** add `PATCH /api/organizations/{organizationId}/membership-requests/{requestId}` to `MembershipController`, bind and validate the decision payload, annotate it with `@RequireOrgPermission(EffectiveOrgRole.MANAGER)`, document 200/400/401/403/404 responses, and return HTTP 200 with the decision result DTO (`ACTIVE` for accepted/already active; `DECLINED` for declined).
5. **Transaction and errors:** ensure the acceptance/decline mutation and persistence operation run in one service transaction. Preserve global handling for invalid decisions, malformed ids, missing requests, unauthenticated callers, non-managers, and requests from another organization; no persistence details should leak in error messages.
6. **Verification:** write focused repository, service, and API tests first, obtain a behavioral red result caused by the missing decision behavior, then implement and run the focused tests followed by the full backend suite. Record exact commands and outcomes in the task test log.

### Files to create / modify

- `../../repos/ubb-events-backend/Model/src/main/java/com/ubb/events/dto` — decision request DTO and, if needed, a response type for the PATCH contract.
- `../../repos/ubb-events-backend/Persistence/src/main/java/com/ubb/events/persistence/MembershipRepository.java` — organization-scoped request lookup (and any delete support required by the repository convention).
- `../../repos/ubb-events-backend/Persistence/src/test/java/com/ubb/events/persistence/MembershipRepositoryTest.java` — lookup isolation and delete/active persistence coverage.
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/MembershipService.java` — decision operation contract.
- `../../repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/MembershipServiceImpl.java` — transactional pending-request decision rules.
- `../../repos/ubb-events-backend/Business/src/test/java/com/ubb/events/implementation/MembershipServiceTest.java` and structure tests — acceptance, decline, already-active, missing, cross-organization, and invalid-state behavior.
- `../../repos/ubb-events-backend/RestServices/src/main/java/com/ubb/events/controllers/MembershipController.java` — PATCH mapping, validation, authorization, and OpenAPI responses.
- `../../repos/ubb-events-backend/RestServices/src/test/java/com/ubb/events/controllers/MembershipControllerApiTest.java` and structure tests — API decision, authorization, validation, not-found, and response-shape coverage.

### Tests to write

- **Acceptance criterion 1:** a controller API test exercises the PATCH endpoint as an authorized manager for both accepted and declined decisions; service/repository tests verify that acceptance leaves an `ACTIVE` `MEMBER` row and decline removes the pending row, with the mutation scoped to the requested organization.
- **Acceptance criterion 2:** service and API tests cover manager-only authorization (401 unauthenticated, 403 non-manager), invalid/missing decision payload (400), malformed ids (400), missing or cross-organization request (404), only-pending mutation, already-active idempotent/current-status behavior, and the shared error response shape. Repository integration tests verify the accepted state and declined deletion in the transaction boundary.
- **Edge cases:** repeated acceptance does not create a duplicate membership; a declined request no longer blocks a later request (the row is gone); a pending request in another organization cannot be decided by changing the path organization id; an already active request cannot be declined or changed.

### Risks / open questions

- The backlog did not define the PATCH response body; the user approved HTTP 200. Return a small decision result containing `ACTIVE` for acceptance or an already-processed active request, and `DECLINED` for a deleted request. `DECLINED` is a response outcome only and is not persisted.
- The backlog sentence “accepting creates the Membership” is reconciled with the current T12/T15 model by promoting the existing pending `Membership` row to `ACTIVE`/`MEMBER`; creating a second row would violate the current request representation and duplicate the membership.
- Declined requests are deleted per the user clarification, so a later lookup returns not-found rather than a persisted declined status; this supersedes the original audit/current-status wording for declined requests.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `MembershipServiceTest`, `MembershipRepositoryTest`, `MembershipControllerApiTest`, and `MembershipControllerStructureTest` were added for the approved acceptance, decline, idempotence, organization scope, response, validation, and authorization behavior.
- **Attempted command:** `gradlew.bat :Business:test --tests com.ubb.events.implementation.MembershipServiceTest :Persistence:test --tests com.ubb.events.persistence.MembershipRepositoryTest :RestServices:test --tests com.ubb.events.controllers.MembershipControllerApiTest --tests com.ubb.events.controllers.MembershipControllerStructureTest`
- **Result:** The sandbox invocation produced no output and did not complete; it was interrupted. The escalated invocation was rejected, so no test result was obtained.
- **Failure reason:** No behavioral failure was observed, so this is not valid red evidence. At the user's explicit direction, implementation proceeded without a red run; this is a deviation from the workspace TDD workflow, and the red status remains unchecked.

### Green (after implementation)

- **Result:** Not run, at the user's direction.
- **Full suite of the affected repo:** Not run, at the user's direction.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “if a membership is approved then it will be mapped to ACTIVE, otherwhise the membership request will be deleted from the Mambership table” | Plan approval as setting the membership status to `ACTIVE`; plan decline as deleting the membership request row. | Clarifies the decision outcome behavior. |
| 2 | plan | “remove that audit requirement” | Removed the reviewer and decision-time audit requirement from the task specification. | Declined requests are deleted and no audit record is required. |
| 3 | plan | “Use 200, also i approve the plan you may continue” | Set successful PATCH responses to HTTP 200 and use the decision result response described in the plan. | Records the approved response contract and Gate 1 approval. |
| 4 | implementation | “you do not have to run tests, please do the code” | Implemented the approved behavior without running the tests; recorded the missing red/green evidence. | User explicitly directed that implementation continue without test execution. |
## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions (draft for user review):** Reuse the pending Membership row for acceptance by setting status to `ACTIVE` and role to `MEMBER`; delete it on decline. Return a response-only result status (`ACTIVE` or `DECLINED`) with HTTP 200. Keep decision logic in the service and perform it transactionally; scope lookup by organization and request id.
- **Alternatives considered:** Persisting a declined status was rejected because the user specified deleting declined requests. Adding a reviewer/time audit record was removed at the user's direction.
- **Limitations:** Tests were not run at the user's direction. The required behavioral red result was not obtained, so no test pass or runtime verification is claimed.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [ ] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``












