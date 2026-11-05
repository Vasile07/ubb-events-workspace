# T56 - Routing and route guards

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T56
- **Type:** workflow
- **Owner:** Cernautan Teofan
- **Requirement(s):** N/A
- **Use case(s):** N/A
- **Workflow:** N/A
- **Drive doc / ADR:** N/A
- **Repo(s):** frontend
- **Branch:** `T56-routing-and-route-guards`
- **PR link(s):** [Frontend PR #9](https://github.com/Vasile07/ubb-events-frontend/pull/9)
- **Tests:** `cypress/e2e/routing-and-guards.cy.ts`; `cypress/e2e/foundations.cy.ts`; `cypress/e2e/home.cy.ts`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 9 — Routing and route guards

**Description:** Define the route map.
- Public: `/login`, `/register`.
- Authenticated: `/events`, `/events/:eventId`, `/my-registrations`, `/my-tasks`, `/organizations`, `/organizations/:organizationId`, `/profile`.
- Workspace (members only): `/org/:organizationId/events`, `/org/:organizationId/events/new`, `/org/:organizationId/events/:eventId/edit`, `/org/:organizationId/events/:eventId/{tasks|registrations|chat|announcements|follow-up}`, `/org/:organizationId/requests`, `/org/:organizationId/members`.

Guards:
- `RequireAuth` (redirect to `/login` remembering the target)
- `RedirectIfAuthenticated`
- `RequireOrgMember` and `RequireOrgManager` (render the 403 page when the role is insufficient; manager-only: create/edit/publish/delete event, membership requests, promote, follow-up).

Default redirect to `/events`. Placeholder pages are fine until the screens exist.
**Depends on:** 8.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/App.tsx` currently has `/components` and a wildcard fallback; `src/main.tsx` mounts `BrowserRouter`, `AuthProvider`, and `ToastProvider`. `src/features/auth/AuthProvider.tsx` exposes `status`, `isMemberOf`, and `isManagerOf`; `types.ts` defines membership roles. `src/shared/api/organizations.ts` and `types.ts` expose organization status/role responses. Cypress tests are under `cypress/e2e/`, with `auth-context.cy.ts` as the auth test pattern. `context/project-context.md` specifies React and frontend verification commands in `package.json` (`typecheck`, `lint`, `build`, Cypress).
- **Related completed tasks:** T8 Sign in (`changes/completed/T8-sign-in.md`) establishes backend JWT authentication. T48 Project setup and T49 Design tokens/global styles are completed frontend tasks. T55 Authentication context provider is merged into frontend `master` (PR #8); its workspace task file remains under `changes/todo/`.
- **Constraints / assumptions:** Frontend `master` is clean and matches `origin/master` at `6d5cf92` (merge PR #8). Backend `master` is clean and contains the T8 sign-in commit as an ancestor of `origin/master`; the login endpoint is present. The user reclassified this task as `workflow` and confirmed route guards only; manager-only publish/delete/promote action checks are outside this task while pages remain placeholders. The `/requests` and `/follow-up` routes are manager-only; the `/members` route is member-accessible, with promote controls outside scope. `AuthProvider` owns the loading UI and blocks route rendering until auth resolution; guards run only after resolution.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement routing as a small frontend authorization layer around the existing `BrowserRouter` and `AuthProvider`. Add reusable route guard components for authentication, authenticated-only redirects, organization membership, and organization manager access; use React Router location state/query information to preserve the requested destination when sending an anonymous user to `/login`. Define placeholder page components and a 403 page so every route is reachable or rejected according to the route map while later screen tasks can replace the placeholders. Organization membership and manager checks will use the existing auth context helpers and numeric `organizationId` path parameter. Action-level publish/delete/promote controls remain outside this task.

The route table will be centralized in `App.tsx` or a focused routing module rather than scattered conditional links. A route configuration abstraction is not needed yet because the app has a small, explicit map and placeholders; reusable guard components keep authorization behavior testable without coupling pages to auth logic. The route/authorization contract should be recorded in this task evidence and, if the team maintains one, the frontend architecture documentation.

### Steps

1. **UI / route primitives:** Create placeholder page components for login, register, authenticated destinations, organization views, workspace views, and a semantic 403 page. Give each page a stable heading or `data-testid` so route tests can identify it and keep the placeholders accessible.
2. **UI / authentication guards:** Keep the existing `AuthProvider` loading gate as the owner of the loading UI: while `status` is `loading`, it blocks route rendering so no protected page can flash and no guard runs against an unresolved session. Once the provider resolves, implement `RequireAuth` to allow authenticated users through or redirect anonymous users to `/login` with the original pathname/search/hash preserved in a `location.state.from` contract. Implement `RedirectIfAuthenticated` for `/login` and `/register`, redirecting authenticated users to `/events` while allowing anonymous users through. Because login is a placeholder in T56, do not implement a login success flow or consume `location.state.from`; leave consumption to the later login workflow.
3. **UI / organization guards:** Implement `RequireOrgMember` and `RequireOrgManager` using `useParams().organizationId`, explicitly validating and parsing the string parameter to a finite positive numeric organization id before calling `useAuth().isMemberOf` / `isManagerOf`. A non-numeric, non-positive, or unsafe integer id renders the 403 page without calling the helper. A path missing the organization segment does not match a workspace route and follows the unknown-path redirect. Insufficient authenticated roles also render 403; anonymous users still pass through `RequireAuth` to the login redirect. Treat manager membership as satisfying member access, and avoid making API calls or embedding business rules in the route layer.
4. **UI / route map:** Replace the wildcard-only app routing with the public, authenticated, and organization workspace paths from the description. Protect `/org/:organizationId/events`, `new`, event `edit`, tasks, registrations, chat, announcements, and members with member access; protect `/requests` and `/follow-up` with manager access. Add `/` and unknown-path handling that redirects to `/events` as specified. Preserve the existing Foundations preview at `/foundations` and `/components` so the default redirect does not remove existing design-system previews.
5. **UI / navigation contract:** Preserve the remembered destination as the documented `location.state.from` contract and verify it through the route tests; do not implement login success or consume that state while login remains a placeholder. Preserve the existing `AuthProvider` logout and expired-session behavior that lands on `/login`. Keep action-level publish/delete/promote authorization out of this task because those controls belong to later screen workflows.
6. **Verification:** Write Cypress route/guard tests before implementation and obtain behavioral red results for missing routes and guard behavior. Implement the route layer, rerun focused Cypress tests, then run `npm run typecheck`, `npm run lint`, `npm run build`, and the full Cypress suite.

### Files to create / modify

- `repos/ubb-events-frontend/src/App.tsx` (modify: route tree, default/unknown redirects, placeholder page wiring).
- `repos/ubb-events-frontend/src/app/routes.tsx` or `src/shared/routing/guards.tsx` (create if separation improves readability: reusable guard components and remembered-location logic; keep the chosen location consistent with the existing folder conventions).
- `repos/ubb-events-frontend/src/features/auth/AuthProvider.tsx` (modify only if login needs a small return-location handoff; preserve existing token/session behavior).
- `repos/ubb-events-frontend/src/features/auth/types.ts` (modify only if a typed React Router location state is introduced).
- `repos/ubb-events-frontend/cypress/e2e/routing-and-guards.cy.ts` (create: route map, redirects, role guards, 403, and remembered target coverage).
- `repos/ubb-events-frontend/cypress/e2e/foundations.cy.ts` and `home.cy.ts` (modify: keep Foundations checks on `/foundations` and verify the new root redirect).
- `repos/ubb-events-frontend/cypress/e2e/auth-context.cy.ts` (modify only if existing auth fixtures need to share route test setup).

### Tests to write

- **Acceptance criterion 1 — route map and placeholders:** Cypress tests visit each public, authenticated, and workspace path and assert the expected placeholder page; test `/` and an unknown path redirect to `/events`, and verify `/components` remains available if retained.
- Preserve the existing Foundations token preview at `/foundations`; the Foundations spec verifies that route, and the home spec verifies `/` now follows the `/events` default route.
- **Acceptance criterion 2 — dependency/auth behavior:** Cypress tests with no token verify every authenticated path redirects to `/login`; the redirect state's `from` value contains the exact original pathname, search, and hash. Tests verify authenticated users are redirected away from `/login` and `/register` to `/events`, while anonymous users can open both public routes. The test asserts the saved destination contract only; consumption after login is deferred to the later login workflow.
- **Acceptance criterion 3 — organization authorization:** Cypress tests stub the auth context/profile for member, manager, and non-member users. Member access succeeds for member workspace routes; a manager succeeds on member and manager routes; a non-member receives the accessible 403 page; a member receives 403 on `/requests` and manager-only `/follow-up`; a membership for one organization does not grant access to another organization ID in the URL. Non-numeric, non-positive, and unsafe integer IDs render 403 without calling the membership helper; a path missing the organization segment follows unknown-path behavior. Include loading-gate coverage that verifies the existing `AuthProvider` `auth-loading` UI blocks route/page rendering and prevents a redirect or protected-page flash until authentication resolves; `RequireAuth` behavior is tested only after resolution.
- Write these Cypress tests before the route implementation, run them to capture behavioral failures, then implement and rerun focused tests. Finish with `npm run typecheck`, `npm run lint`, `npm run build`, and `npm run cypress:run`.

### Risks / open questions

- The current `AuthProvider` redirects on logout and exposes membership helpers based on the loaded profile. The implementation should preserve that contract; no separate organization API query is planned for guards.
- The exact placeholder content is intentionally minimal. Later screen tasks may replace page components without changing route paths or guard semantics.
- The route description names manager-only publish/delete/promote actions but provides no action routes. Per the user's clarification, those controls and checks are excluded from T56; only the manager-protected route surfaces are covered.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/routing-and-guards.cy.ts`.
- **Result:** Initial Cypress attempts were blocked by the sandboxed Cypress cache/runtime, then by missing `VITE_API_URL`; neither was counted as red. With Vite started using `VITE_API_URL=http://127.0.0.1:8080`, `npm run cypress:run -- --spec cypress/e2e/routing-and-guards.cy.ts` exited 1 before implementation: 7 tests ran, 1 passed and 6 failed. The failing tests were `renders public placeholders and redirects the default and unknown paths` (login placeholder missing), `redirects anonymous users to login and remembers the exact target` (stayed on `/events`), `redirects authenticated users away from public auth pages` (stayed on `/login`), `allows a member into member workspace routes and blocks manager routes` (workspace page missing), `allows a manager into manager workspace routes` (manager page missing), and `blocks non-members and invalid organization ids with the 403 page` (403 page missing). `does not flash a route while the auth provider resolves` passed.
- **Failure reason:** The valid run reached the app and behavioral assertions, demonstrating the route map and guards were missing from the wildcard-only app. The loading-gate behavior already existed in `AuthProvider`. After this red run, route-matrix, invalid-ID edge-case, and Foundations-preservation assertions were added while implementation had begun; those later assertions are covered by green results but are not claimed as pre-implementation red evidence.

### Green (after implementation)

- **Result:** `npm run typecheck`, `npm run lint`, and `npm run build` passed. Focused `npm run cypress:run -- --spec cypress/e2e/routing-and-guards.cy.ts` passed all 10 tests with Vite running at `127.0.0.1:5173` and `VITE_API_URL=http://127.0.0.1:8080`.
- **Full suite of the affected repo:** `npm run cypress:run` passed all 40 tests across 8 specs. The first full-suite run exposed the old Foundations-at-root assumption; following the user's direction, the preview now lives at `/foundations`, its existing spec visits that route, and `home.cy.ts` verifies `/` redirects to `/events` for an authenticated user. Route-matrix and invalid-ID assertions added after implementation began are recorded as a TDD coverage deviation above.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “resume, the frontend should be present” | Rechecked the frontend checkout and replaced the stale empty-repository context with the current repository and route/auth details. | The frontend is present and on the clean, up-to-date `master` branch. |
| 2 | plan | T56 was classified as `deployment` despite describing React routes and guards. | Reclassified it as `workflow`; scoped authorization to route guards only. | User direction. |
| 3 | plan | “Yes, preserve it at /foundations” | Kept the Foundations preview at `/foundations` and updated the route and spec plan while `/` redirects to `/events`. | Preserve the existing preview while applying the new default route. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Implement the route table and reusable guards in the frontend app. `RequireAuth` preserves the requested pathname, search, and hash in `location.state.from`; the login placeholder records that value but does not consume it. Organization IDs are parsed as safe positive integers before membership checks. Insufficient roles and invalid IDs render an accessible 403 page. Root and unknown paths go to `/events`; the existing Foundations preview is preserved at `/foundations` and the component showcase remains at `/components`.
- **Alternatives considered:** A separate route-configuration layer was not introduced because this route map is small. The guard components remain colocated with the app routes. Action-level publish/delete/promote checks were deferred because those screens and controls are outside this task.
- **Limitations:** Route guards use the loaded membership snapshot in `AuthProvider`; backend authorization remains authoritative. Pages are placeholders. Login does not yet consume `location.state.from`; that navigation belongs to the login workflow. Route-matrix and invalid-ID cases added after implementation began are covered in green verification but were not part of the pre-implementation red run.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — `2939502` (`T56: implement routing and route guards`)
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [x] Metadata complete (traceability)
- [x] Moved to `changes/completed/`
