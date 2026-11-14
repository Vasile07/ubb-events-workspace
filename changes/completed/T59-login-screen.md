# T59 - Login screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T59
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** 
- **Use case(s):** UC-02
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T59-login-screen`
- **PR link(s):** Frontend: https://github.com/Vasile07/ubb-events-frontend/pull/12
- **Tests:** `cypress/e2e/login-screen.cy.ts` (new); regression: `routing-and-guards.cy.ts`, `auth-context.cy.ts` unchanged

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 12 — Login screen

**Description:** Implement page `Student Screens`, frame `Login` (13:10) using `AuthLayout`: email, password, Sign in, link to Create account (`/register`, a plain route link).
Client validation: both fields required, valid email format. The password is only checked as required: the complexity rules apply to registration only (do not block existing users or reveal the policy on login).
Call `login`; show backend errors (invalid credentials) as the Error state on the form and a toast for network failures.
After success redirect to the originally requested route or `/events`.
Disable the button with a loading state while submitting.
**Depends on:** T51 — Base UI components: actions and form fields; T52 — Base UI components: layout and data display; T55 — Authentication context provider; T56 — Routing and route guards.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code (frontend `master` at `b868dbb`):**
  - `src/App.tsx`:
    - `/login` route = `<RedirectIfAuthenticated><LoginPage/></RedirectIfAuthenticated>`;
    - **`LoginPage` is a T56 placeholder** (`data-testid="page-login"`, `<h1>Sign in</h1>`, and a hidden `<span data-testid="login-from" data-from={state.from}>`);
    - `RequireAuth` sends anonymous users to `/login` with `state.from = pathname+search+hash`;
    - **`RedirectIfAuthenticated` always navigates to `/events`.**
  - `src/features/auth/AuthProvider.tsx` (T55): `login(email, password)` calls `POST /api/auth/login`, stores the token, then **sets `status = "loading"`** and calls `refreshUser()` (`GET /api/users/me`). While `status === "loading"` the provider renders a full-page `auth-loading` instead of its children, which **unmounts the login form mid-submit**.
  - `src/shared/api/client.ts` (T50): `ApiError(status, message, fieldErrors)`. A 401 from `/api/auth/login` does **not** trigger the global logout handler. A network failure rejects with a `TypeError` from `fetch`, not with an `ApiError`.
  - `src/shared/ui`:
    - `AuthLayout` (T52): it already renders the whole left brand panel of the Figma frame (logo, headline, description, 3 features with icons, footer, and the two decor circles via `.ui-auth-brand::before/::after`);
    - `TextField` (`label`, `error`, `helperText`; Error state = `.ui-field-error` + a red message with `aria-describedby`);
    - `Button` (`loading` → disabled + spinner + "Loading…");
    - `useToast().error(title, message)` (T53).
  - Backend: `POST /api/auth/login` → 200 `{token}`. Wrong credentials → **401 `"Invalid email or password"`**. Blank fields → 400 "Request validation failed".
  - Tests: `cypress/e2e/routing-and-guards.cy.ts` (T56) relies on `page-login` and `login-from` and on authenticated users being redirected from `/login` to `/events`. `cypress/e2e/auth-context.cy.ts` (T55) has the `window.fetch` stub pattern.
- **Figma (read once, session `1fb50b79-…`):** `Student Screens` → frame `Login` (13:10), 1440×900.
  - **Left:** `brand-panel`, which is already `AuthLayout`.
  - **Right:** `form-panel`, a 400px column with 24px gaps:
    - title **"Welcome back"** (H1 32/40 Bold, `--text-primary`);
    - subtitle **"Sign in to see your events and organizations."** (Body 14/22, `--text-secondary`);
    - fields group (18px gap) with **Email** (placeholder `you@stud.ubbcluj.ro`) and **Password** (placeholder `••••••••`), each a 48px-high text field;
    - a full-width primary **"Sign in"** button (44px);
    - the line **"New to UBB Events?"** (secondary) + **"Create an account"** (SemiBold, `--accent-default`), centered.
  - No separate error or loading frame was read.
- **Related completed tasks:** T50 (API client), T51 (buttons and fields), T52 (`AuthLayout`), T53 (toast), T55 (auth context), T56 (routing and guards).
- **Constraints / assumptions:**
  - Frontend only; branch `T59-login-screen` from `master` after Gate 1.
  - Pixel fidelity agreed with the user: structure, texts, components and states exactly as in Figma, with the T49 tokens, without chasing every pixel.
  - Figma budget: 20 reads/month. T59 used 1 read; no more reads are planned for T59.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Replace the T56 placeholder with a real `LoginPage` in `src/features/auth/LoginPage.tsx`, rendered inside `AuthLayout`. The form panel follows the Figma frame: title, subtitle, Email and Password `TextField`s, a full-width `Button` "Sign in", and a centered "New to UBB Events? **Create an account**" line, where the link is a plain `react-router` `<Link to="/register">`.

- **Client validation** (on submit; the form uses `noValidate` so the browser's own bubbles don't show):
  - email required ("Email is required") and a valid format ("Enter a valid email address");
  - password **only required** ("Password is required"). There are no complexity rules on login, so existing users aren't blocked and the policy isn't revealed (Description).

  Errors appear in the fields' **Error state** (`TextField` `error`).
- **Submit:** `useAuth().login(email, password)`, with the button in `loading` (disabled, "Loading…") until the request finishes. No second request can be sent while one is in flight.
- **Backend errors:**
  - an `ApiError` (401 "Invalid email or password", or a 400) → the form's **Error state**: both fields marked invalid and the backend message shown under the password field;
  - a **network failure** (non-`ApiError`) → a **toast** `error("Couldn't sign in", "Check your connection and try again.")`.

  The typed values are kept.
- **Success:** go to the originally requested route (`location.state.from`) or `/events`.

Two small fixes in existing code are needed so that the redirect really works (open question 1):
1. **`RedirectIfAuthenticated`** (`App.tsx`) navigates to `state.from` when it's a safe internal path (it starts with `/` and not `//`), otherwise to `/events`. Without this, as soon as the user becomes authenticated, the guard on `/login` sends them to `/events` and the remembered route is lost.
2. **`AuthProvider.login`** no longer switches the whole app to `status = "loading"` (the full-page loader stays only for the initial token restore). Today that switch unmounts the form in the middle of the submit, so the button's loading state and the error handling are interrupted.

