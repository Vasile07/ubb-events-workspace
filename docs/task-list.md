# Backend Task List

Backlog for the backend of the UBB student organization events system, based on the high-level requirements and the use cases in `use-cases.md` (UC-01 to UC-18). Each section below can be turned into a task with the `task-creation` skill.

- The **#** column is only a suggested order. Real task IDs are assigned by `task-creation`, and dependencies are written by task name.
- Types: `architecture`, `entity`, `workflow`, `deployment`.
- The scope is the backend only. The frontend will have its own list.
- **Requirements** use the high-level IDs (ORG, EVT, DSC, REG, LIF, ORGZ). Technical tasks that no high-level requirement covers (database, skeleton, error handling, deployment, entity scaffolds) reference the assignment requirements instead, written as **R7, R8, ...**.

## Summary

| # | Task | Type | Use case | Requirements | Depends on |
|---|------|------|----------|--------------|------------|
| 1 | Database creation | architecture | | R7, R8 | none |
| 2 | Backend skeleton | architecture | | R8 | 1 |
| 3 | API conventions and error handling | architecture | | R8, R9 | 2 |
| 4 | Init User entity | entity | | R7 | 1, 2, 3 |
| 5 | Create account | workflow | UC-01 | ORG-01 | 4 |
| 6 | Init Organization entity | entity | | R7 | 1, 2, 3 |
| 7 | Init Membership entity | entity | | R7 | 4, 6 |
| 8 | Sign in | workflow | UC-02, UC-03 | ORG-01 | 5, 7 |
| 9 | Organization-scoped authorization | architecture | | ORG-05, EVT-06, R8 | 7, 8 |
| 10 | View my profile | workflow | none (ORG-01) | ORG-01 | 4, 8 |
| 11 | Update my profile | workflow | none (ORG-01) | ORG-01 | 10 |
| 12 | Request organization membership | workflow | UC-04 | ORG-02 | 4, 6, 8 |
| 13 | View organizations | workflow | UC-04 | ORG-06 | 7, 12 |
| 14 | View organization details | workflow | UC-04 | ORG-06 | 6, 7, 12 |
| 15 | View organization membership requests | workflow | UC-05 | ORG-03, ORG-05 | 7, 9, 12 |
| 16 | Manage organization membership requests | workflow | UC-05 | ORG-03, ORG-05 | 7, 9, 12 |
| 17 | View organization members | workflow | UC-16 | ORG-04, ORG-05 | 7, 9 |
| 18 | Promote organization member to manager | workflow | UC-16 | ORG-04, ORG-05 | 7, 9 |
| 19 | Init Event entity | entity | | R7 | 1, 2, 3, 6 |
| 20 | Init EventTask entity | entity | | R7 | 7, 19 |
| 21 | Init Registration entity | entity | | R7 | 4, 19 |
| 22 | Event lifecycle model | architecture | UC-06, 08, 13, 17 | LIF-01, LIF-04 | 19 |
| 23 | Create an event with tasks | workflow | UC-06 | EVT-01, EVT-06, ORGZ-01, LIF-01 | 7, 9, 19, 20, 22 |
| 24 | View organization events | workflow | none (supports UC-06, UC-07) | EVT-01, EVT-02 | 9, 23 |
| 25 | View event tasks | workflow | UC-07 | ORGZ-02 | 9, 20, 23 |
| 26 | Edit event and manage tasks | workflow | UC-07 | EVT-02, EVT-06, ORGZ-02 | 9, 21, 22, 23 |
| 27 | View my tasks | workflow | none (ORGZ-03) | ORGZ-03 | 8, 20, 23 |
| 28 | Publish event | workflow | UC-08 | EVT-03, EVT-06, LIF-02 | 9, 22, 23 |
| 29 | Discover events | workflow | UC-09 | DSC-01 | 28 |
| 30 | Filter events | workflow | UC-09 | DSC-02, DSC-05 | 29 |
| 31 | Inspect event | workflow | UC-10 | DSC-03, DSC-04, EVT-04 | 21, 28 |
| 32 | Register for event | workflow | UC-11 | REG-01, REG-02, REG-03, REG-06, LIF-02, LIF-03 | 8, 21, 28 |
| 33 | Cancel event registration | workflow | UC-18 | REG-04 | 32 |
| 34 | View my registrations | workflow | UC-18 | REG-04 | 8, 32 |
| 35 | View event registrations | workflow | UC-12 | REG-05, ORG-05 | 9, 32 |
| 36 | Delete event | workflow | UC-17 | EVT-05, EVT-06 | 9, 22, 32 |
| 37 | Init FollowUp entity | entity | | R7 | 19 |
| 38 | Create event follow-up | workflow | UC-13 | LIF-04, LIF-05, ORGZ-06 | 9, 22, 37 |
| 39 | View event follow-up | workflow | none (supports UC-13) | LIF-05 | 9, 37, 38 |
| 40 | Init ChatMessage entity | entity | | R7 | 4, 19 |
| 41 | View event messages | workflow | UC-14 | ORGZ-05, ORG-05 | 9, 40 |
| 42 | Send event message | workflow | UC-14 | ORGZ-05, ORG-05 | 9, 40 |
| 43 | Real-time chat delivery (optional) | workflow | UC-14 | ORGZ-05 | 41, 42 |
| 44 | Init Announcement entity | entity | | R7 | 4, 19 |
| 45 | Publish event announcements | workflow | UC-15 | ORGZ-04 | 9, 44 |
| 46 | View event announcements | workflow | UC-15 | ORGZ-04 | 7, 32, 44 |
| 47 | Containerization and deployment | deployment | | R12 | 1, 2 |

