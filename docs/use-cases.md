# Use Cases: UBB Student Organization Events (backend scope)

Source: the team's updated use case list (UC-01 to UC-18). This file replaces the earlier version, which was based on the previous list.

**How to read this file**

- The use cases keep the team's wording, with small cleanups. Rows marked *(assumed)* are standard behavior that the source does not state. Confirm or remove them.
- "Open decisions" and "Gaps" list what the team still has to decide. They are good candidates for the Drive documentation and for ADRs.
- Differences from the earlier version:
  - New: Log out (UC-03), Promote organization member to manager (UC-16), Delete event (UC-17), Cancel event registration (UC-18).
  - UC-14 is now organizer-related conversations of an organization, and UC-15 is read-only event announcements for registered participants. The earlier "organization news forum for everybody" is no longer in the list.
  - There is no longer a "Manage event execution" use case, and no "organization not accepting requests" case in UC-04.

## Actors and roles

| Actor | Description |
|-------|-------------|
| Normal User | Any user with an account. Can discover events, register, and ask to join organizations |
| Organization Member | A user whose membership in an organization was accepted (role MEMBER) |
| Organization Manager | A member with the manager role (role MANAGER). Manages events, requests and members of the organization |
| Registered participant | A normal user with a registration for a specific event |

## Overview

| ID | Use case | Primary actor | Requirements |
|----|----------|---------------|--------------|
| UC-01 | Create account | Normal User | R2, R7 |
| UC-02 | Log in | Normal User | R2 |
| UC-03 | Log out | Normal User | R2 |
| UC-04 | Request organization membership | Normal User | R2 |
| UC-05 | Review organization membership requests | Organization Manager | R2 |
| UC-06 | Create event | Organization Manager | R1, R4, R5 |
| UC-07 | Edit event and manage assigned tasks | Organization Manager | R1, R5 |
| UC-08 | Publish event | Organization Manager | R1, R4, R6 |
| UC-09 | Discover, search and filter events | Normal User | R3, R6 |
| UC-10 | View event details | Normal User | R3, R6 |
| UC-11 | Register for event | Normal User | R3 |
| UC-12 | View event registrations | Organization Member | R5 |
| UC-13 | Create event follow-up | Organization Manager | R4, R5 |
| UC-14 | Participate in organizer conversations | Organization Member | R2, R5 |
| UC-15 | View event announcements | Normal User (registered) | R5, R6 |
| UC-16 | Promote organization member to manager | Organization Manager | R2 |
| UC-17 | Delete event | Organization Manager | R1, R4 |
| UC-18 | Cancel event registration | Normal User | R3 |

---

## UC-01 Create account

| Field | Description |
|-------|-------------|
| Primary actor | Normal User |
| Description | Allows a new user to create an account in the system |
| Preconditions | The user does not already have an account.<br>The user has access to the system |
| Postconditions | A new user account is created |
| Main flow | 1. The user opens the registration page.<br>2. The user enters the required account information.<br>3. The user submits the registration form.<br>4. The system validates the information.<br>5. The system creates the account.<br>6. The system confirms that the account was created |
| Alternative flows | A. Required information is missing or invalid: the system displays an error and requests corrections.<br>B. The email address is already associated with an account: the system informs the user and prevents a duplicate account |
| Business rules | The email address is the unique account identifier.<br>Passwords are never stored in plain text *(assumed)* |
| Open decisions | Required fields. Whether only UBB email addresses are accepted. Email verification |
| Requirements | R2, R7 |

## UC-02 Log in

| Field | Description |
|-------|-------------|
| Primary actor | Normal User |
| Description | Allows an existing user to authenticate and access the system |
| Preconditions | The user has an existing account.<br>The user is not currently authenticated |
| Postconditions | The user is authenticated and can access the system according to their permissions |
| Main flow | 1. The user opens the login page.<br>2. The user enters the login credentials.<br>3. The user submits the login form.<br>4. The system validates the credentials.<br>5. The system authenticates the user.<br>6. The system grants access to the functionality permitted by the user's role |
| Alternative flows | A. The credentials are incorrect: the system displays an error and lets the user try again.<br>B. Required credentials are missing: the system requests them |
| Business rules | Authentication uses a JWT.<br>The failure message does not reveal which credential was wrong *(assumed)*.<br>The user's organization roles are determined per organization |
| Open decisions | Token lifetime. Lockout after failed attempts |
| Requirements | R2 |

