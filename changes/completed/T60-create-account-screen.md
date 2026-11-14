# T60 - Create account screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T60
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** 
- **Use case(s):** UC-01
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T60-create-account-screen`
- **PR link(s):** Frontend: https://github.com/Vasile07/ubb-events-frontend/pull/14 (stacked on #12, T59)
- **Tests:** `cypress/e2e/register-validation.cy.ts` (new, unit), `cypress/e2e/create-account-screen.cy.ts` (new, e2e); regression: `routing-and-guards.cy.ts`, `login-screen.cy.ts` unchanged

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 13 — Create account screen

**Description:** Implement page `Student Screens`, frame `Create account` (13:62) using `AuthLayout`: **First name** and **Last name** side by side, email, password (helper "Min. 8 characters with upper & lower case, a number and a symbol"), the "Create account" button and a link to sign in (`/login`; it is only a route path, the login page does not need to exist yet).
Request: `POST /api/auth/register` with `firstName`, `lastName`, `email`, `password`.
Client validation:
- first name, last name and email required (trimmed), valid email format
- password: minimum 8 characters, at least 1 lowercase letter, 1 uppercase letter, 1 number and 1 special character. Implement it as a pure `validatePassword` helper returning the list of unmet rules (unit tested) and show which rules are missing in the field's error text.

Show the Error state on the email field for "email already registered" (409, as drawn in the frame), map other validation errors from `ApiError.fieldErrors` to the matching fields (including `firstName`, `lastName`, `password`) and use a toast for network errors.
On success sign the user in (or redirect to `/login`) and go to `/events`. Disable the button with a loading state while submitting.
**Depends on:** T51 — Base UI components: actions and form fields; T52 — Base UI components: layout and data display; T55 — Authentication context provider; T56 — Routing and route guards.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:**
  - `src/App.tsx`: `/register` renders `PlaceholderPage` (`page-register`) inside `RedirectIfAuthenticated`.
  - `src/shared/api/auth.ts`: `register(payload)` → `POST /api/auth/register`. `src/features/auth/types.ts`: `RegisterPayload { firstName, lastName, email, password }`, `UserResponse`.
  - `src/features/auth/AuthProvider.tsx`: `register` (request only, no sign-in) and `login`.
  - `src/shared/api/client.ts`: `ApiError(status, message, fieldErrors)`. `fieldErrors` maps the backend `errors[].field` to `message`. A 401/403 on register does not trigger the global logout.
  - `src/shared/ui`: `AuthLayout`, `TextField` (`helperText`, `error`), `Button` (`loading`), `useToast`.
  - Backend `RegisterUserRequest`: `@NotBlank` firstName, lastName, email; `@Email`; password `@Size(min=8)` plus a number, an uppercase letter, a lowercase letter and a special character (`[^A-Za-z0-9]`). Validation errors come back as 400 "Request validation failed" with `errors[{field, message}]`, e.g. `firstName is required`. A duplicate email gives 409 "Email already used" (`UserServiceImpl.register`).
- **Related completed tasks:**
  - T59 Login screen (frontend PR #12, **not merged yet**). It added the shared `.auth-form…` CSS, the safe `from` redirect in `RedirectIfAuthenticated`, and the `AuthProvider.login` fix that keeps the form mounted while signing in.
  - T55 Auth context, T56 Routing and guards, T51/T52 base UI.
- **Figma** (single read, session `01591a6e-…`, frame `Create account` 13:62):
  - The left brand panel matches `AuthLayout`, so no new assets are needed.
  - Form panel, 400px wide:
    - title "Create your account" (H1 32/40);
    - subtitle "Join UBB Events to discover and register for events.";
    - name row: "First name" (placeholder `Andrei`) and "Last name" (`Pop`) side by side, 16px gap;
    - Email (`andrei.pop@stud.ubbcluj.ro`), drawn in the Error state with "This email address is already registered";
    - Password (`••••••••`) with the helper "Min. 8 characters with upper & lower case, a number and a symbol";
    - a full-width "Create account" button;
    - the line "Already have an account? Sign in".
  - In the Error state, the error text replaces the helper text.
- **Constraints / assumptions:**
  - There is no unit-test runner in the frontend (only Cypress), and no spec imports `src/` yet.
  - The backend has no CORS configuration (see T59), so the tests stub `fetch`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Replace the `/register` placeholder with a `RegisterPage` that follows the Figma frame and reuses `AuthLayout`, `TextField`, `Button`, `useToast` and the T59 `.auth-form…` styles. Password rules live in a pure `validatePassword` helper that returns the unmet rules. On success, the user is signed in automatically and the existing guard sends them to `/events`.

Decisions (Gate 1: user approved all five as proposed, "all as proposed gate 1 approved"):
1. **Branch base:** create frontend `T60-create-account-screen` **from `T59-login-screen`** (stacked), because T60 needs the T59 form CSS and the `AuthProvider.login` fix. After PR #12 is merged, `master` is merged into T60 (no rebase), and the T60 PR then shows only T60 changes. The alternative is branching from `master` and duplicating that CSS, which gives a conflict in `global.css`.
2. **After a successful register:** call `login(email, password)` automatically, and the guard redirects to `/events`. If that automatic sign-in fails, navigate to `/login` with an info toast: "Account created. Please sign in."
3. **The 409 text:** show the Figma text "This email address is already registered" on the email field, instead of the backend's "Email already used".
4. **Password error text:** the error replaces the helper and lists the missing rules, e.g. "Password needs an uppercase letter, a number and a symbol." The rule names follow the helper: at least 8 characters, a lowercase letter, an uppercase letter, a number, a symbol. "Symbol" uses the same definition as the backend: any character that is not a letter or a digit.
5. **Unit tests for `validatePassword`:** write them in a Cypress spec that imports the helper directly, with no new dependency. The alternative is adding Vitest.

### Steps

1. Create the frontend branch (per decision 1) after checking `git status`.
2. Write the tests first: `register-validation.cy.ts` (unit, helper) and `create-account-screen.cy.ts` (e2e, `fetch` stubbed). Add the minimum stubs so they compile: `validatePassword` returning `[]`, `validateRegister` returning `{}`, and a `RegisterPage` that still renders the placeholder.
3. Run the tests and record a behavioral red result.
4. Implement:
   - `src/features/auth/registerValidation.ts`:
     - `validatePassword(password): PasswordRule[]` returns the unmet rules in a fixed order;
     - `describeMissingRules(rules)` builds the error text;
     - `validateRegister(values)`: first name, last name and email are required after trimming; email must have a valid format (same pattern as login); the password rules apply.
   - `src/features/auth/RegisterPage.tsx`:
     - form per Figma: title, subtitle, name row, Email, Password with helper, "Create account" button, and the "Already have an account? Sign in" link to `/login`;
     - `noValidate`; validate on submit; trimmed values are sent and the password is sent as is;
     - after submit:
       - 409 → the Error state on email;
       - 400 with `fieldErrors` → mapped onto `firstName`, `lastName`, `email` and `password`;
       - any other `ApiError` → toast "Couldn't create account" with the backend message;
       - a non-`ApiError` (network failure) → toast "Couldn't create account / Check your connection and try again."
     - the button is in the loading state while submitting, and a second submit is ignored. Typed values are kept after an error.
   - `src/App.tsx`: render `RegisterPage` on `/register`, keeping the `page-register` test id.
   - `src/styles/global.css`: `.auth-form-name-row` (two columns, 16px gap, stacking on narrow screens).
5. Run the tests to green, then the full Cypress suite, typecheck, lint and build. Do a visual check against the Figma frame.

### Files to create / modify

- New: `src/features/auth/registerValidation.ts`, `src/features/auth/RegisterPage.tsx`, `cypress/e2e/register-validation.cy.ts`, `cypress/e2e/create-account-screen.cy.ts`.
- Modify: `src/App.tsx` (the route) and `src/styles/global.css` (the name row).

### Tests to write

- `register-validation.cy.ts` (unit):
  - `validatePassword` returns no rules for a valid password;
  - it returns each missing rule (length, lowercase, uppercase, number, symbol);
  - it returns all rules for an empty password;
  - `describeMissingRules` builds the text;
  - `validateRegister` requires trimmed names and email and checks the email format.
- `create-account-screen.cy.ts` (e2e):
  - renders the form from the design (texts, side-by-side names, helper, Sign in link);
  - validates required fields and email format without calling the API;
  - shows the missing password rules in the field error;
  - sends the trimmed payload to `POST /api/auth/register`;
  - signs in after registering and goes to `/events`;
  - shows 409 as the email Error state;
  - maps backend field errors to the matching fields;
  - shows a toast on network failure;
  - disables the button with a loading state while submitting;
  - falls back to `/login` when the automatic sign-in fails;
  - "Sign in" navigates to `/login`.
- Regression: the full Cypress suite, including the T56 `routing-and-guards` spec, which checks `page-register`.

### Risks / open questions

- Decisions 1–5 above need confirmation.
- If T59 changes during review, T60 picks those changes up via the merge from `T59-login-screen`/`master`.
- The real register flow in the browser still depends on the backend CORS fix (out of scope).

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:**
  - `cypress/e2e/register-validation.cy.ts` (unit; imports `src/features/auth/registerValidation.ts` directly), 8 tests:
    - `validatePassword returns no rules for a valid password`
    - `validatePassword returns each missing rule`
    - `validatePassword returns every rule for an empty password`
    - `validatePassword treats any non letter or digit as a symbol, like the backend`
    - `describeMissingRules lists the missing rules in the field error text`
    - `validateRegister accepts valid values`
    - `validateRegister requires trimmed names and email`
    - `validateRegister checks the email format and the password rules`
  - `cypress/e2e/create-account-screen.cy.ts` (e2e, `fetch` stubbed), 12 tests:
    - `renders the create account form from the design`
    - `validates required fields and email format without calling the API`
    - `shows the missing password rules in the field error`
    - `sends the trimmed payload to POST /api/auth/register`
    - `signs in after registering and goes to /events`
    - `falls back to /login when the automatic sign-in fails`
    - `shows an already registered email as the email Error state`
    - `maps backend field errors to the matching fields`
    - `shows other server errors in a toast`
    - `shows a toast when the network fails`
    - `disables the button with a loading state while submitting`
    - `navigates to /login from the sign in link`
  - Stubs used so the tests compile: `validatePassword` returns `[]`, `describeMissingRules` returns `""`, `validateRegister` returns `{}`, and `RegisterPage` still renders the placeholder (`<main data-testid="page-register"><h1>Create account</h1></main>`), wired into `/register`.
- **Command:** `npx cypress run --spec "cypress/e2e/register-validation.cy.ts,cypress/e2e/create-account-screen.cy.ts" --browser electron --reporter spec` (Vite running on `127.0.0.1:5173`; `npm run typecheck` exit 0 beforehand).
- **Result:**
  - Unit spec: **3 passing, 5 failing.** The failures are assertion errors on the stubs' empty results, e.g. "expected [] to deeply equal [ 'length' ]", "expected '' to equal 'Password needs a symbol.'", "expected {} to deeply equal { firstName, lastName, … }". The 3 passing tests only assert the "nothing missing" case (a valid password and valid values), which the empty stubs satisfy by construction.
  - Screen spec: **0 passing, 12 failing**, with "Expected to find element: `.ui-auth-layout`" / "`[data-testid=register-first-name]`" / "`[data-testid=register-submit]`" never found, and "Expected to find content: 'Sign in' within the selector: 'a'".
- **Failure reason:** behavioral. The validation rules are missing, and `/register` renders only the placeholder: no form, validation, error mapping, loading state or automatic sign-in. There were no compilation errors, and Vite, Cypress and the stubs worked (the placeholder rendered at `/register`).

### Green (after implementation)

- **Result:** the same command gives **8 passing, 0 failing** (`register-validation.cy.ts`) and **12 passing, 0 failing** (`create-account-screen.cy.ts`).
- **Full suite of the affected repo:** `npx cypress run --browser electron --reporter spec` gives **All specs passed: 82 of 82** (14 specs: the 62 tests on the T59 branch, including `routing-and-guards` with its `page-register` check and `login-screen`, plus the 20 new ones). `npm run typecheck`, `npm run lint` and `npm run build` all exit 0.
- **Visual check** (in-app browser, Vite Network URL `http://192.168.1.137:5173/register`, viewport 1440×900, compared with the screenshot from the single Figma read):
  - the form panel matches the frame: title, subtitle, First and Last name side by side, Email, Password with the helper, a full-width "Create account" button, and "Already have an account? Sign in";
  - measured: a 400px form, name fields 192px wide each with a 16px gap, 48px-tall fields and a 44px button;
  - submitting with an invalid email and a weak password shows the Error state (red border `rgb(178, 58, 72)`), and the missing password rules replace the helper text.

  The Vite server was stopped afterwards.

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
  - `src/features/auth/RegisterPage.tsx` replaces the T56 placeholder at `/register` and keeps the `page-register` test id. It reuses `AuthLayout`, `TextField`, `Button`, `useToast` and the T59 `.auth-form…` styles. The only new CSS is `.auth-form-name-row` (two equal columns with a 16px gap, stacking under 480px).
  - **Validation lives in a pure module** (`registerValidation.ts`, no React), so it can be unit tested directly:
    - `validatePassword` returns the unmet rules in a fixed order (length, lowercase, uppercase, number, symbol). These are the same rules as the backend `RegisterUserRequest`, where a "symbol" is any character that isn't a letter or a digit;
    - `describeMissingRules` turns those rules into the field error text;
    - `validateRegister` checks the trimmed names and email, the email format and the password. An empty password shows "Password is required" instead of the full rule list.
  - **Submit** (Gate 1 decisions):
    - trimmed names and email are sent, and the password is sent exactly as typed;
    - 409 → the email Error state with the Figma text "This email address is already registered";
    - a 400 with `fieldErrors` → the backend messages under the matching fields;
    - any other `ApiError` → toast "Couldn't create account" with the backend message;
    - a network failure → toast "Couldn't create account / Check your connection and try again."

    Typed values are kept, and the button is in the loading state while submitting.
  - **After a successful register**, the user is signed in automatically with the same credentials (`useAuth().login`), and the route guard sends them to `/events`. If that sign-in fails, the page goes to `/login` with a toast "Account created / Please sign in."
  - In the Error state the error text replaces the password helper, as drawn in Figma.
  - Branch `T60-create-account-screen` was created from `T59-login-screen` (Gate 1 decision 1). Once frontend PR #12 is merged, `master` is merged into this branch (no rebase).
- **Alternatives considered:**
  - Redirecting to `/login` after registering: this is simpler, but the user has to type the same credentials again.
  - Showing the backend's "Email already used" text: rejected in favour of the Figma text.
  - Adding Vitest for unit tests: it would be a new dependency; Cypress can import the pure helper directly.
  - A live password checklist while typing: it isn't in the Figma frame or the Description.
- **Limitations:**
  - The design has no "info" toast type (only Success and Error), so the "Account created. Please sign in." fallback uses the Success toast.
  - The email pattern is duplicated from `LoginPage` (one line), so the T59 file wasn't touched. The backend remains the authority.
  - If the profile request after the automatic sign-in fails with a non-401 error, the user stays on `/register` with the button still loading. This is the same existing `AuthProvider.refreshUser` behaviour noted in T59.
  - A real sign-up from the browser still needs the backend CORS fix (see T59).
  - This branch's PR also shows the T59 commit until PR #12 is merged and `master` is merged in.

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