## Requirement coverage

| Requirement | Use cases | Tasks |
|-------------|-----------|-------|
| ORG-01 Account: create, log in, log out, manage details | UC-01, 02, 03 | 5, 8, 10, 11 |
| ORG-02 Request membership | UC-04 | 12 |
| ORG-03 Review requests (managers only) | UC-05 | 15, 16 |
| ORG-04 Promote a member to manager | UC-16 | 17, 18 |
| ORG-05 Restrict organization actions by role | all organization-side use cases | 9 (plus every task under the organization path) |
| ORG-06 View organizations and membership status | UC-04 | 13, 14 |
| EVT-01 Create a draft event | UC-06 | 23, 24 |
| EVT-02 Edit event details | UC-07 | 24, 26 |
| EVT-03 Publish an event | UC-08 | 28 |
| EVT-04 Display event details | UC-10 | 31 |
| EVT-05 Delete an event (managers only) | UC-17 | 36 |
| EVT-06 Block users without permission | UC-06, 07, 08, 17 | 9, 23, 26, 28, 36 |
| DSC-01 Browse published events | UC-09 | 29 |
| DSC-02 Search by title or organization | UC-09 | 30 |
| DSC-03 View event details | UC-10 | 31 |
| DSC-04 Show information to decide | UC-10 | 31 |
| DSC-05 Filter by organization and date | UC-09 | 30 |
| REG-01 Register for a published event | UC-11 | 32 |
| REG-02 Record the registration | UC-11 | 32 |
| REG-03 Prevent duplicate registrations | UC-11 | 32 |
| REG-04 Cancel a registration | UC-18 | 33, 34 |
| REG-05 Members view registrations | UC-12 | 35 |
| REG-06 Enforce registration rules | UC-11 | 32 |
| LIF-01 Event states | UC-06, 08 | 22, 23 |
| LIF-02 Published events open for discovery and registration | UC-08, 11 | 28, 32 |
| LIF-03 Block registration when closed | UC-11 | 32 |
| LIF-04 Mark an event completed | UC-13 | 22, 38 |
| LIF-05 Follow-up after completion | UC-13 | 38, 39 (deviation, see below) |
| ORGZ-01 Create and assign tasks while creating an event | UC-06 | 23 |
| ORGZ-02 Add, edit, reassign, remove tasks while editing | UC-07 | 25, 26 |
| ORGZ-03 Members view their assigned tasks | none | 27 |
| ORGZ-04 Announcement forum | UC-15 | 45, 46 (deviation, see below) |
| ORGZ-05 Organizer conversations | UC-14 | 41, 42, 43 |
| ORGZ-06 Follow-up restricted to the creator | UC-13 | 38 (deviation, see below) |