## UC-03 Log out

| Field | Description |
|-------|-------------|
| Primary actor | Normal User |
| Description | Allows an authenticated user to end their current session |
| Preconditions | The user is authenticated |
| Postconditions | The user's authenticated session is terminated |
| Main flow | 1. The user selects the logout option.<br>2. The system terminates the authenticated session.<br>3. The system redirects the user to the login page or another public page.<br>4. The system confirms that the user is logged out, if applicable |
| Alternative flows | A. The session has already expired: the system redirects the user to the login page |
| Business rules | With a JWT, the session is ended when the client discards the token. An expired or invalid token gets a 401 response, which the frontend turns into the redirect to the login page |
| Open decisions | Whether the server also invalidates the token on logout (a denylist or short-lived tokens with refresh tokens) or logout is client-side only |
| Requirements | R2 |

## UC-04 Request organization membership

| Field | Description |
|-------|-------------|
| Primary actor | Normal User |
| Description | Allows a normal user to request membership in a student organization |
| Preconditions | The user is authenticated.<br>The user is not already a member of the selected organization.<br>The user has no pending request for that organization |
| Postconditions | The request is stored and awaits a decision from an organization manager |
| Main flow | 1. The user browses the available organizations.<br>2. The user selects an organization.<br>3. The user submits a membership request.<br>4. The system records the request with a pending status.<br>5. The system makes the request available to the organization's manager |
| Alternative flows | A. The user already has a pending request: the system prevents a duplicate request.<br>B. The user is already a member: the system informs them that another request is unnecessary |
| Business rules | One membership or pending request per user per organization.<br>The user can see the status of their own requests *(assumed, replaces notifications)* |
| Open decisions | Whether a declined user may apply again. Which information a request contains |
| Gaps | No use case creates or manages organizations, although users browse them here. Added as a supporting backend task |
| Requirements | R2 |

## UC-05 Review organization membership requests

| Field | Description |
|-------|-------------|
| Primary actor | Organization Manager |
| Description | Allows a manager to accept or decline membership requests for their organization |
| Preconditions | The user is authenticated as a manager of the organization.<br>At least one request exists for the organization |
| Postconditions | The request is accepted or declined, and the user's membership is updated when applicable |
| Main flow | 1. The manager opens the membership requests section.<br>2. The system displays the pending requests.<br>3. The manager selects a request.<br>4. The manager accepts or declines it.<br>5. The system updates the request status.<br>6. If accepted, the system grants the user organization member status |
| Alternative flows | A. The request was already processed: the system displays its current status.<br>B. The user is not a manager of that organization: the system denies access |
| Business rules | Only a manager of that organization can decide.<br>Only a pending request can be decided.<br>Acceptance creates a membership with the MEMBER role.<br>A declined request grants no access.<br>The decision is recorded with the reviewer and the time *(assumed)* |
| Open decisions | Whether a decline needs a reason |
| Requirements | R2 |

## UC-06 Create event

| Field | Description |
|-------|-------------|
| Primary actor | Organization Manager |
| Description | Allows a manager to create an event, save it as a draft, and assign written tasks to organization members during creation |
| Preconditions | The user is authenticated as a manager.<br>The manager belongs to the organization organizing the event |
| Postconditions | The event is saved as a draft, together with any tasks and assignments |
| Main flow | 1. The manager opens the event creation form.<br>2. The manager enters the title, description, date, time, location and other relevant details.<br>3. The manager optionally creates written tasks associated with the event.<br>4. The manager assigns tasks to organization members, if necessary.<br>5. The manager saves the event as a draft.<br>6. The system validates and stores the event details, tasks and assignments.<br>7. The system confirms that the event was created |
| Alternative flows | A. Required details are missing or invalid: the system requests corrections.<br>B. A task is assigned to someone who is not a member of the organizing organization: the system rejects the assignment.<br>C. The user is not a manager: the system denies event creation |
| Business rules | A new event starts as a draft, visible only to the members of its organization *(assumed)*.<br>The event belongs to one organization and records its creator (needed by UC-13).<br>Tasks can only be assigned to members of the organizing organization.<br>Capacity is a positive number *(assumed)* |
| Open decisions | Required fields for a draft versus for publication. Date rules (must be in the future?). Registration settings (opening and closing dates, cancellation deadline) |
| Requirements | R1, R4, R5 |

