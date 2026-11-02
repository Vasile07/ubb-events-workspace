# T50 - API client and error handling

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T50
- **Type:** deployment
- **Owner:** Irimies Vasile
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T50-api-client-and-error-handling`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/3
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/api-client.cy.ts`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 3 — API client and error handling

**Description:** Build the HTTP layer in `shared/api` and a small `shared/lib/tokenStorage` module (`getToken`, `setToken`, `clearToken`, backed by localStorage).
The API client:
- prefixes the base URL, reads the token from `tokenStorage` and adds the `Authorization: Bearer` header, serializes JSON
- parses the backend's shared error format (status, message, list of errors) into a typed `ApiError` exposing `status`, `message` and `fieldErrors` (map field path → message). Field paths must support nested/indexed paths such as `tasks[1].description` so forms can show errors on the exact row
- exposes `setUnauthorizedHandler(handler)`. The client calls the handler on any 401, except for requests to the login and register endpoints (wrong credentials must not trigger a logout). The default handler is a no-op: the client must NOT import anything from `features/auth` or the router. The auth layer registers the handler in task 8, so the dependency goes 8 → 3 only and there is no circular dependency.

Generate or hand-write TypeScript DTO types from the OpenAPI spec for every endpoint used in the app.
Configure TanStack Query defaults (no retry on 4xx, 1 retry on network errors, sensible stale time).
Add tests for error parsing (including nested field paths) and for the 401 handler behavior.
Done when `GET /api/organizations` is called through the client, a forced 400 shows up as a typed `ApiError` and a forced 401 calls a test handler.
**Depends on:** T48 — Project setup.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/src/shared/api/client.ts` currently wraps `fetch` with the configured API URL and throws a generic `Error`; `src/shared/lib/env.ts` provides `VITE_API_URL`; `src/main.tsx` creates a default TanStack Query client. The frontend currently has no token storage or API DTO modules. Backend `RestServices/src/main/java/com/ubb/events/controllers/OrganizationController.java` exposes `GET /api/organizations`; the response DTO is `Model/src/main/java/com/ubb/events/dto/OrganizationResponse.java`. Backend error shape and safe field details are specified in `repos/ubb-events-backend/docs/api-error-handling.md`.
- **Related completed tasks:** `T3-api-conventions-and-error-handling.md` defines the shared backend error format; `T13-view-organizations.md` implements the organizations listing endpoint.
- **Constraints / assumptions:** T48 project setup (task dependency 1) is merged to frontend `master`; T49 is also merged. T50 starts from clean `master` at `db11590`. The base URL already removes a trailing slash. Backend errors use `{ status, message, errors: [{ field, message }] }`; nested field paths must remain intact. No frontend API consumers exist yet; this task's explicit integration target is `GET /api/organizations`.

## Plan


<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement a small dependency-free HTTP boundary in `shared/api` backed by `tokenStorage`, keeping authentication and routing out of the client. The boundary will normalize successful JSON responses and the backend `{ status, message, errors }` contract into typed results and `ApiError`, retain nested field paths verbatim, and invoke a replaceable unauthorized callback only for protected requests. Configure the existing TanStack Query provider with explicit retry and stale-time defaults, and add DTO types only for endpoints needed by T50, currently `GET /api/organizations`; later tasks add their own DTOs when those endpoints are introduced. Keep login/register exclusion based on the request path so incorrect credentials never trigger the logout callback.

### Steps

1. **Dependency and contract check:** with dependency 1 (T48 project setup) confirmed merged, inspect the current frontend package/test setup plus the backend organizations response and error contract before implementation; record any OpenAPI fields needed by the app.
2. **Shared storage:** add `shared/lib/tokenStorage` with `getToken`, `setToken`, and `clearToken`, using `localStorage` and a small, testable interface around browser storage where the existing test environment requires it.
3. **API boundary:** replace the generic `apiFetch` behavior in `shared/api` with a typed client that joins the configured base URL and path, attaches `Authorization: Bearer <token>` when a token exists, serializes JSON request bodies and headers, parses JSON responses, and converts non-2xx responses to `ApiError` with `status`, `message`, and a `fieldErrors` map. Preserve field paths exactly, including indexed paths such as `tasks[1].description`.
4. **Unauthorized handling:** expose `setUnauthorizedHandler`, default it to a no-op, and invoke the current handler for 401 responses except login and register paths. Keep the client independent from `features/auth` and the router so the auth layer can register the handler later.
5. **DTOs and query defaults:** add or hand-write the TypeScript DTO type needed now for `GET /api/organizations`; later tasks add DTOs for their endpoints, and configure the `QueryClient` in `main.tsx` with no retries for 4xx errors, one retry for network errors, and a documented sensible stale time.
6. **Verification:** add focused tests first, obtain a behavioral red result, then implement and run the focused tests plus the affected frontend checks. Exercise a successful organizations request, forced 400 parsing, nested/indexed field errors, protected 401 callback invocation, login/register 401 exclusion, token/header/body behavior, and query retry defaults.

### Files to create / modify

- `repos/ubb-events-frontend/src/shared/api/client.ts` (modify the client and typed error handling).
- `repos/ubb-events-frontend/src/shared/api/types.ts` or endpoint-specific DTO files (create the OpenAPI-aligned DTOs used by the app).
- `repos/ubb-events-frontend/src/shared/lib/tokenStorage.ts` (create token storage helpers).
- `repos/ubb-events-frontend/src/main.tsx` (configure TanStack Query defaults).
- `repos/ubb-events-frontend/src/shared/api/*.test.ts`, `src/shared/lib/*.test.ts`, and/or the existing frontend test location (create focused unit tests following the repository's established runner).
- `changes/active/T50-api-client-and-error-handling.md` (record test evidence, decisions, and final metadata during implementation).

### Tests to write

- A client success test proving `GET /api/organizations` uses the base URL and maps the JSON response to the typed organizations DTO.
- Request construction tests proving a stored token creates the Bearer header, JSON bodies are serialized, and no Authorization header is sent when no token exists.
- Error parsing tests proving a forced 400 becomes `ApiError` with backend `status` and `message`, and maps flat plus nested/indexed details such as `tasks[1].description` into `fieldErrors`.
- Unauthorized handling tests proving a protected 401 invokes a registered test handler, the default handler is safe, and login/register 401 responses do not invoke it.
- Query configuration tests (or a directly inspectable configuration assertion) proving 4xx errors are not retried, network errors retry once, and the stale time is set to the chosen value.
- Map each acceptance criterion to at least one of the above tests and record the exact red and green commands/results in the task's Test log.

### Risks / open questions

- The task is marked `deployment` although its description is a frontend code change; retain the existing metadata and implement the approved client, storage, DTO, and query configuration.
- The current frontend has no auth feature or router integration point for handler registration; tests should use the exported setter and verify that no auth/router import is introduced. Task 8 will register the real handler later.
- DTO scope is limited to endpoints needed by this task: currently `GET /api/organizations`. Each later task adds DTOs for the endpoints it introduces.
- Browser `localStorage` behavior in the test runner may require a small injectable storage seam; keep production behavior backed by `localStorage` and avoid adding a new state-management abstraction.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/api-client.cy.ts` (token persistence, GET organizations with Bearer auth, JSON request serialization, typed validation error mapping including `tasks[1].description`, protected 401 callback plus login/register exclusions, and TanStack Query retry/stale-time defaults).
- **Result:** `npm run cypress:run -- --spec cypress/e2e/api-client.cy.ts --reporter spec` failed behaviorally: 0 passing, 6 failing. Failing tests: stores and clears the bearer token (stored value remained null); calls GET /api/organizations through the configured base URL with a Bearer token (Authorization header was absent); serializes JSON request bodies and sends JSON content type (body remained an object); converts backend validation errors to ApiError and preserves nested indexed paths (a generic Error was thrown); calls the unauthorized handler for protected 401s but not login or register (handler was never called); configures query retries for client and network errors with a sensible stale time (staleTime was undefined). The run used the Vite server at `http://127.0.0.1:5173` and completed with exit code 6.
- **Failure reason:** Each failure demonstrates missing T50 behavior in the existing stubs/client. The failures are assertions on observed browser storage, fetch options, error type, callback invocation, and query defaults; no compile or test-setup failures occurred.

### Green (after implementation)

- **Result:** `npm run cypress:run -- --spec cypress/e2e/api-client.cy.ts --reporter spec` passed (7 tests). `npm run typecheck`, `npm run lint`, and `npm run build` also passed. After the initial red run, the typed `getOrganizations` wrapper and the no-token/default-no-op assertions were added; these additions passed in the final suite but were not included in the original pre-implementation red run.
- **Full suite of the affected repo:** `npm run cypress:run -- --reporter spec` passed all 16 tests across `api-client.cy.ts`, `foundations.cy.ts`, and `home.cy.ts`.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “I don't think we need to do that now.” | Limited T50 to the organizations DTO; each task adds the DTO it needs when its endpoint is introduced. | The user clarified that DTOs should be created task by task. |
|   |                               |                |                              |     |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Keep API requests in a shared `apiFetch` boundary with a typed `ApiError`; preserve backend field paths as object keys; use localStorage through three small token helpers; make the unauthorized handler a no-op until auth registers one; isolate the organizations response DTO in shared API types; and use a one-minute query stale time with one retry for non-4xx failures.
- **Alternatives considered:** Reused Cypress, the frontend repository test runner, instead of adding a separate unit-test dependency. Hand-wrote the single needed organization DTO from the backend response contract rather than generating unrelated DTOs.
- **Limitations:** The typed `getOrganizations` wrapper and the no-token/default-no-op assertions were added after the initial six-test red run. Those additions pass in the final suite, but they lack pre-implementation red evidence; the Red checklist remains unchecked to record this TDD deviation. Cypress stubs `fetch`, so the endpoint path, request construction, and response parsing are verified without a live backend integration run.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red) — initial six tests had valid behavioral red; later additions are documented as a TDD deviation.
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed — 2f4c8bf on T50-api-client-and-error-handling
- [x] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to `changes/completed/`