## Deviations from the high-level requirements

Decided during the review of the task list. Update the requirements in the Drive documentation, or revert the decision:

- **LIF-05 and ORGZ-06:** the requirements say only the manager who created the event can create its follow-up. The team decided that any manager of the organization can, and the event does not record its creator.
- **ORGZ-04:** the requirement says the manager responsible for the event publishes announcements for registered participants. The team decided that members and managers publish them, and that members and managers can also read them.
- **LIF-01:** the requirement lists draft, published and completed. The API also shows CANCELED, for a deleted event (a boolean flag).

## Conventions that apply to all tasks

- **Authentication:** every endpoint requires a signed-in user, except *Create account* and *Sign in*.
- **Entity scaffold ("Init X entity"):** the JPA entity, the repository interface, the service interface and its implementation, and a basic controller. No business rules, which belong to the workflow tasks. The entity follows the specification in `docs/entities/<entity>.md`.
- **Scaffold endpoints:** a workflow task removes or replaces the scaffold endpoints that it makes obsolete, so no unprotected CRUD endpoints stay open. *Organization-scoped authorization* checks what is left.
- **Roles:** a normal user is any signed-in user. Inside an organization there are two roles, MEMBER and MANAGER. Organizations and their first managers are created by an application administrator in SQL.
- **Organization-scoped URLs:** what members and managers do for an organization lives under `/api/organizations/{orgCode}/...` and is protected by the authorization interceptor. Endpoints for normal users live outside that path: the organizations list and details, event discovery and details, registering, cancelling, the user's own profile, registrations and tasks, and reading announcements. The path uses the unique `orgCode` and not the numeric id.
- **Event visibility:** two endpoint sets. The public ones (`/api/events`, `/api/events/{id}`) never return events in the draft status, and never include tasks or registration lists. The organization ones (`/api/organizations/{orgCode}/events...`) are for members and managers and return everything, drafts included.
- **Event status in the API:** DRAFT and PUBLISHED are stored. COMPLETED is derived when the event is read (the end date and time has passed). A deleted event (boolean flag) is returned with the status CANCELED.
- **Resources scoped by organization:** services load organization-owned resources by organization (for example `findByIdAndOrganizationCode`), so a member of one organization cannot touch a resource of another.
- **Unknown organization versus non-member:** both get 403.
- **Lists are not paginated.**
- **API documentation is generated** from the code (springdoc-openapi). The implementer annotates every endpoint (summary, response codes) as part of the workflow task.
- **Test levels:** unit tests for the business rules, and integration tests for repositories and for the controllers of each workflow, with a real database and the real security setup. Each plan names the level of every test and includes at least one API-level test per workflow.
- **Database:** created first (schema script), no migrations. JPA validates the schema and never generates it.

---

## Foundation

### 1. Database creation

- **Type:** architecture. **Requirements:** R7, R8. **Depends on:** none.
- Choose the database and set up a local instance.
- Write the schema script (DDL), with no migration tool.
- Write `docs/entities/<entity>.md` in the workspace for every entity: User, Organization, Membership, MembershipRequest, Event, EventTask, Registration, FollowUp, ChatMessage, Announcement. Each spec lists the fields, types, constraints and relationships.
- Key fields: Organization has a unique, human-readable `orgCode`. Membership has a role (MEMBER or MANAGER). Event has a start and an end date and time, a status (DRAFT or PUBLISHED) and a `deleted` boolean, and no creator. Registration has a status (REGISTERED or CANCELLED), with one row per user and event.
- Seed script for the application administrator: a few organizations, each with a first manager (a user with a hashed password and a MANAGER membership). It is reused by deployment and by the tests.
- Needs a workspace change: a `docs/` folder, readable by the agents (see "Workspace changes needed").

