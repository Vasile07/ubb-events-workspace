# UBB Events – Frontend Task List

Figma file key: `wbXB4CAMaecbLaKUd4slWq`
Pages: `Design System`, `Student Screens`, `Organization Screens`.
How to read the design: use the Figma MCP (`get_design_context` / `get_screenshot`) with the file key and the node id given in each task. The Design System page holds tokens (frame `Foundations`, 4:40) and all components (section `Components`, 2:2).
Backend base path: `/api` (backend task list / generated OpenAPI is the source of truth for DTOs).
Product rules that apply to every task:
- There is NO registration deadline: registration is open until the event starts. Never show or build a deadline field, tile or text.
- There is no "registered at" detail on the participant side.
- Capacity is shown only in the registration card on the event details page.
- Organizations are identified only by `organizationId`. There is no organization code/slug: never show, build or search by a code.
- A user has two separate fields: `firstName` and `lastName`. Every form and every request uses both. For display, use a derived `fullName` (`firstName + " " + lastName`).
- Password rule (registration only): minimum 8 characters, at least 1 lowercase letter, 1 uppercase letter, 1 number and 1 special character.
- Event statuses: DRAFT, PUBLISHED, COMPLETED, CANCELED (deleted = canceled). Registration statuses: NOT_REGISTERED, REGISTERED, CANCELLED. Organization statuses for a user: MEMBER, PENDING, NONE. Roles: MEMBER, MANAGER.
Tasks are ordered by dependency; "Depends on" tasks must be merged first.

---

## 1. Project setup
**Description:** Create the frontend with React + TypeScript + Vite (change if another stack is chosen). Add React Router, TanStack Query, ESLint, Prettier and the folder structure by feature: `features/auth`, `features/events`, `features/organizations`, `features/workspace`, `shared/ui`, `shared/api`, `shared/lib`. Configure path aliases, strict TypeScript, `.env` files (`VITE_API_URL`) and a dev proxy to the backend. Add scripts `dev`, `build`, `lint`, `typecheck` and a README with run instructions. Done when all scripts pass on an empty app shell with a placeholder home route.

