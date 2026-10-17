# Announcement

Represents an announcement associated with an event (`public.announcements`).

## Fields

| Field | Type | Constraints |
|---|---|---|
| `announcement_id` | `bigint` | Primary key; generated always as identity; not null |
| `event_id` | `bigint` | Not null; foreign key to `events.event_id` |
| `title` | `character varying` | Not null |
| `content` | `character varying` | Not null |

## Relationships

- Each announcement belongs to one event; an event may have many announcements.
