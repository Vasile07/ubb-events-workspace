# T64 - My tasks screen

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T64
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** 
- **Use case(s):** 
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T64-my-tasks-screen`
- **PR link(s):** Frontend: https://github.com/Vasile07/ubb-events-frontend/pull/15
- **Tests:** `cypress/e2e/my-tasks-helpers.cy.ts` (new, unit), `cypress/e2e/my-tasks-screen.cy.ts` (new, e2e); regression: `routing-and-guards.cy.ts`, `login-screen.cy.ts`, `app-shell-and-navigation.cy.ts` unchanged

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 17 — My tasks screen

**Description:** Implement page `Student Screens`, frame `My tasks` (14:547): three stat tiles (assigned tasks, events involved, organizations) computed on the client, then one card per event with date block, title, organization, status badge, "Open event workspace" button and the list of the user's tasks.
Data from `GET /api/users/me/tasks`; group by event and sort by event start.
The workspace button goes to `/org/:organizationId/events/:eventId/tasks`.
Add loading (skeleton cards), an empty state ("No tasks assigned yet", Users or Calendar icon) and an error state.
**Depends on:** T53 — Feedback components: toast, empty state, skeleton, error state; T57 — App shell and top navigation.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code:**
  - Backend `GET /api/users/me/tasks` (`MyTaskController`, T27): returns `[{ id, description, event: { id, title, date, startTime, status }, organization: { id, name } }]`.
    - It returns every task assigned to the user, including tasks of DRAFT and CANCELED events, ordered by event `date`, then `startTime`, then task id.
    - `EventStatus` is `DRAFT | PUBLISHED | CANCELED`.
    - The endpoint needs authentication only, not organization membership.
  - Frontend `src/App.tsx`: `/my-tasks` is `authPage("page-my-tasks", "My tasks")`, a placeholder inside `RequireAuth` + `AppShell`. `/org/:organizationId/events/:eventId/tasks` is a `memberPage` placeholder (T72).
  - `src/features/events/api.ts` + `types.ts` with `DiscoverEventsPage.tsx`: the pattern for feature api, types, `useQuery`, and the loading/empty/error states.
  - `src/shared/ui`:
    - `PageHeader`, `DateBlock`, `Badge` (Success/Warning/Danger…), `Button` (Secondary, Sm) and `Icon` (`calendar`, `building`, `clock` are Figma assets);
    - `EmptyState` (icons Users/Calendar), `Skeleton`, `ErrorState` (`onRetry`).

    The `tasks` icon is a placeholder list path, used only in the showcase; the Figma `icon/tasks` is a check-square.
  - `src/shared/lib/formatEventDate.ts` parses the date and formats it in **UTC**. With a local `date`+`startTime` string, a browser in Bucharest shows 18:00 as 16:00 (checked with Node, `TZ=Europe/Bucharest`). This affects Discover (T61/T54) and is out of scope here.
- **Related completed tasks:**
  - T27 View my tasks (backend endpoint and its decisions);
  - T53 Feedback components;
  - T57 App shell and top navigation (it already highlights "My tasks");
  - T61 Discover events (page pattern).
- **Figma** (single read, session `5d653ec4-…`, frame `My tasks` 14:547). Content area padding is 48/80px, with a 28px gap between sections.
  - **Header:** "My tasks" (H1 32/40) and "Tasks assigned to you by event organizers." (16/24).
  - **Three stat tiles**, side by side with a 16px gap. Each tile is white, with a 1px border, 20px radius and 20px padding, and has:
    - a 52px soft-blue icon box with a 14px radius and a 24px icon;
    - a number (32/40 bold);
    - a label (14/22).

    The tiles are "Assigned tasks" (tasks icon), "Events involved" (calendar) and "Organizations" (building).
  - **One card per event**, white, 20px radius, 24px padding, 18px gap. Each card has:
    - a header with:
      - a date block (64×72, month "NOV" 12/16, day "14" or "05" 24/32 bold);
      - the title (18/26);
      - a meta line: building icon + organization, clock icon + "Sat, 14 Nov · 18:00";
      - the status badge ("Published" Success, "Draft" Warning);
      - a Secondary Sm button "Open event workspace";
    - the task rows, each with a subtle background, 14px radius and 16px padding, made of a 32px round icon (16px tasks icon) and the description (14/22), with a 10px gap between rows.
  - The top nav is the existing `AppShell`.
- **Constraints / assumptions:**
  - No unit-test runner exists; pure helpers are tested by importing them in a Cypress spec (as in T60).
  - The backend has no CORS configuration, so the tests stub `fetch`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Replace the `/my-tasks` placeholder with a `MyTasksPage` in a new `src/features/tasks/` folder:
- data from `GET /api/users/me/tasks` via `useQuery`;
- a pure `groupTasksByEvent` helper builds one group per event, sorted by event start;
- a pure `summarizeTasks` helper computes the three stat counts on the client.

The UI reuses `PageHeader`, `DateBlock`, `Badge`, `Button`, `Icon`, `EmptyState`, `Skeleton` and `ErrorState`. The only new CSS is page-scoped layout classes using the T49 tokens.

Decisions (Gate 1: user approved all seven as proposed, "all as proposed gate 1 approved"):
1. **Which tasks are shown:** all tasks the backend returns, including those of DRAFT and CANCELED events and past events, each event with its status badge. Badges: PUBLISHED → "Published" (Success), DRAFT → "Draft" (Warning), CANCELED → "Canceled" (Danger, same as the existing event badge). The stats count every returned task. The alternative is hiding canceled events.
2. **Grouping and order:** group by `event.id` and sort the groups by `date`, then `startTime`, then event id. Tasks inside a group keep the backend order (task id).
3. **Stats:**
   - Assigned tasks = number of tasks;
   - Events involved = distinct event ids;
   - Organizations = distinct organization ids.
4. **Date and time display:** a small pure formatter that reads `date` and `startTime` as plain local values: month "NOV", day zero-padded "05", meta "Sat, 14 Nov · 18:00". It does **not** reuse `formatEventDate`, because of the UTC shift noted in Context. That bug is reported to the team, not fixed here.
5. **The tasks icon:** download the Figma `icon/tasks` (check-square) SVG to `public/ui-assets/figma-tasks.svg` and map the shared `Icon name="tasks"` to it. Today it is only used in the showcase; this matches the design-system icon. The alternative is a page-only image.
6. **States:**
   - **Loading:** two skeleton cards (date block, title, meta and two task-row blocks) plus skeleton stat tiles.
   - **Empty:** `EmptyState` with the Users icon, "No tasks assigned yet", and the description "When an organizer assigns you a task, it will appear here." Stats are hidden.
   - **Error:** `ErrorState` "We couldn't load your tasks", with "Try again" calling `refetch`.
   - A 401 is handled by the global logout from T50/T55.
7. **"Open event workspace"** navigates to `/org/:organizationId/events/:eventId/tasks`. That route is still a member-guarded placeholder (T72); if the user is no longer a member, the existing T56 guard handles it.

### Steps

1. Create the frontend branch `T64-my-tasks-screen` from up-to-date `master` (after `git status`).
2. Write the tests first:
   - `my-tasks-helpers.cy.ts` (unit);
   - `my-tasks-screen.cy.ts` (e2e, `fetch` stubbed).

   Add the minimum stubs: helpers returning empty results, and `MyTasksPage` rendering the placeholder.
3. Run them and record a behavioral red result.
4. Implement:
   - `src/features/tasks/types.ts`: `MyTask` and `MyTaskEventStatus`.
   - `src/features/tasks/api.ts`: `getMyTasks()`.
   - `src/features/tasks/myTasks.ts`:
     - `groupTasksByEvent(tasks)`;
     - `summarizeTasks(tasks)`;
     - `formatTaskEventDate(date, startTime)`, returning `{ month, day, label }`;
     - `eventStatusBadge(status)`.
   - `src/features/tasks/MyTasksPage.tsx`: the header, stats, event cards and states above. Keep the `page-my-tasks` test id.
   - `public/ui-assets/figma-tasks.svg` (downloaded from the Figma read), and the `tasks` mapping in `LayoutComponents.tsx`.
   - `src/App.tsx`: render `MyTasksPage` on `/my-tasks` inside `RequireAuth` + `AppShell`.
   - `src/styles/global.css`: `.my-tasks-…` classes for the stat tiles, cards, task rows, the 64×72 date block size and the responsive stacking.
5. Run to green, then the full Cypress suite, typecheck, lint and build. Do a visual check against the Figma frame.

### Files to create / modify

- New:
  - `src/features/tasks/{types.ts, api.ts, myTasks.ts, MyTasksPage.tsx}`;
  - `public/ui-assets/figma-tasks.svg`;
  - `cypress/e2e/my-tasks-helpers.cy.ts`, `cypress/e2e/my-tasks-screen.cy.ts`.
- Modify: `src/App.tsx`, `src/shared/ui/LayoutComponents.tsx` (the `tasks` icon source), `src/styles/global.css`.

### Tests to write

- `my-tasks-helpers.cy.ts` (unit):
  - groups tasks by event, sorted by date then start time;
  - keeps task order inside a group;
  - counts tasks, distinct events and distinct organizations;
  - formats the date block and meta label without a timezone shift;
  - maps statuses to badges.
- `my-tasks-screen.cy.ts` (e2e):
  - renders the header, the three stat tiles with the computed numbers, and one card per event in chronological order. Each card has its date block, title, organization, time, status badge and its tasks;
  - calls `GET /api/users/me/tasks`;
  - "Open event workspace" navigates to `/org/:organizationId/events/:eventId/tasks`;
  - shows skeletons while loading;
  - shows the empty state when there are no tasks;
  - shows the error state and retries.
- Regression: the full suite. The existing specs check `page-my-tasks` (T56 guards, T59 redirect) and the nav highlight (T57).

### Risks / open questions

- Decisions 1–7 above need confirmation.
- The UTC shift in `formatEventDate` (Discover) is reported separately; it is not changed in T64.
- T27's known limitation: tasks stay assigned after a user leaves an organization, so they still show here and the workspace button may hit the member guard.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:**
  - `cypress/e2e/my-tasks-helpers.cy.ts` (unit; imports `src/features/tasks/myTasks.ts` directly), 5 tests:
    - `groups tasks by event, sorted by event date then start time`
    - `keeps the backend task order inside a group and orders same-time events by id`
    - `counts tasks, distinct events and distinct organizations`
    - `formats the date block and meta label from the local date and time, without a timezone shift`
    - `maps event statuses to badges`
  - `cypress/e2e/my-tasks-screen.cy.ts` (e2e, `fetch` stubbed, authenticated session), 5 tests:
    - `renders the header, the computed stats and one card per event in chronological order`
    - `opens the event workspace of a card`
    - `shows skeleton cards while loading`
    - `shows the empty state when no tasks are assigned`
    - `shows the error state and retries`
  - Stubs used so the tests compile:
    - `groupTasksByEvent` returns `[]`;
    - `summarizeTasks` returns zeros;
    - `formatTaskEventDate` returns empty strings;
    - `eventStatusBadge` returns `{ label: "", tone: "Neutral" }`;
    - `MyTasksPage` renders the placeholder (`<main data-testid="page-my-tasks"><h1>My tasks</h1></main>`) and is wired into `/my-tasks` inside `RequireAuth` + `AppShell`;
    - `types.ts` and `api.ts` (`getMyTasks`) were added as plain plumbing.
- **Command:** `npx cypress run --spec "cypress/e2e/my-tasks-helpers.cy.ts,cypress/e2e/my-tasks-screen.cy.ts" --browser electron --reporter spec` (Vite on `127.0.0.1:5173`; `npm run typecheck` exit 0).
- **Result:**
  - Unit spec: **0 passing, 5 failing**, with assertion errors on the stubs' empty results, e.g. "expected [] to deeply equal [ 20, 10, 30 ]", "expected { month: '', day: '', label: '' } to deeply equal …", "expected { label: '', tone: 'Neutral' } to deeply equal { label: 'Published', tone: 'Success' }".
  - Screen spec: **0 passing, 5 failing**:
    - "Expected to find content: 'Tasks assigned to you by event organizers.'": the page and its `h1` were found, but the subtitle wasn't;
    - `[data-testid=my-task-group-20]`, `my-tasks-skeleton`, `my-tasks-empty` and `my-tasks-error` were never found.
- **Failure reason:** behavioral. The grouping, counts, date formatting and badges are missing, and `/my-tasks` renders only the placeholder: no stats, cards, workspace button, or loading, empty or error states. There were no compilation errors, and the authenticated session, route and stubs worked.
- **Setup note:**
  - In the first run (right after starting Vite), the first screen test was redirected to `/login`. The other tests reached `/my-tasks` with the placeholder. This is consistent with Vite's dependency optimisation on a cold start reloading the page without the `fetch` stub.
  - Because that failure wasn't caused by missing behaviour, it wasn't counted as red. The suite was rerun unchanged, and the result above comes from that second run.

### Green (after implementation)

- **Result:** the same command gives **5 passing, 0 failing** (`my-tasks-helpers.cy.ts`) and **5 passing, 0 failing** (`my-tasks-screen.cy.ts`).
- **Full suite of the affected repo:** `npx cypress run --browser electron --reporter spec` gives **All specs passed: 92 of 92** (16 specs: the 82 existing tests, including `routing-and-guards` and `login-screen`, which use `page-my-tasks`, and `app-shell-and-navigation`, plus the 10 new ones). `npm run typecheck`, `npm run lint` and `npm run build` all exit 0.
- **After a later cosmetic tweak** (the skeleton stat-tile height went from 94 to 104px, to match the real tile), `npm run typecheck` and `my-tasks-screen.cy.ts` were rerun: 5 passing.
- **Visual check:**
  - Method: a throwaway Cypress spec (deleted afterwards) rendered the page with the Figma sample data at 1440 width, plus the loading and empty states. Its screenshots were compared with the screenshot from the single Figma read.
  - The header, the three stat tiles (5 / 3 / 2), and the three event cards in date order all match the frame. Each card has its date block, title, organization and time meta ("Sat, 14 Nov · 18:00"), Published/Draft badge, "Open event workspace" button and check-square task rows.
  - Measured: date block 64×72, stat icon box 52×52, task icon circle 32×32, button height 34.
  - The skeleton and empty state ("No tasks assigned yet", Users icon) render as planned.
  - The real backend wasn't used, because of the missing backend CORS configuration. Vite was stopped afterwards.

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
  - New feature folder `src/features/tasks/`, containing:
    - `types.ts` (`MyTask` mirrors the T27 `MyTaskResponse`);
    - `api.ts` (`getMyTasks` → `GET /api/users/me/tasks`);
    - `myTasks.ts` (pure helpers);
    - `MyTasksPage.tsx`.

    `/my-tasks` renders it inside `RequireAuth` + `AppShell` and keeps the `page-my-tasks` test id.
  - **Client-side computation:**
    - `groupTasksByEvent` makes one group per event, sorted by date, start time, then event id; tasks keep the backend order;
    - `summarizeTasks` counts tasks, distinct events and distinct organizations.

    All returned tasks are shown and counted, including draft, canceled and past events (Gate 1).
  - **Badges:** Published → Success, Draft → Warning, Canceled → Danger (the same colour as the existing event badge for canceled).
  - **Date display:** `formatTaskEventDate` reads `date` and `startTime` as plain local values (the weekday comes from the date alone), so the time is never shifted by the browser's timezone. The shared `formatEventDate` was not used (see Limitations).
  - **Icon:** the Figma `icon/tasks` (check-square) was downloaded to `public/ui-assets/figma-tasks.svg`, and the shared `Icon name="tasks"` now uses it instead of the placeholder list path (until now it was used only in the components showcase).
  - **States:**
    - `useQuery(["my-tasks"])`: skeleton stat tiles and two skeleton cards while loading;
    - `EmptyState` (Users) "No tasks assigned yet";
    - `ErrorState` "We couldn’t load your tasks", whose retry calls `refetch` and whose error code shows `HTTP_<status>` or `NETWORK_ERROR`.

    The global client retries a 5xx once before the error state appears; a 401 triggers the global logout.
  - **"Open event workspace"** navigates to `/org/:organizationId/events/:eventId/tasks`. That page is still the member-guarded placeholder (T72).
  - **Styling:** page-scoped `.my-tasks-…` classes with the T49 tokens. They reuse `PageHeader`, `DateBlock` (resized only inside `.my-tasks-date`), `Badge`, `Button` (Secondary Sm), `Icon`, `Skeleton`, `EmptyState` and `ErrorState`. Below 900px, the tiles stack and the card header wraps.
- **Alternatives considered:**
  - Hiding canceled or past events: rejected at Gate 1, because the status badge shows the state instead.
  - Reusing `formatEventDate`: it shifts times by the browser offset.
  - A page-only icon image: rejected in favour of fixing the shared design-system icon.
  - Server-side grouping: not needed, because the endpoint already returns everything in a small list.
- **Limitations:**
  - **Out of scope, to report to the team:** `src/shared/lib/formatEventDate.ts` formats in UTC. On Discover (T61/T54), a browser in Bucharest shows an 18:00 event as 16:00.
  - T27's known limitation: tasks stay assigned after the user leaves an organization, so they still appear here, and the workspace button then hits the member guard.
  - The event workspace page itself is still a placeholder until T72.
  - The real backend flow can't be tried in the browser until the backend CORS configuration exists.

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


