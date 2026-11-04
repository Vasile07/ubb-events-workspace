# T55 - Authentication context provider

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T55
- **Type:** workflow
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** UC-01, UC-02, UC-03
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T55-authentication-context-provider`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/8
- **Tests:** `cypress/e2e/auth-context.cy.ts`, `cypress/e2e/api-client.cy.ts`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 8 — Authentication context provider

**Description:** Create `AuthProvider` and `useAuth()` exposing:
- `user` (id, firstName, lastName, derived `fullName`, email, memberships with `organizationId`, organization name and role)
- `token` (kept in `shared/lib/tokenStorage`) and `status` (loading | authenticated | anonymous)
- `login(email, password)` → `POST /api/auth/login`
- `register(payload)` → `POST /api/auth/register` with `firstName`, `lastName`, `email`, `password`
- `logout()` (client-side only: clear the token, clear the TanStack Query cache, redirect to `/login`)
- `refreshUser()` → `GET /api/users/me`
- helpers `isMemberOf(organizationId)` and `isManagerOf(organizationId)`.

Restore the session on startup by calling `/users/me` when a token exists; show a full-page loader meanwhile.
Register the API client's unauthorized handler here (`setUnauthorizedHandler` from task 3) so an expired token logs the user out and shows a toast. The auth layer depends on the API client, never the other way around.
Add tests for restore-on-reload, login failure (no logout loop) and logout.
**Depends on:** T48 — Project setup; T50 — API client and error handling; T53 — Feedback components: toast, empty state, skeleton, error state.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/shared/api/client.ts` exports `apiFetch`, `ApiError`, and `setUnauthorizedHandler`; `src/shared/lib/tokenStorage.ts` stores the token; `src/shared/lib/queryClient.ts` exports the TanStack Query client factory; `src/shared/ui/FeedbackComponents.tsx` provides `useToast`; `src/main.tsx` and `src/App.tsx` define app setup and routing integration points.
- **Related completed tasks:** `changes/completed/T48-project-setup.md`, `T50-api-client-and-error-handling.md`, and `T53-feedback-components-toast-empty-state.md`; T48, T50, and T53 are merged on frontend `master` (currently at `fa9c4d8`).
- **Constraints / assumptions:** Auth must depend on the API client, while the API client remains independent of auth/router; logout clears token and query cache and redirects to `/login`; startup restore calls `/api/users/me` only when a token exists. The user confirmed this application-level authentication feature should be classified as `workflow`. The current backend profile response contains membership `organizationId` and `role` but no organization name, so the source of the organization name must be confirmed before implementation.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement a typed authentication context in a new `features/auth` module, using the existing shared API client and token storage as the transport and persistence boundaries. The provider will own session state and expose auth actions, install the API client's unauthorized callback, and render a full-page loading state while an existing token is restored. Mount it beneath the existing query and router providers, with toast access available for expired-session feedback. Route guards remain in T56.

### Steps

