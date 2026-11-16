# T74 - Event tasks tab

<!--
Do NOT rename, remove or reorder the section headings below.
Agents locate sections by heading.
Fill metadata as information becomes available. Leave unknown values empty, do not invent them.
-->

## Metadata

- **Task ID:** T74
- **Type:** workflow
- **Owner:** Blaj Horatiu
- **Requirement(s):** 
- **Use case(s):** UC-07
- **Workflow:** 
- **Drive doc / ADR:** (link or section reference)
- **Repo(s):** frontend
- **Branch:** `T74-event-tasks-tab`
- **PR link(s):** Frontend: https://github.com/Vasile07/ubb-events-frontend/pull/28 (master build fix: https://github.com/Vasile07/ubb-events-frontend/pull/27, merged)
- **Tests:** `cypress/e2e/event-tasks-helpers.cy.ts` (new, unit), `cypress/e2e/event-tasks-tab.cy.ts` (new, e2e); regression: `routing-and-guards.cy.ts`, `event-workspace-shell.cy.ts`, `event-actions.cy.ts`, `create-event-flow.cy.ts`, `my-tasks-screen.cy.ts` unchanged

## Description


<!-- What needs to be done and why. Which user need does it address? -->

Source task 27 — Event tasks tab

**Description:** Implement page `Organization Screens`, frame `Org – Event: Tasks` (15:412): table with task description, assignee (avatar, name, role) and an Edit action; header with "Event tasks" description and "Add task" (managers).
Data from `GET /api/organizations/{organizationId}/events/{eventId}/tasks`.
Managers add, edit and remove tasks in a modal using `TextArea` and a `Select` of organization members; members see the list read-only.
States: loading (TableSkeleton), empty (EmptyState with "Add task" for managers) and error. Disable changes on canceled and completed events.
**Depends on:** T72 — Event workspace shell.

## Acceptance criteria


<!-- Proposed from the frontend task list; confirm or edit. -->

- [x] All behavior and UI details listed in the Description are implemented, including stated API interactions and applicable product rules.
- [x] The task's stated dependency prerequisites are merged before implementation.
- [x] Applicable loading, empty, error, validation, and accessibility states listed in the Description are implemented and verified.

## Context

<!-- Filled by the agent at the start: relevant files in repos/, related completed tasks, constraints. Only what is relevant. -->

- **Relevant code** (frontend `master` at `f576b64`, backend `master`):
  - **Backend `EventTaskController`** (T25/T26):
    - `GET /api/organizations/{organizationId}/events/{eventId}/tasks` (member) → `[{ id, description, assignee: { id, firstName, lastName } | null }]`, ordered by id;
    - `POST …/tasks` (manager, 201);
    - `PUT …/tasks/{taskId}` (manager; a `null` `assigneeUserId` unassigns the task);
    - `DELETE …/tasks/{taskId}` (manager, 204).

    The request is `{ description, assigneeUserId }`. The backend answers:
    - 400 `BusinessRuleException` with `errors[]` `description` ("Task description is required") or `assigneeUserId` ("Assignee must be an active member of the organization");
    - 409 "Tasks of a canceled event cannot be changed" / "Tasks of an ended event cannot be changed", where *ended* means `date` + (`endTime` ?? `startTime`) is not after now;
    - 404 when the event or task is not found.
  - **Backend `GET /api/organizations/{organizationId}/members`** (member) → `[{ id, firstName, lastName, email, role }]`, active members only.
  - **Frontend:**
    - `src/App.tsx`: `/org/:organizationId/events/:eventId/tasks` is `eventPage("page-org-event-tasks", "Event tasks")`, a placeholder inside `RequireOrgMember` + `AppShell` + `OrganizationWorkspace` + `EventWorkspace` (T72).
    - `src/features/events/EventWorkspace.tsx` loads the event with query key `["organization-event", organizationId, eventId]` and already shows the "This event is canceled and read-only." banner.
    - `src/features/events/api.ts` / `types.ts`: `OrganizationEvent` (with `status` DRAFT | PUBLISHED | COMPLETED | CANCELED, `date`, `startTime`, `endTime`).
    - `src/shared/api/organizations.ts`: `getOrganizationMembers()`. Its shared type `OrganizationMemberResponse` currently omits `role` and `email`.
    - `src/shared/ui`: `Table`, `TableSkeleton`, `EmptyState`, `ErrorState`, `Modal` (focus trap, Esc), `TextArea`, `Select`, `Button` (Primary/Ghost/Danger, Sm), `Avatar` (36), `InfoBanner`, `useToast` (Success/Error).
    - `useAuth().isManagerOf(orgId)`.
  - **Open frontend PR #25 (T73, colleague)** also touches `App.tsx`, `EventWorkspace.tsx`, `api.ts` and `global.css`, so a later merge may need a small conflict resolution.
