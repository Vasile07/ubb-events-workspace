# T68 - Organization workspace shell

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T68
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** N/A
- **Use case(s):** N/A
- **Workflow:** N/A
- **Drive doc / ADR:** N/A
- **Repo(s):** frontend, backend
- **Branch:** `T68-organization-workspace-shell`
- **PR link(s):** [Frontend PR #16](https://github.com/Vasile07/ubb-events-frontend/pull/16); [Backend PR #48](https://github.com/Vasile07/ubb-events-backend/pull/48)
- **Tests:** frontend `cypress/e2e/organization-workspace-shell.cy.ts`, `cypress/e2e/routing-and-guards.cy.ts`; backend `Model/src/test/java/com/ubb/events/domain/OrganizationTest.java`, `Business/src/test/java/com/ubb/events/implementation/OrganizationServiceTest.java`, `RestServices/src/test/java/com/ubb/events/controllers/OrganizationControllerApiTest.java`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 21 — Organization workspace shell

**Description:** Build the workspace layout used by every frame on page `Organization Screens`: the `Workspace sidebar` from page `Design System` (3:299) with the organization header (icon, name, category, role badge Manager/Member), items Events / Membership requests (with pending count badge) / Members and "Back to discover", next to a scrollable main area (48px horizontal, 40px vertical padding).
Load the organization from the route `/org/:organizationId` via `GET /api/organizations/{organizationId}` and the role from the auth context. Hide "Membership requests" for plain members.
The pending count comes from `GET /api/organizations/{organizationId}/membership-requests` and refreshes after accept/decline. The sidebar stretches to the full height of the content.
**Depends on:** T56 — Routing and route guards; T57 — App shell and top navigation.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` has authenticated routes and T56 organization route guards; `src/features/auth/AuthProvider.tsx` provides the current user and membership role helpers; `src/shared/api/organizations.ts` currently supports organization listing and membership requests but has no organization detail or pending-request-list call; `src/shared/api/types.ts` defines the existing organization response shape; `src/shared/ui/AppShell.tsx` and `src/styles/global.css` establish the app shell/layout patterns. Frontend package scripts currently expose `typecheck`, `lint`, `build`, and Cypress.
- **Related completed tasks:** T56 (Routing and route guards) and T57 (App shell and top navigation) are present in frontend `master`; T15 (View organization membership requests) and T16 (Manage organization membership requests) define the related backend request API behavior.
- **Constraints / assumptions:** Frontend `master` was clean before syncing and was fast-forwarded to `a7d440c`; implementation is on `T68-organization-workspace-shell`. Backend was clean on `T45-publish-event-announcements`; implementation is on `T68-organization-workspace-shell`. T56/T57 are merged. Figma metadata identified `Org – Events` (`15:10`) as the shared workspace frame and node `3:299` as the sidebar reference. Production Hibernate uses `ddl-auto=validate`; the approved schema change is provided as an idempotent SQL script, leaving existing category values null. Membership requests are manager-only per existing backend contract.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the organization workspace as a route-scoped layout that composes with the existing authenticated/member/manager guards. Extend the backend organization detail contract to return a persisted category, then load that detail in the workspace and derive the current member role from `AuthProvider`; a shared query for pending membership requests will provide the manager-only badge count. The shell will render the organization header and sidebar around the existing organization routes, while keeping page-specific content and accept/decline actions in their own screens. Query invalidation after a membership decision will refresh the count without duplicating API state.

Use the existing API client, TanStack Query patterns, shared feedback components, and design tokens. Add explicit loading, error, empty, and accessible navigation states. Reuse one workspace layout for every `/org/:organizationId/...` frame so sidebar height, scrolling, padding, and navigation remain consistent. Figma metadata identified `Org – Events` (`15:10`) as the target shared workspace frame; the Design System sidebar reference is node `3:299`.

### Steps

1. **API / service / persistence:** Add a category field to the Organization model and map it into `OrganizationDetailsResponse` from `OrganizationServiceImpl`. Add the necessary database schema update using the team's approved schema-change method; do not invent category values for existing organizations. Add a backend service/API-level test proving category is returned by `GET /api/organizations/{organizationId}`.
2. **Frontend API/types:** Add typed organization-detail and membership-request-list functions under `src/shared/api/organizations.ts`, using the existing `apiFetch` error behavior and backend endpoint contracts. Define only response fields needed by the header and pending count; derive the count from the returned pending list unless the confirmed API contract supplies an explicit count.
3. **Workspace data/loading:** Create a workspace query/layout component that validates the route id, loads `GET /api/organizations/{organizationId}`, and loads pending requests for managers. Render accessible loading, retryable error, not-found/forbidden, and empty states without flashing organization data from another route. Read role from `useAuth()` and ensure manager access controls the request-count query.
4. **UI/sidebar:** Build the full-height sidebar using the Design System `Workspace sidebar` reference: organization icon, name and category, Manager/Member role badge, Events, Members, manager-only Membership requests with pending count, and `Back to discover`. Use semantic navigation, active link state, visible focus styles, and responsive behavior. Keep the main area independently scrollable with 48px horizontal and 40px vertical padding.
5. **Route integration:** Wrap all existing `/org/:organizationId/...` route content in the workspace layout while preserving T56 member/manager guards and existing route destinations. Ensure members never see or query Membership requests, and managers retain access to the request route. Keep public routes and the global app shell unchanged.
6. **Refresh contract:** Centralize the membership-request query key and expose invalidation/refetch behavior for accept/decline mutations on the requests screen. After either decision succeeds, invalidate the organization request list so the sidebar badge updates; failed decisions must leave the current count intact.
7. **Verification:** Add backend service/API-level tests for category mapping and Cypress coverage for category, manager/member rendering, API calls, badge count and refresh seam, route links/active state, loading/error/empty states, and accessibility. Run backend focused tests plus the backend suite, then frontend focused Cypress, typecheck, lint, build, and full Cypress; record red and green evidence.

### Files to create / modify

- `repos/ubb-events-frontend/src/shared/api/organizations.ts` and `src/shared/api/types.ts` — organization detail, membership request response, and shared query-key/API helpers.
- `repos/ubb-events-backend/Model/src/main/java/com/ubb/events/domain/Organization.java` and `Model/src/main/java/com/ubb/events/dto/OrganizationDetailsResponse.java` — persisted category field and detail response contract.
- `repos/ubb-events-backend/Business/src/main/java/com/ubb/events/implementation/OrganizationServiceImpl.java` and relevant service/API tests — category mapping and response verification.
- `repos/ubb-events-backend/database/schema/001_add_organization_category.sql` — idempotent nullable category column addition; existing rows remain null.
- `repos/ubb-events-frontend/src/features/organizations/OrganizationWorkspace.tsx` (or equivalent feature module) — data loading and workspace layout/sidebar.
- `repos/ubb-events-frontend/src/features/organizations/membershipRequestQueries.ts` and `src/shared/api/organizations.ts` — shared pending-request query key and invalidation seam.
- `repos/ubb-events-frontend/public/ui-assets/workspace-*.svg` — local Figma sidebar icon assets with active/inactive variants.
- `repos/ubb-events-frontend/src/App.tsx` — compose the workspace layout around organization routes without changing existing guard policy.
- `repos/ubb-events-frontend/src/styles/global.css` — workspace grid, full-height sidebar, scrolling content, spacing, badges, and responsive styles using existing tokens.
- `repos/ubb-events-frontend/cypress/e2e/organization-workspace-shell.cy.ts` — focused behavior and accessibility coverage.
- The existing membership-request page/component, if present after dependency integration, only to call the shared query invalidation after successful accept/decline; do not move its business rules into the shell.

### Tests to write

- **Description/API loading:** stub organization detail and request-list calls and verify the organization name/category/role, manager/member variants, request count, and exact organization-scoped URLs.
- **Backend category contract:** service and API tests verify the category is mapped from the organization and serialized in `GET /api/organizations/{organizationId}`.
- **Sidebar/navigation:** verify Events, Members, Back to discover, manager-only Membership requests, active route state, link destinations, full-height layout, and the required main-area padding/scroll behavior.
- **Role/access rule:** verify a plain member has no Membership requests link, no pending-count request, and cannot expose manager-only navigation; verify a manager sees the link and count.
- **States/accessibility:** verify loading skeleton/status, organization-load error with retry, request-count failure fallback, empty request list as count zero, semantic navigation landmarks, accessible labels, keyboard focus, and visible focus behavior.
- **Refresh behavior:** after an accept and after a decline, verify the request-list query is invalidated/refetched and the sidebar count changes; verify failed mutations leave the prior count intact.
- **Dependency/route regression:** visit representative nested organization routes and confirm the existing T56 member/manager guards still decide access and non-organization routes remain unchanged.

### Risks / open questions

- Figma metadata identified `Org – Events` (`15:10`) as the shared workspace frame; `Design System` node `3:299` contains the three sidebar active states.
- The backend request endpoint returns a list of `MembershipRequestResponse`; the nullable category field is included in organization details. Existing database rows intentionally remain null until populated by the team.
- The existing route set uses separate flat routes rather than a nested React Router outlet. The implementation should use a shared wrapper/helper around those elements unless a small route refactor is required to prevent duplicated layout logic.
- Accept/decline actions may belong to a later or already merged screen. If no mutation component exists on the branch, implement the shared query key/invalidation hook and cover it with a focused integration seam rather than inventing request-management UI.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/organization-workspace-shell.cy.ts` and backend model/service/API category tests.
- **Result:** `npx cypress run --browser electron --spec cypress/e2e/organization-workspace-shell.cy.ts --reporter spec` with Vite running using `VITE_API_URL=http://localhost:8080` — 0 passing, 3 failing.
- **Failure reason:** Each test reached the existing organization routes but timed out because the planned `organization-workspace`, `workspace-loading`, and sidebar elements did not exist before implementation. This is a behavioral failure caused by the missing workspace shell, not a compile or setup failure.
- **Backend red deviation:** The initial category test attempt first failed at compilation because `Organization.getCategory()` did not exist. After correcting the test setup and before backend implementation, the model test produced a behavioral `NoSuchFieldException` for the missing `category` field. A clean pre-implementation red across all backend modules was not captured.

### Green (after implementation)

- **Result:** Frontend `npx cypress run --browser electron --spec cypress/e2e/organization-workspace-shell.cy.ts --reporter spec` — 6 passing, 0 failing, including local asset load and intrinsic SVG geometry checks. `npm run typecheck`, `npm run lint`, and `npm run build` passed. Backend `gradlew.bat :Model:test --tests com.ubb.events.domain.OrganizationTest :Business:test --tests com.ubb.events.implementation.OrganizationServiceTest :RestServices:test --tests com.ubb.events.controllers.OrganizationControllerApiTest` — BUILD SUCCESSFUL, 17 selected tests.
- **Full suite of the affected repo:** Full frontend `npx cypress run --browser edge --reporter spec` ran 16 specs: 89 passing, 2 failing. Both failures are in `organizations-list.cy.ts` at lines 41 and 60, where Cypress expected hidden `<span>` elements to be visible. T68 spec passed 6/6 and routing passed 10/10. Backend `gradlew.bat test --continue` ran 146 Business tests and 180 RestServices tests; Business failed `OrganizationServiceStructureTest.implementationIsAnnotatedAndImplementsServiceWithoutOperations` and `OrganizationServiceStructureTest.serviceInterfaceIsEmpty`; RestServices failed `AnnouncementControllerStructureTest.controllerDefinesRootMappingAndServiceDependencyWithoutHandlers` and `OrganizationAuthorizationInterceptorTest.organizationScopedProductionHandlersDeclareAuthorization`. The focused category tests pass; the backend suite is not green.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “change on backend and add what you need in response body and the category. you are allowed to make changes there to make the category be returned” | Expanded T68 from frontend-only to frontend and backend, with a persisted organization category mapped into the organization details response. | The current backend response and Organization entity do not contain category, so the frontend cannot display it from the specified API. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Use a shared route-scoped `OrganizationWorkspace` around all existing organization pages, keeping T56 guards and AppShell ownership. Load organization details and manager-only pending requests with TanStack Query; derive the badge from the returned list and expose a shared query key for later accept/decline invalidation. Add nullable persisted `category` to the backend entity and organization details response, plus an idempotent SQL `ALTER TABLE` script that leaves existing rows null. Use local Figma sidebar SVGs, including active/inactive variants, at their intrinsic geometry.
- **Alternatives considered:** A nested router outlet was not introduced because the existing route map is flat; a wrapper component keeps the change focused. Membership request actions remain in their own screen; the shell owns only the shared query contract.
- **Limitations:** Existing organization rows remain null for category by design. The current branch has placeholder organization pages, so shell integration is verified around placeholders. The broader frontend Cypress run has two organization-list visibility failures. Backend full suite has four failing structure checks across Business and RestServices. The shared pending-count invalidation seam is covered, but no accept/decline UI was added because that screen is outside this task.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** revised plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — frontend `d59281b`; backend `c679e5a`
- [x] PR created — [Frontend #16](https://github.com/Vasile07/ubb-events-frontend/pull/16); [Backend #48](https://github.com/Vasile07/ubb-events-backend/pull/48)
- [x] PR reviewed — frontend PR #16 and backend PR #48 approved
- [x] **Gate 3:** merged by user — frontend PR #16 and backend PR #48 merged on 2026-11-14
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`