### 2. Backend skeleton

- **Type:** architecture. **Requirements:** R8. **Depends on:** Database creation.
- Gradle project with the layered packages (API, service, persistence).
- Lombok and JPA setup, datasource configuration, profiles.
- springdoc-openapi setup, so the API specification is generated from the code.
- Test setup with a real database (for example Testcontainers or a dedicated test database), working from the first task.

### 3. API conventions and error handling

- **Type:** architecture. **Requirements:** R8, R9. **Depends on:** Backend skeleton.
- Generic exception types (business, not found, conflict, validation, forbidden).
- A global exception handler (`@RestControllerAdvice`) that maps them to HTTP statuses.
- One shared error response format (status, message, list of errors), also used for validation errors and unexpected errors (no stack traces).
- Bean Validation on request DTOs.
- URL, status code and DTO conventions, recorded in `context/project-context.md`.
- The error format appears in the generated OpenAPI document. No pagination.

## Accounts and organizations

### 4. Init User entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Database creation, Backend skeleton, API conventions and error handling.
- Entity scaffold for User. The password is never exposed through the API.

### 5. Create account (UC-01)

- **Type:** workflow. **Requirements:** ORG-01. **Depends on:** Init User entity.
- `POST /api/auth/register`: validation of the required fields (name, email, password), unique email address, password hashing, "email already used".
- Returns 201 with the created user details (never the password).

### 6. Init Organization entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Database creation, Backend skeleton, API conventions and error handling.
- Entity scaffold for Organization, including the unique `orgCode`. Organizations are read-only through the API (they are created in SQL).

### 7. Init Membership entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Init User entity, Init Organization entity.
- Entity scaffold for Membership: user, organization, role (MEMBER or MANAGER), active status.

### 8. Sign in (UC-02, UC-03)

- **Type:** workflow. **Requirements:** ORG-01. **Depends on:** Create account, Init Membership entity.
- `POST /api/auth/login`: JWT-based sign in, with the validation filter that sets the current user.
- The token holds only the user identity and expiry, not the organizations.
- The response includes the user's information and memberships with their roles per organization (there is no global role), for the UI. They are not the source of truth for authorization.
- Same message for any invalid credentials, and a clear response when credentials are missing.
- Log out (UC-03) is client-side only, so there is no endpoint. An expired or invalid token gets a 401.

### 9. Organization-scoped authorization

- **Type:** architecture. **Requirements:** ORG-05, EVT-06, R8. **Depends on:** Init Membership entity, Sign in.
- Role model: MEMBER and MANAGER per organization, with the permissions derived from the role.
- `@RequireOrgPermission` and `@NoOrgCheck` annotations.
- A `HandlerInterceptor` registered on `/api/organizations/{orgCode}/**`: it denies methods without an annotation, reads `{orgCode}` from the URL, checks the membership and the required permission, and answers 403 for an unknown organization and for a non-member.
- A test that fails if any org-scoped controller method has neither annotation.
- A test that a member of one organization cannot access a resource of another.
- The organization is not stored in the JWT. Applied to all existing org-scoped endpoints, and it checks that no scaffold endpoint is left open.

### 10. View my profile

- **Type:** workflow. **Requirements:** ORG-01. **Use case:** none (the requirement says users manage their account details). **Depends on:** Init User entity, Sign in.
- `GET /api/users/me`: user id, name, email and the memberships with their roles. Authentication required, no body.

### 11. Update my profile

- **Type:** workflow. **Requirements:** ORG-01. **Use case:** none. **Depends on:** View my profile.
- `PUT /api/users/me`: update the editable fields. Only the name is editable for now (decided, more fields can be added later). The email is the identifier and cannot be changed. Validation errors use the shared error format.

### 12. Request organization membership (UC-04)