The `data-testid="page-login"` and the hidden `login-from` element from T56 are kept, so the existing guard tests stay valid.

**Alternatives considered:**
- Navigating from the form after `await login()` without touching the guard: rejected, because the guard reacts first and the form may already be unmounted.
- A form-level banner (`InfoBanner`) for invalid credentials: possible, but the Description says the "Error state on the form", which matches the field Error state of the design system (open question 2).

### Steps

1. **Tests first:** `cypress/e2e/login-screen.cy.ts` (see Tests to write). Run it and record a behavioral red (the placeholder has no form).
2. **UI:** `src/features/auth/LoginPage.tsx`:
   - `AuthLayout` with `testId="page-login"` (or a wrapper that keeps that test id);
   - a header (h2 "Welcome back" + subtitle, so the brand headline stays the page's h1);
   - `<form noValidate aria-label="Sign in">` with the two `TextField`s (`type="email"`/`autoComplete="email"` and `type="password"`/`autoComplete="current-password"`);
   - a full-width `Button type="submit" loading={submitting}`;
   - the register line with `<Link to="/register">`.

   Plus small CSS in `global.css` (`.auth-form…` classes using T49 tokens: the 24px/18px gaps, full-width button, centered register line, link colour `--accent-default`).
3. **Logic:** validation helper (`validateLogin`), submit handler, `ApiError` vs network failure handling, and keeping the values.
4. **Routing:** `App.tsx` imports the new page in place of the placeholder; `RedirectIfAuthenticated` honours a safe `state.from`.
5. **Auth:** `AuthProvider.login` without `setStatus("loading")`.
6. **Verification:**
   - focused spec green, then the full Cypress suite (with Vite running) plus `npm run typecheck`, `npm run lint`, `npm run build`;
   - a visual comparison with the Figma screenshot in the in-app browser (Vite `Network` URL), with no further Figma reads.

### Files to create / modify

Frontend (`repos/ubb-events-frontend/`, branch `T59-login-screen` from `master`):
- Create `src/features/auth/LoginPage.tsx`
- Create `cypress/e2e/login-screen.cy.ts`
- Modify `src/App.tsx` (remove the placeholder `LoginPage`, use the new page, `RedirectIfAuthenticated` honours `state.from`)
- Modify `src/features/auth/AuthProvider.tsx` (`login` without the full-app loading state)
- Modify `src/styles/global.css` (login form layout classes)
- Not committed: `package-lock.json` (changed only by the local `npm install` with an older npm; see the chat)

### Tests to write

All in `cypress/e2e/login-screen.cy.ts`, with `window.fetch` stubbed as in the existing specs (no backend needed).

- **AC1: UI and behaviour from the Description:**
  - `renders the login form from the design`: `page-login` is visible inside the auth layout; the texts "Welcome back" and "Sign in to see your events and organizations."; Email and Password fields with their placeholders; "Sign in" button; "Create an account" link with `href="/register"`.
  - `signs in and returns to the originally requested route`: visiting `/my-tasks` anonymously redirects to `/login`; submitting valid credentials sends `POST /api/auth/login` with `{email, password}`, stores the token, and the user lands on `/my-tasks`.
  - `signs in and goes to /events when there is no remembered route`.
  - `create account link navigates to /register`.
- **AC3: validation, error, loading and accessibility states:**
  - `validates required fields and email format without calling the API`: empty submit shows "Email is required" + "Password is required"; `ana@` shows "Enter a valid email address"; no fetch to `/api/auth/login`; the fields are `aria-invalid` and linked to their messages.
  - `does not apply password complexity rules on login`: the short password `abc` is accepted and sent.
  - `shows invalid credentials as the form error state`: 401 `{message: "Invalid email or password"}` → both fields invalid, the message is shown, the user stays on `/login`, values are kept, and there is no toast.
  - `shows a toast when the network fails`: a rejected fetch → `toast-error` visible, still on `/login`.
  - `disables the button with a loading state while submitting`: a pending login request → the button is disabled with "Loading…"; a second click sends no second request.
  - `keeps unsafe "from" values out of the redirect`: `state.from = "//evil.com"` → `/events`.
- **Regression:** `routing-and-guards.cy.ts` (T56) and `auth-context.cy.ts` (T55) must still pass unchanged.
- **Expected red:** the placeholder renders only "Sign in" text with no form, so the form tests fail on missing elements and the redirect test fails on the location. These are behavioral failures, not setup errors.

### Risks / open questions

**User decision at Gate 1: all five points confirmed as proposed ("all as proposed"). `package-lock.json` was restored with `git checkout` at the user's request; the installed versions (Node v22.22.0, npm 10.9.4) satisfy `engines` (Node ^22.12/^24/>=26, npm >=10), so no upgrade is needed. Use `npm ci` in the future so the lock file isn't rewritten.**
1. **Small changes in T55/T56 code** (`RedirectIfAuthenticated` honours `state.from`; `AuthProvider.login` doesn't switch to the full-page loader). Without them the Description's "redirect to the originally requested route" can't work. Both are minimal and the existing tests stay green. OK?
2. **"Error state on the form" for invalid credentials:** both fields in the Error state (red border) with the backend message under the password field (my proposal, no extra Figma read). Alternative: a form-level error banner above the button. The second option would need checking whether the design has such a frame (+1 Figma read).
3. **Network-failure toast text:** "Couldn't sign in" / "Check your connection and try again."
4. **Validation messages:** "Email is required", "Enter a valid email address", "Password is required".
5. **`package-lock.json`:** changed only by the local `npm install` (npm 10 removes the `libc` fields that the colleague's newer npm wrote). It is not committed with T59; restoring it with `git checkout` is suggested.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Setup:**
  - local `.env` with `VITE_API_URL=http://localhost:8080` (git-ignored; the app throws without it);
  - Vite running with `npm run dev -- --host` (`http://localhost:5173/`, Network `http://192.168.1.137:5173/`);
  - Cypress 16.2.0. All backend calls are stubbed via `window.fetch` in the spec.
- **Tests:** `cypress/e2e/login-screen.cy.ts`:
  - `renders the login form from the design`
  - `signs in and returns to the originally requested route`
  - `signs in and goes to /events when there is no remembered route`
  - `navigates to /register from the create account link`
  - `validates required fields and email format without calling the API`
  - `does not apply password complexity rules on login`
  - `shows invalid credentials as the form error state`
  - `shows a toast when the network fails`
  - `disables the button with a loading state while submitting`
  - `keeps unsafe remembered routes out of the redirect`
- **Command:** `npx cypress run --spec cypress/e2e/login-screen.cy.ts --browser electron --reporter spec`
- **Result:** **0 passing, 10 failing.** Test 1 found `page-login` and then failed on "Expected to find element: `.ui-auth-layout`, but never found it". The others failed on "Expected to find element: `[data-testid=login-email]` / `[data-testid=login-submit]`", or on "Expected to find content: 'Create an account' within the selector: 'a'".
- **Failure reason:** behavioral. `/login` still renders the T56 placeholder (only an `<h1>Sign in</h1>`), so the planned form, its validation, error, loading and redirect behaviour are missing. There were no compilation errors, and Vite, Cypress and the fetch stubs worked (the placeholder rendered and anonymous `/my-tasks` redirected to `/login`).

### Green (after implementation)

- **Result:** `npx cypress run --spec cypress/e2e/login-screen.cy.ts --browser electron --reporter spec` gives **10 passing, 0 failing**.
- **Full suite of the affected repo:** `npx cypress run --browser electron --reporter spec` gives **All specs passed: 62 of 62** (12 specs: the 52 existing tests, including T55 `auth-context` and T56 `routing-and-guards` unchanged, plus the 10 new ones). `npm run typecheck`, `npm run lint` and `npm run build` all exit 0.
- **Visual check** (in-app browser, Vite Network URL `http://192.168.1.137:5173/login`, viewport 1440×900, compared with the screenshot from the single Figma read): the form panel matches the frame (title, subtitle, Email and Password fields with placeholders, full-width "Sign in" button, "New to UBB Events? Create an account" line). The empty-submit Error state shows red borders with "Email is required" and "Password is required" under the fields. The Vite server was stopped afterwards.

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
  - `src/features/auth/LoginPage.tsx` replaces the T56 placeholder. It reuses `AuthLayout` (the left brand panel was already built in T52), `TextField`, `Button` and `useToast`. The form panel follows Figma `Login` (13:10); the only new CSS is the layout classes (`.auth-form…`), using the T49 tokens.
  - **Client validation** (`validateLogin`): email required plus a basic format check; password **only required**, so complexity rules apply to registration only (Description). The form uses `noValidate`, so the messages come from the app, not the browser. Errors use the field Error state (`aria-invalid` + an `aria-describedby` message).
  - **Errors after submit:**
    - an `ApiError` (401 "Invalid email or password", or a 400) → the form Error state: both fields invalid, with the backend message under the password field;
    - anything else (network failure) → a toast "Couldn't sign in / Check your connection and try again."

    The typed values are kept (user decisions).
  - **Loading:** the `Button` `loading` prop disables the button and shows "Loading…" while the request is in flight. A second submit is ignored.
  - **Redirect** (user-approved small changes in T55/T56 code):
    - `RedirectIfAuthenticated` now sends an authenticated user to `state.from` when it's a safe in-app path (it starts with `/` and not `//`), otherwise to `/events`;
    - `AuthProvider.login` no longer switches the whole app to the full-page loader, which unmounted the form mid-submit. The full-page loader is still used for the initial token restore.
  - The T56 test hooks `page-login` and `login-from` are kept, so the existing guard tests pass unchanged.
- **Alternatives considered:**
  - Navigating from the form after `await login()` with the guard unchanged: the guard redirects first, so the remembered route would be lost.
  - A form-level error banner for invalid credentials: it would need another Figma read to confirm the design; the field Error state matches the Description.
  - Keeping the full-page loader during login: it unmounts the form and interrupts its loading and error handling.
- **Limitations:**
  - **Out of scope, noticed during the visual check:** at a 1440×900 viewport, the left panel's headline "Every campus event, in one place." wraps onto 4 lines, while Figma shows 2 lines (a 560px text width). This is `AuthLayout`/T52 styling and wasn't changed in T59.
  - The email format check is intentionally simple (`x@y.z`). The backend remains the authority.
  - **Backend CORS:** the backend has no CORS configuration, so a real browser login from `http://localhost:5173` will be blocked until the backend allows the frontend origin. T59 behaviour is verified with stubbed responses; the fix belongs to a separate backend task.
  - If the profile request after a successful login fails with a non-401 error, `AuthProvider.refreshUser` clears the session and the user stays on the login page without a specific message (existing T55 behaviour).

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
- [x] Moved to `changes/completed/`