## UC-07 Edit event and manage assigned tasks

| Field | Description |
|-------|-------------|
| Primary actor | Organization Manager |
| Description | Allows a manager to edit event information and add, edit, reassign or remove written tasks |
| Preconditions | The event exists.<br>The user is a manager authorized to manage the event |
| Postconditions | The event details and task assignments reflect the saved changes |
| Main flow | 1. The manager opens an event they are authorized to manage.<br>2. The system displays the current event details and its tasks.<br>3. The manager updates the event information as necessary.<br>4. The manager creates new tasks or edits, reassigns or removes existing tasks.<br>5. The manager saves the changes.<br>6. The system validates and stores the updated information and assignments.<br>7. The system confirms that the changes were saved |
| Alternative flows | A. The updated information is invalid: the system requests corrections.<br>B. A task is assigned to a non-member of the organization: the system rejects the assignment.<br>C. The user lacks the required permissions: the system denies the modification |
| Business rules | Permission is checked for the event's organization.<br>Which edits are allowed depends on the event state.<br>Capacity cannot drop below the number of existing registrations *(assumed)*.<br>Reassigning a task is allowed |
| Open decisions | Editable fields per state (draft, published, completed). Whether any manager of the organization can edit, or only the creator. Whether registered participants are notified of changes |
| Requirements | R1, R5 |

## UC-08 Publish event

| Field | Description |
|-------|-------------|
| Primary actor | Organization Manager |
| Description | Allows a manager to publish an event so that it becomes available for discovery and, where applicable, registration |
| Preconditions | The event exists as a draft.<br>The user is a manager authorized to manage the event.<br>The required event details are complete |
| Postconditions | The event is published and available for discovery |
| Main flow | 1. The manager opens the draft event.<br>2. The manager reviews its information.<br>3. The manager selects the publish action.<br>4. The system validates the required details.<br>5. The system changes the event status to published.<br>6. The event becomes available to users according to its registration settings |
| Alternative flows | A. Required information is missing or invalid: the system prevents publication and indicates what to correct.<br>B. The user lacks the required permissions: the system denies publication |
| Business rules | Only a draft can be published.<br>Publication requires complete required details.<br>A published event is visible to all users |
| Open decisions | Visibility policy (all users, UBB only?). Whether an event can be unpublished |
| Requirements | R1, R4, R6 |

## UC-09 Discover, search and filter events

| Field | Description |
|-------|-------------|
| Primary actor | Normal User |
| Description | Allows users to browse published events, search by event title or organization name, and filter by organization or date |
| Preconditions | The user can access the event discovery section |
| Postconditions | The user sees a list of events matching the criteria |
| Main flow | 1. The user opens the event discovery section.<br>2. The system displays the available published events.<br>3. The user optionally enters an event title or organization name.<br>4. The user optionally selects an organization filter or a date filter.<br>5. The system applies the search criteria and filters.<br>6. The system displays the matching events.<br>7. The user can select an event to view its details |
| Alternative flows | A. No events match: the system displays an appropriate message.<br>B. The user removes the criteria or filters: the results update accordingly |
| Business rules | Only published events are returned (drafts and deleted events never).<br>Search matches the event title or the organization name.<br>Filters: organizing organization and date |
| Open decisions | Whether discovery requires being signed in. Sorting and the page size |
| Requirements | R3, R6 |

## UC-10 View event details

| Field | Description |
|-------|-------------|
| Primary actor | Normal User |
| Description | Allows users to view detailed information about an event before deciding whether to register |
| Preconditions | The event exists and is available for viewing |
| Postconditions | The user has viewed the event information |
| Main flow | 1. The user selects an event from the list.<br>2. The system displays the title, description, organizing organization, date, time, location and registration information.<br>3. The system indicates whether registration is available.<br>4. The user can proceed to register if eligible |
| Alternative flows | A. The event is no longer available: the system informs the user.<br>B. Registration is closed or unavailable: the system displays the status |
| Business rules | Availability reflects the registration status and the remaining capacity |
| Open decisions | None beyond the visibility rules of UC-08 |
| Requirements | R3, R6 |