- **Type:** workflow. **Requirements:** ORG-02. **Depends on:** Init User entity, Init Organization entity, Sign in.
- Includes the `MembershipRequest` entity scaffold (entity, repository, service, controller).
- `POST /api/organizations/{orgCode}/membership-requests`, no body (the user comes from the token). The endpoint is under the organization path but marked `@NoOrgCheck`, because the requester is not a member.
- Record a pending request and return 201 with the request id, organization and pending status.
- Prevent a duplicate pending request, and tell a user who is already a member that no request is needed. A declined request does not block a new one.

### 13. View organizations (UC-04)

- **Type:** workflow. **Requirements:** ORG-06. **Depends on:** Init Membership entity, Request organization membership.
- `GET /api/organizations` for any signed-in user, outside the organization path. No pagination.
- Each organization shows its public information (name, description, orgCode) and the user's status: MEMBER, PENDING or NONE (a declined request counts as NONE).
- Never expose private member or request data.

### 14. View organization details (UC-04)

- **Type:** workflow. **Requirements:** ORG-06. **Depends on:** Init Organization entity, Init Membership entity, Request organization membership.
- `GET /api/organizations/{orgCode}` for any signed-in user, marked `@NoOrgCheck`: the public details of one organization and the user's status (MEMBER, PENDING or NONE), so the user can decide before requesting.
- No members, requests or management data.

### 15. View organization membership requests (UC-05)

- **Type:** workflow. **Requirements:** ORG-03, ORG-05. **Depends on:** Init Membership entity, Organization-scoped authorization, Request organization membership.
- `GET /api/organizations/{orgCode}/membership-requests`, optionally filtered by status (pending by default), for managers only.

### 16. Manage organization membership requests (UC-05)

- **Type:** workflow. **Requirements:** ORG-03, ORG-05. **Depends on:** Init Membership entity, Organization-scoped authorization, Request organization membership.
- `PATCH /api/organizations/{orgCode}/membership-requests/{requestId}` with the new status (accepted or declined), for managers only.
- Accepting creates the Membership with the MEMBER role in the same transaction as the decision.
- Only pending requests can be decided, and an already processed request shows its current status.
- The decision is recorded with the reviewer and the time.

### 17. View organization members (UC-16)

- **Type:** workflow. **Requirements:** ORG-04, ORG-05. **Depends on:** Init Membership entity, Organization-scoped authorization.
- `GET /api/organizations/{orgCode}/members`: the members with their roles, for members and managers. Not public data.

### 18. Promote organization member to manager (UC-16)

- **Type:** workflow. **Requirements:** ORG-04, ORG-05. **Depends on:** Init Membership entity, Organization-scoped authorization.
- `PATCH /api/organizations/{orgCode}/members/{userId}/role` with `MANAGER`, for managers only. Promotion only: there is no demotion.
- Reject users who are not members, and inform when the user is already a manager.

## Events

### 19. Init Event entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Database creation, Backend skeleton, API conventions and error handling, Init Organization entity.
- Entity scaffold for Event: start and end date and time, status, `deleted` flag. No creator.

### 20. Init EventTask entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Init Membership entity, Init Event entity.
- Entity scaffold for EventTask: title, description and an optional assignee (a member of the organization). A task has no due date and no status.

### 21. Init Registration entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Init User entity, Init Event entity.
- Entity scaffold for Registration, with a status (REGISTERED or CANCELLED) and one row per user and event.

### 22. Event lifecycle model

- **Type:** architecture. **Requirements:** LIF-01, LIF-04. **Depends on:** Init Event entity.
- Stored status: DRAFT and PUBLISHED. Derived: COMPLETED (now is past the end date and time, computed when the event is read, no scheduler) and CANCELED (the `deleted` flag).
- The allowed transitions and actions per state: edit, publish, register, delete, follow-up, chat and announcements.
- Used by edit, publish, delete, registration and follow-up.

### 23. Create an event with tasks (UC-06)