- **Related completed tasks:**
  - T25 View event tasks and T26 Edit event and manage tasks (the backend rules above);
  - T72 Event workspace shell;
  - T51/T52/T53 UI components;
  - T64 My tasks (it shows the same tasks from the assignee's side).
- **Figma** (session `08e4f94c-…`, 1 read, frame `Org – Event: Tasks` 15:412). The sidebar, event header and tabs are the T72 shell; T74 covers only the tab content:
  - **Header row:** "Event tasks" (H3 18/26) and the description "Only visible to organization members, never shown on the public event page." (14/22 muted). On the right, a Primary Sm "Add task" button.
  - **Table:** white, 1px border, 20px radius.
    - The head is `--bg-subtle` with the columns "TASK" (flex), "ASSIGNED TO" (260px) and an empty action column (90px), 12/16 semibold muted.
    - Each row has 24px horizontal and 16px vertical padding:
      - the description (14/22 medium, secondary);
      - the assignee: a 36px avatar with initials, the name (14/22 semibold) and the role "Member" / "Manager" (12/16 muted);
      - a Ghost Sm "Edit" button.
  - The modal for add and edit is **not drawn** in the frame. The Description asks for a modal with `TextArea` + `Select`.
- **Constraints / assumptions:** tests stub `fetch`.

## Plan

<!-- Written by the planner, refined by the user. All refinements stay visible here. -->

### Approach

Replace the tasks placeholder with an `EventTasksPage` in `src/features/events/`. It is rendered inside the existing T72 shells.

It loads:
- the tasks (`GET …/tasks`);
- the organization members (`GET …/members`, for the role column and the assignee `Select`);
- the event, from the cached `["organization-event", …]` query, for the editable check.

Managers get "Add task" and "Edit", which open one `Modal` with a `TextArea` description and a `Select` assignee. Edit mode has a "Remove task" action with an inline confirmation. Members see the table read-only.

Decisions (Gate 1: user approved all seven as proposed, "GATE 1 APPROVED"):
1. **Role column:** the role comes from the members list, matched by user id ("Manager" / "Member").
   - If the assignee is no longer in the active members list, show only the name, with no role.
   - An unassigned task shows a muted "Unassigned" with no avatar.
2. **When changes are disabled:** the event is CANCELED or COMPLETED, or it has *ended* (`date` + (`endTime` ?? `startTime`) is in the past, the same rule as the backend). In those cases "Add task", "Edit" and the empty-state action are **hidden**.
   - For ended or completed events, an `InfoBanner` says: "This event has ended, so its tasks can no longer be changed."
   - Canceled events already get the T72 read-only banner.

   If the backend still answers 409 (for example, the event ended while the page was open), show a toast with the backend message and refresh the event and the tasks.
3. **Modal:**
   - The title is "Add task" or "Edit task".
   - The fields:
     - "Description": a `TextArea`, required, trimmed, error "Task description is required";
     - "Assignee": a `Select` whose first option is "Unassigned", then each member as "First Last · Manager/Member".
   - The buttons are "Cancel" and "Save task".
   - While saving, the button shows the loading state.
   - A 400 from the backend maps its `fieldErrors` (`description`, `assigneeUserId`) onto the fields. Any other error shows a toast "Couldn't save the task".
   - On success: a toast "Task added" or "Task updated", the modal closes, and the `tasks` and `my-tasks` queries are refreshed.
4. **Remove:** edit mode has a Danger "Remove task" button. It switches the modal to a confirmation ("Remove this task? This can't be undone." with "Cancel" / "Remove task"), then sends `DELETE`. Success shows a toast "Task removed".
5. **States:**
   - **Loading:** `TableSkeleton`.
   - **Empty:** `EmptyState` (Calendar icon) "No tasks yet". Managers on an editable event see "Add the first task for this event." with an "Add task" action. Members, or a non-editable event, see "No tasks have been added to this event yet." with no action.
   - **Error:** `ErrorState` "We couldn’t load the tasks", with "Try again".
   - A 403 or 404 on the tasks query is already covered by the T72/T58 shells, because the event query fails first.

   If the members query fails, the table still renders without roles, and the modal's `Select` shows "Members unavailable".
6. **Shared type:** add the optional `email` and `role` fields to `OrganizationMemberResponse` in `src/shared/api/types.ts`, matching the backend. The fields are optional, so existing users of the type are unaffected.
7. **Table:** reuse the shared `Table` (`TASK` / `ASSIGNED TO` 260px / action 90px) with page-scoped CSS for the Figma look. For members, the action column is left out.

### Steps

1. Create the frontend branch `T74-event-tasks-tab` from up-to-date `master` (after `git status`).
2. Write the tests first:
   - `event-tasks-helpers.cy.ts` (unit): the editable check and the role lookup;
   - `event-tasks-tab.cy.ts` (e2e, `fetch` stubbed).

   Minimum stubs: the helpers return defaults, and `EventTasksPage` renders the placeholder.
3. Run them and record a behavioral red result.
4. Implement:
   - `src/features/events/eventTasks.ts`: types, `canChangeTasks(event, now)`, `memberRoleLabel`, and the request body builder;
   - api functions in `src/features/events/api.ts`: `getEventTasks`, `createEventTask`, `updateEventTask`, `deleteEventTask`;
   - `src/features/events/EventTasksPage.tsx`, including the modal;
   - the route in `App.tsx`, keeping `page-org-event-tasks`;
   - the type update;
   - page-scoped CSS.
5. Run to green, then the full Cypress suite, typecheck, lint and build. Do a visual check against the frame.

### Files to create / modify

- New:
  - `src/features/events/eventTasks.ts`, `src/features/events/EventTasksPage.tsx`;
  - `cypress/e2e/event-tasks-helpers.cy.ts`, `cypress/e2e/event-tasks-tab.cy.ts`.
- Modify:
  - `src/features/events/api.ts`;
  - `src/shared/api/types.ts`;
  - `src/App.tsx` (the tasks route only);
  - `src/styles/global.css`.

### Tests to write

- `event-tasks-helpers.cy.ts`:
  - `canChangeTasks` is false for CANCELED, COMPLETED and ended events (using the end time, or the start time when there is no end time), and true for upcoming DRAFT and PUBLISHED events;
  - the role lookup finds Manager / Member, and returns nothing for a former member.
- `event-tasks-tab.cy.ts`:
  - **Rendering:**
    - renders the header, description and table rows (description, avatar initials, name, role) for a manager, with "Add task" and "Edit";
    - a member sees the list read-only, with no "Add task" and no "Edit";
    - an unassigned task shows "Unassigned".
  - **Add:**
    - adds a task: the modal sends `POST` with the trimmed description and `assigneeUserId`; then a toast and the refreshed list;
    - shows the description required error without calling the API;
    - maps backend field errors onto the modal fields.
  - **Edit and remove:**
    - edits and reassigns a task, or unassigns it (`PUT` with `assigneeUserId: null`);
    - removes a task after confirmation (`DELETE`);
    - a 409 shows a toast and refreshes.
  - **Disabled changes:** hides the actions and shows the banner for an ended event; hides them for a canceled event.
  - **States:**
    - loading shows `TableSkeleton`;
    - empty for a manager shows "Add task", and empty for a member shows no action;
    - an error state with retry.
- Regression: the full suite, including the T56 route checks on `page-org-event-tasks` and `event-workspace-shell`.

### Risks / open questions

- Decisions 1–7 above need confirmation. The modal layout isn't in Figma; it follows the Description and the existing `Modal`.
- Open PR #25 (T73) touches the same files (`App.tsx`, `api.ts`, `global.css`), so whichever merges second may need a small conflict merge.
- The "ended" check uses the browser's clock against the event's local date and time, while the backend uses `Europe/Bucharest`. The difference matters only for users in other time zones, and the backend 409 still protects the rule.

## Test log

<!-- Written by the implementer. Red-green evidence. -->

### Red (tests written, expected to fail)

- **Tests:**
  - `cypress/e2e/event-tasks-helpers.cy.ts` (unit; imports `src/features/events/eventTasks.ts`), 5 tests:
    - `allows changes for upcoming draft and published events`
    - `blocks changes for canceled and completed events`
    - `blocks changes once the event has ended, using the end time or the start time`
    - `looks up the member role label by user id`
    - `builds the task request with a trimmed description and an optional assignee`
  - `cypress/e2e/event-tasks-tab.cy.ts` (e2e, `fetch` stubbed with a stateful task list), 13 tests:
    - `renders the header and the task table with assignees for a manager`
    - `shows the list read-only for a member`
    - `adds a task through the modal`
    - `requires a description before calling the API`
    - `maps backend field errors onto the modal fields`
    - `edits, reassigns and unassigns a task`
    - `removes a task after confirmation`
    - `shows a toast and refreshes when the backend refuses the change`
    - `hides changes and explains why for an ended event`
    - `hides changes for a canceled event`
    - `shows the table skeleton while loading`
    - `shows the empty state with Add task for a manager and without it for a member`
    - `shows the error state and retries`
  - Stubs used so the tests compile:
    - `canChangeTasks` returns `true`;
    - `memberRoleLabel` returns `undefined`;
    - `toTaskRequest` doesn't trim;
    - `EventTasksPage` renders the placeholder (`<main data-testid="page-org-event-tasks"><h1>Event tasks</h1></main>`), wired into the tasks route inside the T72 shells.

    Plumbing added as is: the `getEventTasks`, `createEventTask`, `updateEventTask` and `deleteEventTask` api functions, and the optional `email`/`role` on `OrganizationMemberResponse`.
- **Command:** `npx cypress run --spec "cypress/e2e/event-tasks-helpers.cy.ts,cypress/e2e/event-tasks-tab.cy.ts" --browser electron --reporter spec` (Vite on `127.0.0.1:5173`, warmed up first; `tsc -b` exit 0).
- **Result:**
  - Helpers: **1 passing, 4 failing**, with "expected true to equal false" (canceled/completed/ended), "expected undefined to equal 'Manager'", and the request not trimmed. The passing test only asserts the "allowed" case, which the `true` stub satisfies by construction.
  - Screen: **0 passing, 13 failing**:
    - "Expected to find content: 'Event tasks' within the selector: 'h2'": the page was found, but not the header;
    - `event-task-row-101`, `event-tasks-add`, `event-task-edit-10x`, `table-skeleton`, `event-tasks-empty` and `event-tasks-error` were never found.
- **Failure reason:** behavioral. The editable rule, the role lookup and the trimming are missing, and the tasks route renders only the placeholder: no table, modal, add/edit/remove, states or disabled rules. There were no compilation errors, and the session, shells and stubs worked.
- **Setup note:** frontend `master` (`27a35b9`) didn't compile because of duplicate imports in `EventWorkspace.tsx`, left by the T58/T73 conflict resolution in `7ff4b9a`.
  - The user chose to fix it in a separate PR: https://github.com/Vasile07/ubb-events-frontend/pull/27 (`fix-eventworkspace-imports`, commit `ef3d32d`).
  - The same two-line fix is applied **uncommitted** in this working tree so the code compiles; it is not part of the T74 commit.

### Green (after implementation)

- **Result:** the same command gives **13 passing, 0 failing** (`event-tasks-tab.cy.ts`) and **6 passing, 0 failing** (`event-tasks-helpers.cy.ts`, with the extra test described below).
- **Regression found and fixed:**
  - The first full-suite run showed 3 new failures:
    - `create-event-flow`: `renders the manager create form and submits one atomic draft request`;
    - `my-tasks-screen`: `opens the event workspace of a card`;
    - `routing-and-guards`: `registers every member workspace route in the route map`.
  - All three failed with "Cannot read properties of undefined (reading 'split')". Those specs stub the event request with data that has no `date`, and `canChangeTasks` called `event.date.split` without a guard.
  - A unit test, `treats an event without a valid date as not editable instead of failing`, was added first and confirmed failing with that same `TypeError`.
  - Then `canChangeTasks` was guarded: an unreadable date means "not editable", and the backend decides anyway.
- **Full suite of the affected repo:** `npx cypress run --browser electron --reporter spec` gives **173 of 175 passing, 28 of 29 specs.**
  - The only failures are the 2 tests in `organizations-list.cy.ts` (T65), which already fail on `origin/master` (assertion on a `hidden` element).
  - The T56 route checks, `event-workspace-shell`, `event-actions`, `create-event-flow` and `my-tasks-screen` all pass.
- `tsc -b`, `npm run lint` and `npm run build` all exit 0.

  This run includes the uncommitted `EventWorkspace.tsx` import fix from PR #27, without which `master` doesn't compile. The user decided the fix stays in PR #27 and is **not** part of the T74 commit.
- **Visual check:**
  - Method: a throwaway Cypress spec (deleted afterwards) took screenshots at 1440 width with the Figma sample data, as a manager.
  - The tab matches the frame:
    - the "Event tasks" heading and its description;
    - the bordered, rounded table with the subtle head and uppercase "TASK" / "ASSIGNED TO" columns;
    - each row with its description and assignee (36px initials avatar, name, Member/Manager role), plus the Ghost "Edit" action.
  - The edit modal shows the Description textarea, the Assignee select ("Radu Moldovan · Member"), "Remove task" (Danger), "Cancel" and "Save task".
  - Vite was stopped afterwards.

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
  - `src/features/events/EventTasksPage.tsx` replaces the tasks placeholder inside the T72 shells (`RequireOrgMember` → `AppShell` → `OrganizationWorkspace` → `EventWorkspace`), and keeps the `page-org-event-tasks` test id.
  - It uses three queries:
    - `["event-tasks", org, event]`;
    - `["organization-members", org]`, shared with the event form;
    - the shell's own `["organization-event", org, event]`, read from the cache for the editable check.
  - **Pure helpers** in `eventTasks.ts`:
    - `canChangeTasks` follows the backend rule: not CANCELED/COMPLETED, and `date` + (`endTime` ?? `startTime`) is after now, in local time; an unreadable date means not editable;
    - `memberRoleLabel` finds the role by user id in the members list;
    - `toTaskRequest` trims the description and turns an empty assignee into `null`.
  - **Permissions:** "Add task", the Edit column and the empty-state action appear only for managers on an editable event. Members, and anyone viewing a canceled, completed or ended event, see the table read-only. An ended or completed event also shows the `InfoBanner` "This event has ended, so its tasks can no longer be changed." (canceled events already show the T72 banner).
  - **Modal** (`Modal` + `TextArea` + `Select`):
    - add and edit share one form, with "Unassigned" plus members as "Name · Role";
    - a description that is empty after trimming shows the error before any request;
    - a 400 `fieldErrors` (`description`, `assigneeUserId`) maps onto the fields;
    - a 409 shows a toast with the backend message, closes the modal and refreshes the event and the tasks;
    - other errors show a toast "Couldn't save the task".
  - **Remove** is in the edit modal, behind a confirmation step ("Remove this task? This can't be undone.").
  - **After any change:** a success toast, and the `event-tasks` and `my-tasks` queries are invalidated.
  - **Table:** the page renders its own `<table>` inside the shared `ui-table-wrap` styles, instead of the shared `Table` component, because that component can't put a key or test id on rows (a small deviation from Gate 1 decision 7: same look, no shared component change). The action header is empty, with `aria-label="Actions"`.
  - **Shared type:** `OrganizationMemberResponse` gained the optional `email` and `role` fields, matching the backend.
  - **API:** `getEventTasks`, `createEventTask`, `updateEventTask` and `deleteEventTask` were added to `src/features/events/api.ts`.
- **Alternatives considered:**
  - Disabling instead of hiding the actions on non-editable events: hiding matches the read-only member view and avoids dead buttons.
  - A separate confirmation dialog for remove: the inline step keeps one modal and its focus trap.
  - Extending the shared `Table` with row props: it would touch a shared component for one page.
  - Fixing master's `EventWorkspace.tsx` duplicate imports inside T74: the user chose a separate PR (#27).
- **Limitations:**
  - **PR #27 (master build fix):** it was merged before the T74 commit. The local copy of the fix was discarded, being identical, and `master` was merged into the T74 branch (`173e3b8`). The T74 commit itself (`9818aad`) doesn't contain the fix.
  - The "ended" check uses the browser's clock against the event's local date and time. The backend uses `Europe/Bucharest`, and its 409 remains the authority.
  - A task assigned to someone who left the organization shows the name without a role, because the members list holds active members only.
  - The modal layout isn't in Figma; it follows the Description and the shared `Modal`.

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


