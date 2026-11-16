# T67 - Profile screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T67
- **Type:** workflow
- **Owner:** Karina Kentsch
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T67-profile-screen`
- **PR link(s):** 
- **Tests:** `repos/ubb-events-frontend/cypress/e2e/profile-screen.cy.ts`; `npm run typecheck`, `npm run lint`, `npm run build`; full `npx cypress run --browser electron`

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 20 — Profile screen

**Description:** Implement page `Student Screens`, frame `Profile` (14:978): account card with avatar (initials from first and last name), **First name** and **Last name** fields side by side, read-only email with helper "Your email is your sign-in identifier and can't be changed", Save changes / Cancel; right column with "My organizations" (name + role badge per membership) and a Log out button (calls `logout()`).
Data from `GET /api/users/me` (returns `firstName` and `lastName`), save with `PUT /api/users/me` sending both fields; the email is never sent.
Validate that both names are required and trimmed, disable Save until the form is dirty and valid, show backend field errors on the matching input, refresh the auth context after saving so the navbar chip and avatars update, and show a success toast. Cancel resets the form to the saved values.
**Depends on:** T55 — Authentication context provider; T57 — App shell and top navigation.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [ ] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [ ] The task's stated dependency prerequisites are merged before implementation.
- [ ] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `../../repos/ubb-events-frontend/src/App.tsx` (guarded routes and placeholder), `src/features/auth/AuthProvider.tsx` (`user`, `refreshUser`, `logout`), `src/features/auth/types.ts` (profile and membership DTOs; membership `organizationName` is optional), `src/shared/api/auth.ts` and `src/shared/api/client.ts` (`GET /api/users/me`, API transport, and `ApiError.fieldErrors`), `src/shared/ui/AppShell.tsx`, `src/shared/ui/TextField.tsx`, and `src/shared/ui/FeedbackComponents.tsx` (toast); Cypress specs cover auth and shell behavior. Backend `RestServices/src/main/java/com/ubb/events/controllers/UserController.java` confirms `PUT /api/users/me` returns HTTP 200 with the updated `UserProfileResponse`.
- **Related completed tasks:** `T55-authentication-context-provider.md` and `T57-app-shell-and-top-navigation.md`; both dependencies are completed and their implementation is on frontend `master`. T57 established `Organization {organizationId}` as the membership-name fallback when `organizationName` is absent.
- **Constraints / assumptions:** Frontend-only workflow, branch `T67-profile-screen`. The frontend repo was clean on `master` (`## master...origin/master`). Fetch/pull was attempted but unavailable due to a network DNS/thread failure; the local branch reports no ahead/behind difference. Preserve existing auth behavior and use the established membership-name fallback. `project-context.md` does not define frontend test scripts; inspect `package.json` during planning.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Implement the profile as a guarded authenticated page that composes the existing app shell, auth context, shared form controls, avatar/badge primitives, and toast provider. Add a small API wrapper for the profile update, keep form state local, and call `refreshUser()` after a successful update so the shell consumes the new identity immediately. Membership names will use `organizationName` when present and the established `Organization {organizationId}` fallback otherwise. Keep email display-only and omit it from the update payload.

### Steps

1. **API/shared layer — write tests first:** add coverage for the `PUT /api/users/me` request shape and verify only trimmed `firstName` and `lastName` are sent; confirm API errors preserve field-level messages through `ApiError.fieldErrors`.
2. **UI — write Cypress tests first:** add profile-screen scenarios against stubbed `/api/users/me` responses for initial rendering, initials and memberships, read-only email/helper, validation and dirty-state button rules, cancel reset, successful save/refresh/toast, backend field errors, update failure, loading/error/empty membership states, logout, and accessible labels/roles. Run the focused tests to capture the required behavioral red result before implementation.
3. **API/shared implementation:** add the typed update payload and `PUT /api/users/me` wrapper in `src/shared/api/auth.ts` (or the existing user API module), without including email; reuse the shared API error mapping.
4. **UI implementation:** create a profile page component under `src/features/profile/` (or the established feature location), initialize local names from `useAuth().user`, trim on validation/submit, show matching field errors, and disable Save unless values are dirty and both trimmed names are non-empty. Render the account card and responsive two-column organization/logout area with accessible semantics and the specified helper text. Cancel restores the last saved values.
5. **Routing/auth integration:** replace the `/profile` placeholder in `src/App.tsx` with the new page inside `RequireAuth` and `AppShell`; call `refreshUser()` after PUT success, show a success toast, and use `logout()` for the right-column action.
6. **Verification:** run focused Cypress tests, then frontend `typecheck`, `lint`, `build`, and the wider Cypress suite; record exact red/green commands and outcomes in this task file.