- **Type:** workflow. **Requirements:** EVT-01, EVT-06, ORGZ-01, LIF-01. **Depends on:** Init Membership entity, Organization-scoped authorization, Init Event entity, Init EventTask entity, Event lifecycle model.
- `POST /api/organizations/{orgCode}/events`, for managers: create an event as a draft with validation (title, description, start and end date and time with the end after the start, location, capacity, registration information). Returns 201 with the event id and the draft status.
- Create written tasks on the event and assign them to members, rejecting a non-member of the organizing organization.

### 24. View organization events

- **Type:** workflow. **Requirements:** EVT-01, EVT-02. **Use case:** none, needed by every organization-side use case. **Depends on:** Organization-scoped authorization, Create an event with tasks.
- `GET /api/organizations/{orgCode}/events` and `.../{eventId}`, for members and managers, returning all events of the organization including drafts and CANCELED ones.
- The event data only: the task list is a separate endpoint.

### 25. View event tasks (UC-07)

- **Type:** workflow. **Requirements:** ORGZ-02. **Depends on:** Organization-scoped authorization, Init EventTask entity, Create an event with tasks.
- `GET /api/organizations/{orgCode}/events/{eventId}/tasks`, for members and managers: the tasks with their descriptions and assignees. Verify that the event belongs to the organization.
- Never returned by the public event endpoints.

### 26. Edit event and manage tasks (UC-07)

- **Type:** workflow. **Requirements:** EVT-02, EVT-06, ORGZ-02. **Depends on:** Organization-scoped authorization, Init Registration entity, Event lifecycle model, Create an event with tasks.
- `PUT /api/organizations/{orgCode}/events/{eventId}`: edit the event details with permission and state checks and validation. Capacity cannot drop below the active registrations. A canceled event cannot be edited. Any manager of the organization can edit.
- Add, edit, reassign and remove tasks under `.../events/{eventId}/tasks` and `.../tasks/{taskId}`, rejecting a non-member assignee.

### 27. View my tasks

- **Type:** workflow. **Requirements:** ORGZ-03. **Use case:** none. **Depends on:** Sign in, Init EventTask entity, Create an event with tasks.
- `GET /api/users/me/tasks`: only the tasks assigned to the current user, with the event and organization they belong to. Outside the organization path.

### 28. Publish event (UC-08)

- **Type:** workflow. **Requirements:** EVT-03, EVT-06, LIF-02. **Depends on:** Organization-scoped authorization, Event lifecycle model, Create an event with tasks.
- `PATCH /api/organizations/{orgCode}/events/{eventId}/publish`, for managers: validate the required details (including the end date and time), change the status from DRAFT to PUBLISHED, and return the updated event.
- Reject incomplete events with what to correct, and reject an event that is already published or canceled.

### 29. Discover events (UC-09)

- **Type:** workflow. **Requirements:** DSC-01. **Depends on:** Publish event.
- `GET /api/events`: the list of events that are not drafts, with summary information. Completed events are included with the status COMPLETED, and deleted ones that had been published appear with the status CANCELED.
- "No results" returns an empty list. No pagination.

### 30. Filter events (UC-09)

- **Type:** workflow. **Requirements:** DSC-02, DSC-05. **Depends on:** Discover events.
- Extend the list endpoint with a search term (event title or organization name), an organization filter and a date range filter (start and end date).
- Removing the criteria returns the full list.

### 31. Inspect event (UC-10)

- **Type:** workflow. **Requirements:** DSC-03, DSC-04, EVT-04. **Depends on:** Init Registration entity, Publish event.
- `GET /api/events/{eventId}`: title, description, organization, start and end date and time, location, registration information, status, whether registration is available, and the user's `registrationStatus` (NOT_REGISTERED, REGISTERED or CANCELLED).
- No task assignments and no participant lists in this response.
- Draft events and unknown ids return 404 for normal users. A deleted event is returned with the status CANCELED.

### 32. Register for event (UC-11)