## UC-11 Register for event

| Field | Description |
|-------|-------------|
| Primary actor | Normal User |
| Description | Allows a normal user to register for a published event that is accepting registrations |
| Preconditions | The user is authenticated.<br>The event is published and accepting registrations.<br>The user has not already registered |
| Postconditions | The user is registered and the registration is recorded |
| Main flow | 1. The user opens the event details.<br>2. The user selects the registration option.<br>3. The system verifies eligibility and the registration rules.<br>4. The system records the registration.<br>5. The system confirms the registration |
| Alternative flows | A. The user is already registered: the system prevents a duplicate.<br>B. Registration is closed, capacity is reached, or another condition is not met: the system prevents registration and explains why |
| Business rules | One active registration per user per event.<br>The registrations never exceed the capacity, including under concurrent requests *(assumed)*.<br>Registration is possible only while it is open |
| Open decisions | Eligibility rules. Whether there is a waiting list. Whether a user can register again after cancelling (UC-18) |
| Requirements | R3 |

## UC-12 View event registrations

| Field | Description |
|-------|-------------|
| Primary actor | Organization Member |
| Description | Allows an organization member to view the users registered for events organized by their organization |
| Preconditions | The user is an organization member.<br>The organization has an event with registration information |
| Postconditions | The member can view the registrations for the selected event |
| Main flow | 1. The member opens their organization's event section.<br>2. The member selects an event.<br>3. The member opens the registration list.<br>4. The system displays the users registered for the event |
| Alternative flows | A. No users registered: the system displays an empty list or a message.<br>B. The user does not belong to the organizing organization: the system denies access |
| Business rules | Any member of the organizing organization can view the list (not only managers).<br>Cancelled registrations are not shown as registered *(assumed)* |
| Open decisions | Which participant fields are visible. Whether an export is allowed |
| Requirements | R5 |

## UC-13 Create event follow-up

| Field | Description |
|-------|-------------|
| Primary actor | Organization Manager |
| Description | Allows the manager who created an event to create its follow-up after the event has been completed |
| Preconditions | The event has been marked as completed.<br>The authenticated manager is the creator of the event |
| Postconditions | The follow-up is saved and associated with the completed event |
| Main flow | 1. The manager opens the completed event they created.<br>2. The system verifies that the event is completed and that the manager is its creator.<br>3. The manager selects the follow-up option.<br>4. The manager enters the follow-up information.<br>5. The manager saves the follow-up.<br>6. The system stores the follow-up and confirms the action |
| Alternative flows | A. The event is not completed: the system prevents follow-up creation.<br>B. The manager did not create the event: the system denies the action.<br>C. Required follow-up information is missing: the system requests corrections |
| Business rules | Only the creator of the event can create the follow-up.<br>The event must be in the completed state.<br>One follow-up per event *(assumed)* |
| Open decisions | What follow-up information is recorded. Who can read it |
| Gaps | No use case marks an event as completed. Decide whether it happens automatically after the event date or through a manager action |
| Requirements | R4, R5 |

## UC-14 Participate in organizer conversations

| Field | Description |
|-------|-------------|
| Primary actor | Organization Member |
| Description | Allows organization members to access organizer-related conversations associated with their organization's events |
| Preconditions | The user is an organization member.<br>An organizer-related conversation is available to that member |
| Postconditions | The member has accessed or participated in an authorized conversation |
| Main flow | 1. The member opens the relevant event or organization section.<br>2. The system displays the available conversations.<br>3. The member selects a conversation.<br>4. The system displays its content.<br>5. The member participates according to the conversation's permissions |
| Alternative flows | A. No conversation is available: the system displays a message.<br>B. The user does not have access: the system denies access |
| Business rules | Only active members of the organization can access its conversations.<br>Messages are persisted with author and time *(assumed)* |
| Open decisions | How conversations are created: one general conversation per organization and one per event, created automatically *(assumed, to confirm)*, or created by members. Real-time delivery versus polling. Editing and deleting messages. Retention policy when a membership ends |
| Requirements | R2, R5 |

## UC-15 View event announcements

