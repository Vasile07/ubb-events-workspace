# T48 - Project setup

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T48
- **Type:** deployment
- **Owner:** 
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T48-project-setup`
- **PR link(s):** https://github.com/Vasile07/ubb-events-frontend/pull/1
- **Tests:** `cypress/e2e/home.cy.ts`; npm typecheck, lint, build, and Cypress open/run scripts; Prettier check

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 1 — Project setup

**Description:** Create the frontend with React + TypeScript + Vite (change if another stack is chosen). Add React Router, TanStack Query, ESLint, Prettier and the folder structure by feature: `features/auth`, `features/events`, `features/organizations`, `features/workspace`, `shared/ui`, `shared/api`, `shared/lib`. Configure path aliases, strict TypeScript, and `.env.example` with `VITE_API_URL`. Frontend API calls use this URL directly without a development proxy or hard-coded port; never commit `.env`. Add scripts `dev`, `build`, `lint`, `typecheck` and a README with run instructions. Done when all scripts pass on an empty app shell with a placeholder home route.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] The frontend is scaffolded with React, TypeScript, and Vite, with strict TypeScript, the requested feature/shared folder structure, path aliases, React Router, TanStack Query, ESLint, and Prettier configured.
- [x] The app shell serves a placeholder home route, and the `dev`, `build`, `lint`, and `typecheck` npm scripts run successfully.
- [x] Cypress is configured and a smoke test verifies that the placeholder home route renders.
- [x] `VITE_API_URL` is documented through a committed `.env.example`; no `.env` file is committed. Frontend API calls use that base URL directly, without a development proxy or hard-coded port.
- [x] The frontend README documents prerequisites, environment setup, and how to run the development server, build, lint, typecheck, and Cypress smoke test.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:** `repos/ubb-events-frontend/` is an empty Git checkout with no tracked application files yet. `repos/ubb-events-backend/RestServices/src/main/resources/application.properties` contains no explicit server port.
- **Related completed tasks:** None found for frontend setup.
- **Constraints / assumptions:** `context/project-context.md` specifies React for the frontend and leaves frontend test/build commands open for this task to establish. User confirmed npm, Cypress setup now, `.env.example` committed, and `.env` never committed. User also directed that frontend calls use `VITE_API_URL` directly, without a proxy or hard-coded port. The GitHub `master` branch was initialized with an empty commit and the local checkout tracks `origin/master`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Create a minimal React + TypeScript + Vite frontend on the existing GitHub `master` baseline. Use npm and configure the dependencies and project conventions named in the description: routing, query client, linting, formatting, strict TypeScript, feature-oriented folders, path aliases, environment configuration, and Cypress. Keep the initial UI to an empty application shell with a placeholder home route so the setup can be verified independently of later feature work. Commit `.env.example` only and never commit `.env`. Use `VITE_API_URL` as the direct base URL for backend API calls; do not configure a development proxy or hard-code a port.

The alternative is to introduce a more complete application scaffold or implement feature APIs/UI now. That would add requirements beyond project setup, so it is excluded. The choice of Vite, direct API URL configuration, and the exact Cypress smoke command should be recorded in the task evidence and Drive/ADR if the team requires those decisions documented.

### Steps

1. **Repository/configuration:** Confirm the frontend checkout is on the clean `master` baseline and create the task branch only after Gate 1. Initialize the Vite React TypeScript package manifest and npm lockfile, with scripts for `dev`, `build`, `lint`, and `typecheck`; add ESLint, Prettier, React Router, and TanStack Query configuration.
2. **Shared/application shell (UI):** Add strict TypeScript settings, path aliases, the requested `features/*` and `shared/*` directories, the app entry point, a query client provider, and a placeholder home route through React Router. Add `.gitkeep` files to requested folders that would otherwise be empty, and remove each `.gitkeep` when that folder gains a real file. Keep the route accessible without authentication because no auth behavior is specified here.
3. **Configuration (external/API boundary):** Add `.env.example` containing `VITE_API_URL` without secrets and configure frontend API calls to use that base URL directly. Do not configure a development proxy or hard-code a port. Never commit a `.env` file.
4. **Documentation:** Add the frontend README with prerequisites, environment setup, and commands for running the development server, build, lint, typecheck, and Cypress smoke test.
5. **Verification:** Write the Cypress shell smoke test and any setup/configuration checks first, obtain a behavioral red result for the missing shell/configuration behavior, then implement and rerun the checks. Verify that the production build, lint, and strict typecheck pass and that Cypress confirms the placeholder home route renders.

### Files to create / modify

- `repos/ubb-events-frontend/package.json` and lockfile: scripts and dependencies.
- `repos/ubb-events-frontend/vite.config.*`: React plugin and path aliases (exact filename/format follows the chosen Vite setup).
- `repos/ubb-events-frontend/tsconfig*.json`: strict compiler options and aliases.
- `repos/ubb-events-frontend/eslint.config.*`, Prettier configuration, and ignore files.
- `repos/ubb-events-frontend/index.html`, `src/main.*`, `src/App.*`, router/query setup, and the placeholder home route.
- `repos/ubb-events-frontend/src/features/auth/`, `src/features/events/`, `src/features/organizations/`, `src/features/workspace/`, `src/shared/ui/`, `src/shared/api/`, and `src/shared/lib/`: requested directories with `.gitkeep` placeholders where no real file is needed yet; remove a placeholder when adding the first actual file to its directory.
- `repos/ubb-events-frontend/.env.example` (no real `.env` committed).
- `repos/ubb-events-frontend/README.md`: setup and run instructions.
- Cypress configuration and frontend smoke test files under the repository's Cypress layout.

### Tests to write

- **Description / shell:** a Cypress smoke test that loads the root route and verifies the placeholder home content is rendered.
- **Scripts/configuration:** verify `npm run build`, `npm run lint`, and `npm run typecheck` complete successfully with strict TypeScript enabled; include a failure case for missing/invalid required environment configuration if the implementation makes `VITE_API_URL` mandatory at startup.
- **API configuration:** verify the frontend uses `VITE_API_URL` directly for backend API endpoints, with no development proxy and no hard-coded port.
- Because this is a deployment/setup task, there are no business-rule, authentication, registration, or feature API tests to add; those belong to later tasks.

### Risks / open questions

- No open questions remain from planning.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:** `cypress/e2e/home.cy.ts` was written before the application shell. A pre-implementation shell assertion checked the root document for the expected `UBB Events home` content.
- **Result:** `Get-Content -Raw index.html` followed by a shell assertion failed with `Behavioral red: placeholder home content is missing from the root document`.
- **Failure reason:** This assertion inspected the static HTML source rather than rendering the React route, so it is not valid behavioral red evidence for the planned Cypress smoke test. Cypress was not run before the app shell was implemented. This is a TDD deviation; leave the red status item unchecked.

### Green (after implementation)

- **Result:** Final `npm run typecheck`, `npm run lint`, and `npm run build` passed. `npx prettier --check .` passed. `npm run cypress:run` passed with 1 passing test for `home.cy.ts`, using the installed Cypress cache and Vite test server at `http://127.0.0.1:5174`. The attempted post-format Cypress rerun was rejected by the approval gate, so the recorded Cypress pass is from immediately before formatting-only changes. The npm lockfile was generated; no `.env` file was created.
- **Full suite of the affected repo:** No wider suite exists for this new frontend repository. Final strict typecheck, lint, build, and Prettier checks passed; the Cypress home-route smoke test passed 1/1 before formatting-only changes.

## Human Corrections

<!--
Evidence of human oversight.
Agents add an entry ONLY when the user points out a problem or requests a change,
in the user's own terms. Never on the agent's own initiative.
-->

| # | Phase (plan / implementation) | What was wrong | What was changed or rejected | Why |
|---|-------------------------------|----------------|------------------------------|-----|
| 1 | plan | “Yes, we should replace for this task the acceptance criteria”; “npm, pacakge maanger”; “Yes, we should commit an .env.example, never commit .env”; “We should also configure the Cypress now”; “We should not put a port, we should add an API_URL directly”; “we should not use a proxy, call directly the backend api endpoints”; “for empty folders please put a .gitkeep files that will be removed after first actual file created” | Replaced the generic criteria with T48-specific setup criteria, selected npm, included Cypress setup in this task, specified `.env.example` only, configured direct backend API calls through `VITE_API_URL` without a proxy or hard-coded port, and added the requested `.gitkeep` lifecycle for empty folders. | These are the user's requested scope and configuration decisions for T48. |
| 2 | implementation | “ok i tested and it works, please just add one more script in package.json for cypress open” | Added the `cypress:open` npm script to launch Cypress interactively. | The user requested an interactive Cypress command after testing the implementation. |

## Design decisions, alternatives, limitations

<!-- Study note for the defense. Decisions made, alternatives considered, known limitations. -->

- **Decisions:** Vite React TypeScript was selected with strict compiler settings, feature-oriented folders, React Router, TanStack Query, ESLint, Prettier configuration, and Cypress. API calls use `VITE_API_URL` directly; no proxy or hard-coded API port is configured. `.env.example` is committed while `.env` is ignored. Empty feature/shared UI folders retain `.gitkeep` placeholders.
- **Alternatives considered:** A proxy or hard-coded backend port was excluded per the user's direction. A larger feature scaffold was excluded because T48 is limited to project setup.
- **Limitations:** Cypress was not run before implementation, so the required pre-implementation behavioral red evidence is missing and the red status remains unchecked. Cypress passed after implementation, before formatting-only changes; a requested rerun was rejected by the approval gate. The scaffold contains no feature behavior or backend integration, which belong to later tasks.

## Status

- [x] Task read and context gathered
- [x] Plan written
- [x] **Gate 1:** plan approved by user
- [ ] Tests written and confirmed failing (red)
- [x] Implementation done, tests passing (green)
- [x] Implementation reviewed by user
- [x] **Gate 2:** approved for commit and push
- [x] Committed and pushed
- [x] PR created
- [x] PR reviewed
- [x] **Gate 3:** merged by user
- [ ] Metadata complete (traceability)
- [x] Moved to `changes/completed/`