- **Type:** workflow. **Requirements:** REG-01, REG-02, REG-03, REG-06, LIF-02, LIF-03. **Depends on:** Sign in, Init Registration entity, Publish event.
- `POST /api/events/{eventId}/registrations`, no body: check that the event is published and not completed or canceled, registration is open, the user is eligible and capacity remains (also under concurrent requests). Returns 201 with the registration details.
- One registration row per user per event: a duplicate shows the current status (409), and a user who cancelled is registered again by reactivating the row.
- Full event or closed registration returns 409 with the reason.

### 33. Cancel event registration (UC-18)

- **Type:** workflow. **Requirements:** REG-04. **Depends on:** Register for event.
- `DELETE /api/events/{eventId}/registrations/me`: cancel the user's own active registration until the event starts, freeing a place. Returns 204.
- Respond clearly when there is no registration or the event has started.

### 34. View my registrations (UC-18)

- **Type:** workflow. **Requirements:** REG-04. **Depends on:** Sign in, Register for event.
- `GET /api/users/me/registrations`: only the current user's own registrations (active and cancelled) with the public details of each event. Never other participants' data.

### 35. View event registrations (UC-12)

- **Type:** workflow. **Requirements:** REG-05, ORG-05. **Depends on:** Organization-scoped authorization, Register for event.
- `GET /api/organizations/{orgCode}/events/{eventId}/registrations`: the registered users of an event, for members and managers of the organizing organization (403 for others). Verify that the event belongs to the organization.
- Cancelled registrations are not shown as registered. Empty list case. Visible fields follow the privacy policy. No export.

### 36. Delete event (UC-17)

- **Type:** workflow. **Requirements:** EVT-05, EVT-06. **Depends on:** Organization-scoped authorization, Event lifecycle model, Register for event.
- `DELETE /api/organizations/{orgCode}/events/{eventId}`, for managers: a soft delete (set the `deleted` flag), refused for an event that has already ended, with the reason. Returns 204.
- The event is returned with the status CANCELED by the public endpoints (unless it was a draft) and by the organization endpoints. It cannot be edited, published or registered for, and its chat and announcements become read-only.

### 37. Init FollowUp entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Init Event entity.
- Entity scaffold for FollowUp. Fields to confirm: summary and feedback (the team has not decided them yet).

### 38. Create event follow-up (UC-13)

- **Type:** workflow. **Requirements:** LIF-04, LIF-05, ORGZ-06. **Depends on:** Organization-scoped authorization, Event lifecycle model, Init FollowUp entity.
- `POST /api/organizations/{orgCode}/events/{eventId}/follow-up`: any manager of the organization can create it, only when the event is completed (end date and time has passed) and not canceled. Deviates from LIF-05 and ORGZ-06, which say the creator only.
- Reject with the reason when the event is not completed or required information is missing.

### 39. View event follow-up

- **Type:** workflow. **Requirements:** LIF-05. **Use case:** none, supports UC-13. **Depends on:** Organization-scoped authorization, Init FollowUp entity, Create event follow-up.
- `GET /api/organizations/{orgCode}/events/{eventId}/follow-up`: the saved follow-up, for members and managers of the organization. Returns 404 when there is none.

## Communication

### 40. Init ChatMessage entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Init User entity, Init Event entity.
- Entity scaffold for ChatMessage (event, author, text, time). There is no Conversation entity: the conversation of an event is the set of its messages.

### 41. View event messages (UC-14)

- **Type:** workflow. **Requirements:** ORGZ-05, ORG-05. **Depends on:** Organization-scoped authorization, Init ChatMessage entity.
- `GET /api/organizations/{orgCode}/events/{eventId}/messages`: the messages of the event, for members and managers. No messages returns an empty list, and others get 403.
- Available from the draft state. Not paginated.

### 42. Send event message (UC-14)

- **Type:** workflow. **Requirements:** ORGZ-05, ORG-05. **Depends on:** Organization-scoped authorization, Init ChatMessage entity.
- `POST` on the same path: the author comes from the token, the text cannot be empty and has a maximum length. Returns 201 with the message, author and time.
- Refused for a canceled event (read-only).