| Field | Description |
|-------|-------------|
| Primary actor | Normal User (registered participant) |
| Description | Allows registered participants to view event announcements and updates published by the manager responsible for the event. It replaces the participant chat and is for event news only |
| Preconditions | The event exists.<br>The user is registered for the event |
| Postconditions | The user has viewed the available announcements |
| Main flow | 1. The user opens an event for which they are registered.<br>2. The user opens the announcement forum.<br>3. The system displays the announcements published for the event.<br>4. The user selects an announcement to view its full content, if applicable |
| Alternative flows | A. No announcements: the system displays a message.<br>B. The user is not registered for the event: the system denies access |
| Business rules | Only registered participants can read the announcements of an event (a cancelled registration loses access *(assumed)*).<br>Announcements are read-only for participants |
| Open decisions | Whether members of the organizing organization can also read them *(assumed yes)*. Whether announcements can be edited or deleted |
| Gaps | No use case publishes an announcement. The description says the responsible manager publishes it, so a supporting backend task is added |
| Requirements | R5, R6 |

## UC-16 Promote organization member to manager

| Field | Description |
|-------|-------------|
| Primary actor | Organization Manager |
| Description | Allows a manager to promote an existing member of their organization to manager |
| Preconditions | The user is a manager.<br>The selected user is already a member of the same organization |
| Postconditions | The selected member has manager permissions for that organization |
| Main flow | 1. The manager opens the organization's member list.<br>2. The manager selects a member.<br>3. The manager selects the option to promote the member.<br>4. The system verifies the manager's permissions and the selected user's membership.<br>5. The system updates the user's role.<br>6. The system confirms the promotion |
| Alternative flows | A. The selected user is not a member: the system rejects the action.<br>B. The requesting user is not a manager: the system denies the action.<br>C. The selected user is already a manager: the system informs the requesting manager |
| Business rules | Roles per organization: MEMBER and MANAGER.<br>A promotion applies to one organization only |
| Open decisions | Whether a manager can be demoted or removed (not in any use case). Whether an organization must always keep at least one manager |
| Requirements | R2 |

## UC-17 Delete event

| Field | Description |
|-------|-------------|
| Primary actor | Organization Manager |
| Description | Allows a manager to delete an event they are authorized to manage, subject to the event state and registration constraints |
| Preconditions | The event exists.<br>The user is a manager authorized to manage the event |
| Postconditions | The event is deleted or marked as deleted, if deletion is permitted |
| Main flow | 1. The manager opens the event.<br>2. The manager selects the delete option.<br>3. The system requests confirmation.<br>4. The manager confirms.<br>5. The system checks whether deletion is permitted under the event's state and constraints.<br>6. The system deletes the event or marks it as deleted, according to the deletion policy.<br>7. The system confirms the result |
| Alternative flows | A. The manager cancels the confirmation: the event is unchanged.<br>B. Deletion is not permitted because of the event's state or registrations: the system prevents it and explains why.<br>C. The user lacks the required permissions: the system denies the action |
| Business rules | A deleted event is no longer discoverable or editable.<br>Deletion is allowed or refused depending on the event state and its registrations.<br>The confirmation step is handled by the client |
| Open decisions | Deletion policy: soft delete *(recommended)* or hard delete. Which states and registration conditions forbid deletion. What happens to tasks, registrations, follow-up and announcements of a deleted event |
| Requirements | R1, R4 |

## UC-18 Cancel event registration

| Field | Description |
|-------|-------------|
| Primary actor | Normal User |
| Description | Allows a registered user to cancel their registration for an event when cancellation is permitted |
| Preconditions | The user is authenticated.<br>The user is registered for the event.<br>The event's registration rules permit cancellation |
| Postconditions | The registration is cancelled and the registration list is updated |
| Main flow | 1. The user opens the event or their registered events list.<br>2. The user selects the event.<br>3. The user chooses the cancellation option.<br>4. The system verifies that the user has a registration and that cancellation is permitted.<br>5. The system cancels the registration.<br>6. The system confirms the cancellation |
| Alternative flows | A. The user is not registered: the system informs them that no registration exists.<br>B. The cancellation deadline has passed or cancellation is prohibited: the system prevents it and explains why |
| Business rules | Cancelling frees a place in the event's capacity *(assumed)*.<br>The user can see their own registrations *(assumed, needed for the registered events list)* |
| Open decisions | The cancellation rule (deadline field on the event, or until the event starts). Whether the user can register again after cancelling |
| Requirements | R3 |