### Files to create / modify

- `repos/ubb-events-frontend/src/features/profile/ProfilePage.tsx` (new profile UI and form state), plus feature-local styles only if existing shared styles cannot express the frame.
- `../../repos/ubb-events-frontend/src/shared/api/auth.ts` or a new `src/shared/api/users.ts` (typed PUT wrapper and update DTO).
- `../../repos/ubb-events-frontend/src/App.tsx` (replace `/profile` placeholder).
- `repos/ubb-events-frontend/cypress/e2e/profile-screen.cy.ts` (focused API/UI behavior and accessibility tests).
- `../../repos/ubb-events-frontend/src/styles/global.css` only for profile layout styles if needed after reusing existing tokens/primitives.

### Tests to write

- **Acceptance criterion 1 / dependencies:** profile tests run with the merged T55 auth provider and T57 shell, and verify the authenticated route, navbar identity refresh, and logout redirect.
- **Acceptance criterion 1 / behavior:** assert GET data renders avatar initials, both editable names, read-only email/helper, membership name and role badge, and the specified controls; assert PUT contains exactly trimmed names and never email, then refreshes the auth profile and shows a success toast.
- **Acceptance criterion 1 / validation/error handling:** assert Save is disabled when pristine, whitespace-only, or invalid; Cancel restores saved values; backend `firstName`/`lastName` errors attach to matching inputs; non-field update failure displays an error state/toast without losing edits.
- **Acceptance criterion 2 / loading, empty, accessibility:** assert loading indicators while profile data is pending, a usable error state on profile load failure, an empty-state message for no memberships, labeled inputs/buttons, read-only email semantics, and keyboard-accessible organization/logout controls.

### Risks / open questions

- `PUT /api/users/me` returns HTTP 200 with the updated profile; the UI will still call `refreshUser()` as required so auth context and navigation use the canonical current profile.
- Membership responses may omit organization names; use the existing deterministic fallback rather than adding an unplanned organizations lookup.
- No frontend unit-test runner exists in `package.json`; Cypress is the API-level/UI test mechanism, with typecheck/lint/build as additional verification.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `npx cypress run --browser electron --spec cypress/e2e/profile-screen.cy.ts`
- **Result:** 0 passing, 1 failing; `profile screen — renders the profile and saves trimmed names without sending email` timed out looking for `[data-testid=page-profile]`.
- **Failure reason:** The authenticated `/profile` route still renders the placeholder and has no profile screen test target. This is a behavioral failure caused by the planned functionality being absent; Cypress and the Vite server ran successfully after installing the local Cypress binary.

### Green (after implementation)

- **Result:** Focused `npx cypress run --browser electron --spec cypress/e2e/profile-screen.cy.ts` passed: 1 passing. `npm run typecheck`, `npm run lint`, and `npm run build` passed.
- **Full suite of the affected repo:** `npx cypress run --browser electron` completed 126 tests: 124 passed, 2 failed in pre-existing `organizations-list.cy.ts` visibility assertions at lines 41 and 60; T67's profile spec passed. These failures are outside the profile files and were not changed by this task.

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

- **Decisions:** Draft for review: profile form state stays local; only trimmed first and last names are sent to `PUT /api/users/me`; a successful response is followed by `refreshUser()` so the shared auth context updates the navbar identity. Backend field errors map through `ApiError.fieldErrors`. Membership names use `organizationName` with the established organization-ID fallback.
- **Alternatives considered:** Draft for review: email remains a disabled shared `TextField` rather than being part of the update payload; no separate user query or organization lookup was added because the auth context already owns the profile and membership data.
- **Limitations:** Draft for review: the full frontend Cypress suite has two unrelated existing failures in `organizations-list.cy.ts`; the profile screen itself passes focused Cypress coverage. Styling uses existing shared primitives and lightweight page classes; visual comparison against the Figma frame was not run.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [x] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [ ] Implementation reviewed by user
- [ ] **Gate 2:** approved for commit and push
- [ ] Committed and pushed
- [ ] PR created
- [ ] PR reviewed
- [ ] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [ ] Moved to ``