## 2. Design tokens and global styles
**Description:** Implement the design tokens so no component hardcodes colors, sizes or fonts.
Figma: page `Design System`, frame `Foundations` (4:40) and the variable collections `Primitives`, `Semantic`, `Spacing & Radius` (read them through `get_variable_defs`).
Include:
- the blue palette (#223A5E, #355982, #4D79A8, #6E9ECC, #9FC0E3, #D0E1F2, #F3F8FD)
- semantic tokens (bg/page, bg/surface, bg/brand, text/primary|secondary|muted|inverse, border/default|strong|focus, accent, status success/warning/danger fg+bg)
- spacing (4–48)
- radii (8, 12, 16, 24, pill)
- shadows `Shadow/Card` and `Shadow/Raised`
- font Plus Jakarta Sans (400–800) with the 11 text styles (Display/Hero, Heading H1–H3, Body Large/Default/Medium/Strong, Caption Default/Strong, Label/Button).

Add a global reset and set the page background to `bg/page`.
Done when a demo page side by side with the Foundations frame is visually identical.

## 3. API client and error handling
**Description:** Build the HTTP layer in `shared/api` and a small `shared/lib/tokenStorage` module (`getToken`, `setToken`, `clearToken`, backed by localStorage).
The API client:
- prefixes the base URL, reads the token from `tokenStorage` and adds the `Authorization: Bearer` header, serializes JSON
- parses the backend's shared error format (status, message, list of errors) into a typed `ApiError` exposing `status`, `message` and `fieldErrors` (map field path → message). Field paths must support nested/indexed paths such as `tasks[1].description` so forms can show errors on the exact row
- exposes `setUnauthorizedHandler(handler)`. The client calls the handler on any 401, except for requests to the login and register endpoints (wrong credentials must not trigger a logout). The default handler is a no-op: the client must NOT import anything from `features/auth` or the router. The auth layer registers the handler in task 8, so the dependency goes 8 → 3 only and there is no circular dependency.

Generate or hand-write TypeScript DTO types from the OpenAPI spec for every endpoint used in the app.
Configure TanStack Query defaults (no retry on 4xx, 1 retry on network errors, sensible stale time).
Add tests for error parsing (including nested field paths) and for the 401 handler behavior.
Done when `GET /api/organizations` is called through the client, a forced 400 shows up as a typed `ApiError` and a forced 401 calls a test handler.
**Depends on:** 1.

## 4. Base UI components: actions and form fields
**Description:** Implement the reusable components from page `Design System`, section `Components` (2:2):
- `Button` (variants Primary, Secondary, Ghost, Danger; sizes Md 44px and Sm 34px; node 2:20) with `loading` and `disabled` states and optional leading icon
- `Badge` (tones Neutral, Info, Success, Warning, Danger, Brand with the leading dot; node 2:40)
- `TextField` (states Default, Focus, Error; label, helper/error text; node 2:57) plus a `TextArea` variant with the same look and configurable height
- `Select` styled like TextField with a chevron
- `SearchField` (pill, search icon; node 2:59)
- `FilterChip` (Default/Active with chevron; node 2:72)

All must forward refs, support keyboard focus (focus ring uses `border/focus`), `aria-invalid` and `aria-describedby`. Add a demo page (or Storybook) showing every variant.
**Depends on:** 2.

## 5. Base UI components: layout and data display
**Description:** Implement the shared building blocks used by many screens:
- `AuthLayout` (split layout used by Login and Create account: left brand panel with gradient, headline, three feature rows and decorative circles; right form panel; frames `Login` 13:10 and `Create account` 13:62). Both auth screens use it, so they stay independent from each other
- `Tabs` in two styles (pill/segmented as in frame `My registrations` 14:355, and underline as in the event workspace frame 15:412)
- `Table` (header row, rows, bottom dividers, fixed and fill columns, as in `Org – Events` 15:10)
- `Modal` (centered card, `Shadow/Raised`, dark backdrop as in `Org – Delete event confirmation` 16:1061, focus trap, Esc to close)
- `InfoBanner` (info and warning tones)
- `Avatar` (initials from first and last name, sizes 36/48/80)
- `ProgressBar` (capacity)
- `DateBlock` (month + day tile)
- `InfoTile` (icon box + label + value, as in `Event details` 13:417)
- `PageHeader` (title, subtitle, right slot)
- `Icon` wrapper (calendar, pin, clock, users, building, bell, check, x, plus, send, lock, trash, edit, info, alert, chevrons, arrow-left, search, ticket, megaphone, tasks, shield, logout).

**Depends on:** 2, 4.

## 6. Feedback components: toast, empty state, skeleton, error state
**Description:** Implement the components that cover non-happy-path states.
- `Toast` (Success and Error, icon circle, title, message, close; Design System node 46:30) with a `ToastProvider` and `useToast()` that stacks toasts top-right, auto-dismisses after ~5 s and is announced via `aria-live`.
- `EmptyState` (icon variants Search, Calendar, Ticket, Users; title, description, optional primary action; node 46:68).
- `Skeleton` primitives (rounded `bg/muted` blocks) plus ready-made `EventCardSkeleton` and `TableSkeleton` (see frames `Discover events – loading` 48:587 and `Org – Events (loading)` 49:1175).
- `ErrorState` (danger icon, title, description, "Try again" button, error code caption; see frame `Discover events – error` 48:757) and a React `ErrorBoundary` that renders it.

Done when each component has a demo. Every screen task below must implement its own loading, empty and error states with these components.
**Depends on:** 4, 5.

## 7. Event card and event status logic
**Description:** Build `EventCard` from page `Design System`, component `Event card` (3:301): gradient cover with date chip and status badge, organization name row, title, location row, capacity progress bar and "X of Y spots taken", and the "View details" action.
Create the pure function `getEventBadge(event, registrationStatus)` returning label and tone: Open (Success), Few spots left (Warning, when ≤10% or ≤5 places remain), Full (Danger), Registered (Info), Completed (Info), Canceled (Danger).
Cover precedence rules in unit tests (e.g. Canceled beats Full; Registered beats Open).
Format dates as "Sat, 14 Nov · 18:00" with a shared `formatEventDate` helper.
**Depends on:** 4, 5.

## 8. Authentication context provider
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
**Depends on:** 1, 3, 6.

## 9. Routing and route guards
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

## 10. App shell and top navigation
**Description:** Build the authenticated layout with the `Navbar` from page `Design System` (3:180): logo, links Discover / My registrations / My tasks / Organizations / Workspace, notification bell and the user chip (avatar initials + `fullName`) linking to `/profile`.
The active link follows the current route (the `None` variant is used on the profile page).
The Workspace link must appear only when the user has at least one membership: with one organization it goes to `/org/:organizationId/events`, with several it opens a small picker.
The bell can stay non-functional (no notifications in scope) but must render.
Content is centered with 80px horizontal padding on desktop.
**Depends on:** 5, 8, 9.

## 11. Error pages: 404 and 403
**Description:** Implement Figma page `Student Screens`, frames `404 – Page not found` (48:876) and `403 – No access` (48:914): large faded code, title, description, decorative circles and two buttons (Back to discover, Go back / My organizations).
Use them as the router's catch-all 404 and as the target of `RequireOrgManager`/`RequireOrgMember` failures and API 403 responses on page-level queries.
Keep the navbar visible when the user is authenticated.
**Depends on:** 6, 9, 10.

## 12. Login screen
**Description:** Implement page `Student Screens`, frame `Login` (13:10) using `AuthLayout`: email, password, Sign in, link to Create account (`/register`, a plain route link).
Client validation: both fields required, valid email format. The password is only checked as required: the complexity rules apply to registration only (do not block existing users or reveal the policy on login).
Call `login`; show backend errors (invalid credentials) as the Error state on the form and a toast for network failures.
After success redirect to the originally requested route or `/events`.
Disable the button with a loading state while submitting.
**Depends on:** 4, 5, 8, 9.

## 13. Create account screen
**Description:** Implement page `Student Screens`, frame `Create account` (13:62) using `AuthLayout`: **First name** and **Last name** side by side, email, password (helper "Min. 8 characters with upper & lower case, a number and a symbol"), the "Create account" button and a link to sign in (`/login`; it is only a route path, the login page does not need to exist yet).
Request: `POST /api/auth/register` with `firstName`, `lastName`, `email`, `password`.
Client validation:
- first name, last name and email required (trimmed), valid email format
- password: minimum 8 characters, at least 1 lowercase letter, 1 uppercase letter, 1 number and 1 special character. Implement it as a pure `validatePassword` helper returning the list of unmet rules (unit tested) and show which rules are missing in the field's error text.

Show the Error state on the email field for "email already registered" (409, as drawn in the frame), map other validation errors from `ApiError.fieldErrors` to the matching fields (including `firstName`, `lastName`, `password`) and use a toast for network errors.
On success sign the user in (or redirect to `/login`) and go to `/events`. Disable the button with a loading state while submitting.
**Depends on:** 4, 5, 8, 9.

## 14. Discover events screen
**Description:** Implement page `Student Screens`, frame `Discover events` (13:119): header with result count, `SearchField`, chips (Organization, Date range, "Open for registration" toggle) with "Clear filters", 3-column grid of `EventCard` and "Load more events".
Data from `GET /api/events` (non-drafts; completed and canceled included with their status); the Organization chip lists organizations from `GET /api/organizations` and filters by `organizationId`. If the API has no search/filter params do the filtering client-side and mention it in the PR. Keep filters in the URL query string.
States: loading → frame `Discover events – loading` (48:587), no results → `Discover events – no results` (48:694, EmptyState Search with "Clear filters"), error → `Discover events – error` (48:757, ErrorState + error toast).
Cards link to `/events/:eventId`.
**Depends on:** 6, 7, 10.

## 15. Event details screen (participant)
**Description:** Implement page `Student Screens`, frames `Event details` (13:417) and `Event details – registered` (13:538) as one route with two states.
Left: hero (status badge, organization name, title), info tiles for Date & time and Location only, "About this event" with the registration information block.
Right: registration card (places left, progress bar, "X of Y places taken", action) and the organizer card. Capacity appears only in the right card; there is no deadline and no registered-on date.
Actions:
- Register `POST /api/events/{id}/registrations`
- Cancel registration `DELETE /api/events/{id}/registrations/me` after a confirmation modal

Both update the UI optimistically and invalidate the event, my-registrations and discover queries; show success/error toasts.
Handle 409 (already registered, show the current status), full, started, completed and canceled events by disabling the button with an explanatory text.
When registered, also render the Announcements card from `GET /api/events/{id}/announcements` (hide on 403, empty state when none).
**Depends on:** 5, 6, 7, 10.

## 16. My registrations screen
**Description:** Implement page `Student Screens`, frame `My registrations` (14:355): pill tabs Active / Canceled / Past with counts, info banner and rows (date block, title, organization, location, date/time, status badge, actions).
Data from `GET /api/users/me/registrations` (only the user's own, active and cancelled).
Actions:
- View event
- Cancel registration (confirmation, then `DELETE`) on active rows
- Register again on canceled rows whose event has not started (re-register reactivates the row)
- no actions on completed events

Empty state per tab, see frame `My registrations – empty` (48:829, EmptyState Ticket with "Discover events"). Loading and error states.
**Depends on:** 6, 15.

## 17. My tasks screen
**Description:** Implement page `Student Screens`, frame `My tasks` (14:547): three stat tiles (assigned tasks, events involved, organizations) computed on the client, then one card per event with date block, title, organization, status badge, "Open event workspace" button and the list of the user's tasks.
Data from `GET /api/users/me/tasks`; group by event and sort by event start.
The workspace button goes to `/org/:organizationId/events/:eventId/tasks`.
Add loading (skeleton cards), an empty state ("No tasks assigned yet", Users or Calendar icon) and an error state.
**Depends on:** 6, 10.

## 18. Organizations list screen
**Description:** Implement page `Student Screens`, frame `Organizations` (14:699): header, search field and a 3-column grid of organization cards (icon tile, status badge, name, description, action).
Data from `GET /api/organizations` including the user's status.
Actions:
- MEMBER → "Open organization" (to `/org/:organizationId/events`)
- PENDING → disabled "Waiting for approval"
- NONE → "Request membership" calling `POST /api/organizations/{organizationId}/membership-requests` (no body), then switch the card to Pending without a reload and show a toast; handle an already-pending 409 gracefully.

The search filters by organization name only, on the client. Equal-height cards with the action pinned to the bottom. Loading, empty ("No organizations match") and error states.
**Depends on:** 6, 10.

## 19. Organization details screen
**Description:** Implement page `Student Screens`, frame `Organization details` (14:833): gradient hero with logo tile, name, description, status badge and the membership action (same logic and mutation as task 18), "Upcoming events" as two-column `EventCard` grid (events from `GET /api/events` for this organization, non-completed) and the "About" card explaining that member lists are private.
Data from `GET /api/organizations/{organizationId}` (route `/organizations/:organizationId`). Show the 404 page for an unknown id and an empty state when there are no upcoming events. No organization code is shown anywhere.
**Depends on:** 7, 18.

## 20. Profile screen
**Description:** Implement page `Student Screens`, frame `Profile` (14:978): account card with avatar (initials from first and last name), **First name** and **Last name** fields side by side, read-only email with helper "Your email is your sign-in identifier and can't be changed", Save changes / Cancel; right column with "My organizations" (name + role badge per membership) and a Log out button (calls `logout()`).
Data from `GET /api/users/me` (returns `firstName` and `lastName`), save with `PUT /api/users/me` sending both fields; the email is never sent.
Validate that both names are required and trimmed, disable Save until the form is dirty and valid, show backend field errors on the matching input, refresh the auth context after saving so the navbar chip and avatars update, and show a success toast. Cancel resets the form to the saved values.
**Depends on:** 8, 10.

## 21. Organization workspace shell
**Description:** Build the workspace layout used by every frame on page `Organization Screens`: the `Workspace sidebar` from page `Design System` (3:299) with the organization header (icon, name, category, role badge Manager/Member), items Events / Membership requests (with pending count badge) / Members and "Back to discover", next to a scrollable main area (48px horizontal, 40px vertical padding).
Load the organization from the route `/org/:organizationId` via `GET /api/organizations/{organizationId}` and the role from the auth context. Hide "Membership requests" for plain members.
The pending count comes from `GET /api/organizations/{organizationId}/membership-requests` and refreshes after accept/decline. The sidebar stretches to the full height of the content.
**Depends on:** 9, 10.

## 22. Organization events dashboard
**Description:** Implement page `Organization Screens`, frame `Org – Events` (15:10): header with "Create event" (managers only), pill tabs All / Drafts / Published / Completed / Canceled with counts, and the table (title + location, date, status badge, registrations "x / capacity" or "—" for drafts, action).
Data from `GET /api/organizations/{organizationId}/events` (includes drafts and canceled).
Row actions:
- Manage (to the event's Tasks tab)
- "Add follow-up" for completed events (managers)
- View for canceled

States:
- loading → frame `Org – Events (loading)` (49:1175, TableSkeleton)
- empty → `Org – Events (empty)` (49:1081, EmptyState Calendar with "Create event" for managers)
- error → ErrorState.

**Depends on:** 5, 6, 21.

## 23. Event form component
**Description:** Build the reusable `EventForm` from page `Organization Screens`, frames `Org – Create event` (15:202) and `Org – Edit event` (49:740): sections "Event details" (title, description textarea, location), "Schedule & capacity" (start date, start time, end date, end time, capacity, registration information) and, in create mode, "Tasks" (rows of description + assignee select + remove button + "Add another task").
Right column in create mode: Event lifecycle card and "Before you publish" checklist; in edit mode: the information card and current status card.
Validation: required title, description, location, start and end; end after start (Error state on End time with "End must be after the start"); capacity ≥ 1; in edit mode capacity cannot be below the active registrations (show the backend message as shown in the edit frame).
The assignee select lists members from `GET /api/organizations/{organizationId}/members`.
Backend errors: the form accepts `fieldErrors` with nested paths (e.g. `tasks[1].description`, `tasks[2].assigneeId`) and shows each message on the exact task row, plus a short error summary at the top of the form listing everything that must be corrected. Form values must never be lost after a failed submit.
Props: `mode`, `initialValues`, `onSubmit`, `submitting`, `fieldErrors`.
**Depends on:** 4, 5, 21.

## 24. Create event flow
**Description:** Wire `EventForm` (create mode) to `/org/:organizationId/events/new` (managers only).
On submit call `POST /api/organizations/{organizationId}/events` once, sending the event data together with the list of tasks. The endpoint is atomic: either the event and all its tasks are created, or nothing is created and the backend returns the list of errors. There is no partial state and no retry mechanism: on error keep the form as it is, show every returned error on the matching field (event fields and task rows, using task 23), let the user correct them and submit again.
Buttons: "Save as draft" (primary) and Cancel (confirmation modal when the form is dirty).
On success the event is a DRAFT: show a toast and redirect to the event workspace (Tasks tab) where it can be published.
**Depends on:** 23.

## 25. Event workspace shell
**Description:** Build the event workspace header and tab bar shared by the frames `Org – Event: Tasks` (15:412), `Registrations` (15:589), `Organizer chat` (15:822), `Announcements` (16:431) and `Follow-up` (16:591): back link to events, title with status badge, meta row (date and time range, location, number of places) and the underline tabs Tasks / Registrations / Organizer chat / Announcements / Follow-up that map to the nested routes.
Data from `GET /api/organizations/{organizationId}/events/{eventId}` (verify the event belongs to the organization, otherwise 404).
The actions slot is filled by task 26. For canceled events show a read-only banner; for completed events keep Follow-up enabled.
**Depends on:** 5, 21.

## 26. Event actions: edit, publish and delete
**Description:** Add the manager actions to the event workspace header.
- Edit event → route `/org/:organizationId/events/:eventId/edit` reusing `EventForm` in edit mode (frame `Org – Edit event` 49:740) with `PUT .../events/{eventId}`; disable it for completed/canceled events.
- Publish (DRAFT only): primary button "Publish event" next to Edit and Delete as in frame `Org – Event: Tasks (draft)` (49:897) with the draft warning banner; clicking opens the confirmation modal from frame `Org – Publish event confirmation` (49:1055) and calls `PATCH .../events/{eventId}/publish`; show backend validation messages (missing end date etc.) in the modal.
- Delete event: Danger button opening the modal from frame `Org – Delete event confirmation` (16:1061) that shows the number of affected participants and calls `DELETE .../events/{eventId}` (soft delete; the backend refuses ended events, show that message).

Invalidate event, events list and discover queries and show toasts. Hide all of these for plain members.
**Depends on:** 23, 25.

## 27. Event tasks tab
**Description:** Implement page `Organization Screens`, frame `Org – Event: Tasks` (15:412): table with task description, assignee (avatar, name, role) and an Edit action; header with "Event tasks" description and "Add task" (managers).
Data from `GET /api/organizations/{organizationId}/events/{eventId}/tasks`.
Managers add, edit and remove tasks in a modal using `TextArea` and a `Select` of organization members; members see the list read-only.
States: loading (TableSkeleton), empty (EmptyState with "Add task" for managers) and error. Disable changes on canceled and completed events.
**Depends on:** 25.

## 28. Event registrations tab
**Description:** Implement page `Organization Screens`, frame `Org – Event: Registrations` (15:589): three stat tiles (active registrations / capacity, places remaining, canceled), the info banner and the participants table (avatar, full name, email, registered at, status).
Data from `GET /api/organizations/{organizationId}/events/{eventId}/registrations` (members and managers; a 403 shows the 403 page). Canceled registrations are not listed but are counted in the tile.
If the backend does not return a registration timestamp, remove that column and tell the planner.
Add search by name/email on the client, an empty state ("No registrations yet", Users icon) and loading/error states.
**Depends on:** 25.

## 29. Event organizer chat tab
**Description:** Implement page `Organization Screens`, frame `Org – Event: Organizer chat` (15:822): chat card with header ("Organizer conversation", lock note "Not visible to participants"), message list (other users on the left with avatar, name and time; the current user's messages on the right in the brand color) and a composer with Send.
Data from `GET` and `POST .../events/{eventId}/messages`.
Poll every ~5 s (`refetchInterval`) and keep the scroll at the bottom unless the user scrolled up; trim and validate text (not empty, backend max length with a counter near the limit); Enter sends, Shift+Enter adds a line.
Disable the composer for canceled events and show an empty state when there are no messages. Keep the data layer isolated so polling can be swapped for WebSocket later.
**Depends on:** 25.

## 30. Event announcements tab
**Description:** Implement page `Organization Screens`, frame `Org – Event: Announcements` (16:431): list of announcements (avatar, author with role, time, title, content, newest first) and the "New announcement" composer card (title, content, Publish).
Create with `POST .../events/{eventId}/announcements` (members and managers), read with `GET /api/events/{eventId}/announcements`.
Validate required title and content, refresh the list and clear the form after publishing, show a toast, hide the composer for canceled events and show an empty state when there are none. Reuse the `AnnouncementItem` component built for the participant view in task 15.
**Depends on:** 15, 25.

## 31. Event follow-up tab
**Description:** Implement page `Organization Screens`, frame `Org – Event: Follow-up` (16:591): info banner, form with Summary and Feedback textareas and "Save follow-up", plus the "Event recap" card (registered participants, tasks, announcements counts, computed from existing queries).
Read with `GET .../events/{eventId}/follow-up` (404 = none yet, show the empty form) and create with `POST` on the same path (any manager of the organization, only after the event has ended).
Before the event ends show the form in a locked state with an explanation; plain members see the saved follow-up read-only or an empty state. After saving switch to a read view with an "editing is not available" note (no update endpoint) and a success toast.
**Depends on:** 25.

## 32. Membership requests screen
**Description:** Implement page `Organization Screens`, frame `Org – Membership requests` (16:734): pill tabs Pending / Accepted / Declined with counts, table (applicant avatar, full name and email, requested date, status badge, Accept and Decline buttons on pending rows) and the info banner.
Data from `GET /api/organizations/{organizationId}/membership-requests?status=...`; actions with `PATCH .../membership-requests/{requestId}` (accepted | declined) with per-row loading state.
After an action update the row/tab counts, the sidebar pending badge and the members query, and show a toast.
Managers only (guard from task 9). Empty state per tab ("No pending requests") and error state.
**Depends on:** 21.

## 33. Organization members screen
**Description:** Implement page `Organization Screens`, frame `Org – Members` (16:887): header with the member-count badge, info banner about promotion, and the table (avatar, full name, email, role badge Manager = Brand / Member = Neutral, "Promote to manager" on member rows for managers only).
Data from `GET .../members`; promote with `PATCH .../members/{userId}/role` sending `MANAGER`.
Promotion is permanent: the confirmation modal must say it cannot be undone and there is no demote action. Plain members see the list without actions.
Loading, empty and error states; invalidate the auth context memberships if the current user is affected.
**Depends on:** 21.

## 34. Build and deployment configuration
**Description:** Add the production build, a multi-stage Dockerfile (Node build, Nginx serve with SPA history fallback and `/api` reverse proxy), environment-specific configuration, a docker-compose example that runs the frontend with the backend, and a GitHub Actions workflow that builds and publishes the image. Document the required environment variables and the deployment steps in the README.
**Depends on:** 33.