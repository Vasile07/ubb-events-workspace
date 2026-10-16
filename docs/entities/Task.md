# Task

Represents work associated with an event (`public.tasks`).

## Fields

| Field | Type | Constraints |
|---|---|---|
| `task_id` | `bigint` | Primary key; generated always as identity; not null |
| `description` | `character varying` | Not null |
| `assigned_user_id` | `bigint` | Nullable; foreign key to `users.user_id` |
| `event_id` | `bigint` | Not null; foreign key to `events.event_id` |

## Relationships

- Each task belongs to one event.
- A task may be assigned to one user; a user may be assigned many tasks.
- An event may have many tasks.
