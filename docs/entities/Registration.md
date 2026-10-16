# Registration

Represents a user's registration for an event (`public.registrations`).

## Fields

| Field | Type | Constraints |
|---|---|---|
| `registration_id` | `bigint` | Primary key; generated always as identity; not null |
| `event_id` | `bigint` | Not null; foreign key to `events.event_id` |
| `user_id` | `bigint` | Not null; foreign key to `users.user_id` |

## Relationships

- Each registration belongs to one event and one user.
- The supplied schema does not define a unique constraint on `(event_id, user_id)` or a registration status field.