1. **API contract and DTOs (API/shared):** confirm login, register, and `/api/users/me` response shapes against the backend contract; add frontend DTOs and the normalized `AuthUser` model, including derived `fullName` and membership helper data. Expose membership `organizationId` and `role`; defer organization names until a source is defined.
2. **Auth provider state (service/UI):** add `AuthProvider` and `useAuth()` with `loading`, `authenticated`, and `anonymous` states; restore `/api/users/me` once on startup only when a token exists; normalize profile data and expose `refreshUser`, `isMemberOf`, and `isManagerOf`.
3. **Auth actions (API/service):** implement login and register request wrappers, persist the login token, refresh the profile after login, and preserve login failures as ordinary errors without invoking logout or causing a retry/logout loop. Registration returns the created user without automatically logging in. Implement client-side logout by clearing the token, clearing the shared TanStack Query cache, clearing user state, and navigating to `/login`.
4. **Unauthorized integration (service/UI):** register `setUnauthorizedHandler` from the provider; expired protected requests call logout and show one error toast, while login/register 401 responses remain handled as login errors by the existing API client exclusion.
5. **Application wiring and loading UI (UI):** mount the provider in `main.tsx` with the existing `QueryClientProvider` and `BrowserRouter`, ensure `ToastProvider` is available to auth, and render an accessible full-page loader during startup restoration. Keep route protection and login/register screens for T56/T59/T60.
6. **Verification (UI/API):** write Cypress tests first for restore-on-reload, anonymous startup without a token, login success and failure without logout loops, registration payload, logout side effects/redirect, membership helpers, unauthorized logout toast, and loading UI; capture a behavioral red run, then implement and run focused plus typecheck/lint/build and the wider Cypress suite.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/auth/AuthProvider.tsx` (provider, hook, state and actions).
- `repos/ubb-events-frontend/src/features/auth/types.ts` and/or `repos/ubb-events-frontend/src/shared/api/auth.ts` (auth DTOs and normalized user types).
- `repos/ubb-events-frontend/src/shared/ui/FeedbackComponents.tsx` or a focused loader component (reuse existing feedback primitives for startup loading/toast).
- `repos/ubb-events-frontend/src/main.tsx` (provider wiring).
- `repos/ubb-events-frontend/src/App.tsx` (only if provider or loading fallback needs route integration).
- `repos/ubb-events-frontend/cypress/e2e/auth-context.cy.ts` (focused behavior coverage).

### Tests to write

- **Acceptance criterion 1:** auth-context Cypress coverage verifies the context shape, login/register methods, normalized user/full name, token persistence, refresh, membership helpers, logout cache clearing and `/login` redirect.
- **Acceptance criterion 2:** record T48, T50 and T53 as merged before implementation; focused tests exercise the existing API client and toast provider.
- **Acceptance criterion 3:** restore tests cover token present with successful `/api/users/me`, token present with failed/expired restore, no-token startup, full-page loading while pending, login 401 without logout recursion, expired protected 401 invoking logout and toast, and accessible status transitions.

### Risks / open questions

- Backend `MembershipResponse` currently has only `organizationId` and `role`. Per the user's decision, defer organization names until a source is defined.
- Registration returns `UserResponse` without a token. Per the user's decision, `register()` returns the created user without automatically authenticating.
- The task says `/users/me` in prose, but the concrete backend route is `/api/users/me`; use the concrete path unless corrected.
- `logout()` needs navigation access; use React Router navigation inside the provider and keep route guards in T56.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/auth-context.cy.ts` was written before implementation.
- **Result:** Before implementation, the focused Cypress command could not reach `http://127.0.0.1:5173`; no valid behavioral red run was observed. The first post-implementation full-suite run had 28 passing and 2 failing tests. After the Cypress setup corrections, the user reran the suite and reported all 30 tests passing.
- **Failure reason:** The pre-implementation run was blocked by server reachability. The later failures were not valid red evidence because they occurred after implementation: the API client test expected a hard-coded base URL that differed from configured `VITE_API_URL`, and the auth restore test set its token and fetch stub after the app had loaded. Both test setup issues were corrected.

### Green (after implementation)

- **Result:** `npm run typecheck` passed after correcting the Cypress test setup. The user reran `npm run cypress:run`; all 30 tests across 7 specs passed. `npm run build` and `npm run lint` passed earlier, before the test setup corrections.
- **Full suite of the affected repo:** User-reported `npm run cypress:run`: 7 specs passed, 30 tests passed, 0 failed.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | Membership organization names are unavailable from the current profile response. | Defer organization names; retain membership organization IDs and roles. | User chose to defer names until a source is defined. |
| 2 | plan | Registration has no token response, so automatic authentication was undecided. | `register()` returns the created user without logging in automatically. | User chose created-user-only behavior. |
| 3 | implementation | No valid behavioral red result was observed before implementation. | User instructed us to ignore the missing red result and proceed with commit, push, and PR creation. | User requested moving ahead. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Auth state is owned by `AuthProvider`; transport stays in `shared/api/auth.ts` over the existing API client. Startup restore runs only when a token exists. Logout clears token and TanStack Query state, then navigates to `/login`. Memberships currently expose `organizationId` and `role`; organization names are deferred. Registration returns the created user without automatic sign-in.
- **Alternatives considered:** Keeping auth requests directly in the provider was rejected to preserve the API/client layering. Route guards remain deferred to T56 so this provider does not own route policy.
- **Limitations:** The current backend profile contract does not provide organization names. No valid behavioral red result was observed before implementation because Cypress could not reach the dev server at that time; the later full Cypress run passed after test setup corrections.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed (`26f91d6`)
- [x] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`