### 43. Real-time chat delivery (UC-14, optional)

- **Type:** workflow. **Requirements:** ORGZ-05. **Depends on:** View event messages, Send event message.
- WebSocket delivery on top of the REST chat. Only if time allows.

### 44. Init Announcement entity

- **Type:** entity. **Requirements:** R7. **Depends on:** Init User entity, Init Event entity.
- Entity scaffold for Announcement (event, author, title, content, date and time).

### 45. Publish event announcements (UC-15)

- **Type:** workflow. **Requirements:** ORGZ-04. **Depends on:** Organization-scoped authorization, Init Announcement entity.
- `POST /api/organizations/{orgCode}/events/{eventId}/announcements` with a title and content, for members and managers. Returns 201. Refused for a canceled event.
- Deviates from ORGZ-04, which says the responsible manager publishes. UC-15 itself only describes reading.

### 46. View event announcements (UC-15)

- **Type:** workflow. **Requirements:** ORGZ-04. **Depends on:** Init Membership entity, Register for event, Init Announcement entity.
- `GET /api/events/{eventId}/announcements`, outside the organization path. The service allows registered participants and members or managers of the organizing organization, and denies everyone else.
- A cancelled registration loses access. "No announcements" returns an empty list. Each announcement shows content, author and creation date.

## Operations

### 47. Containerization and deployment

- **Type:** deployment. **Requirements:** R12. **Depends on:** Database creation, Backend skeleton.
- Dockerfile, Compose file with the database, configuration through environment variables, instructions to run and demonstrate the system outside the development machine.
- Runs the seed script (organizations and first managers) for the demo.

---

## Considered and not added

From the endpoint table:

- **Logout endpoint:** log out is client-side only.
- **Generic role change:** only promotion to manager exists. Demotion is in no requirement.
- **Mark an event completed:** completion is derived from the end date and time.
- **Separate integration test and API documentation tasks:** tests belong to every workflow task, and the OpenAPI document is generated.
- **Pagination parameters.**
- **Separate endpoints for creating, updating and deleting a task:** they stay inside *Create an event with tasks* and *Edit event and manage tasks*, as decided. Only the task list view has its own task.
- **Registration rules as a separate item:** they are the business rules inside *Register for event*.

## Decisions taken

- Organizations and first managers are created in SQL by an application administrator. No creation endpoint, no admin role in the API.
- Unknown organization and non-member both get 403.
- Log out is client-side only.
- An event is completed when its end date and time has passed (derived, no scheduler).
- Soft delete is a boolean flag. A deleted event is shown with the status CANCELED to everyone who may see it. Normal users never see drafts.
- A deleted event is allowed unless the event has already ended.
- Any manager of the organization can edit, publish and delete its events and create their follow-up. No creator is recorded.
- Registration can be cancelled until the event starts, and a user can register again (the row is reactivated).
- Announcements are published by members and managers, and read by registered participants and by members.
- Lists are not paginated.
- The path uses `orgCode`, not the numeric organization id.
- Conversation entity removed: one conversation per event, as the messages of the event.
- The entity scaffolds for Registration, FollowUp, ChatMessage and Announcement stay separate.
- The member list is visible to members and managers.
- The follow-up can be read by members and managers of the organization.
- Tasks have a title, a description and an optional assignee only: no due date and no status.
- The profile has only the name as an editable field for now.
- Completed events appear in the public event list with the status COMPLETED.

## Still open

- The fields of the follow-up (assumed: summary and feedback).
- More editable profile fields, if the team wants them later.
- Everything in the "Open decisions" rows of `use-cases.md`.

## Workspace changes needed

- Add `docs/entities/` to the workspace, and let the *Database creation* task write there (today `AGENTS.md` says agents don't write documentation).
- Add the `entity` task type everywhere the types are listed.
- Add the test levels, the API annotation rule and the scaffold endpoint rule to `project-context.md` and the implementer.