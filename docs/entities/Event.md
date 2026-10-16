# Event

Represents an event hosted by an organization (`public.events`).

## Fields

| Field | Type | Constraints |
|---|---|---|
| `event_id` | `bigint` | Primary key; generated always as identity; not null |
| `organization_id` | `bigint` | Not null; foreign key to `organizations.organization_id` |
| `category_id` | `bigint` | Not null; foreign key to `categories.category_id` |
| `title` | `character varying` | Not null |
| `description` | `character varying` | Not null |
| `date` | `date` | Not null |
| `start_time` | `time without time zone` | Not null |
| `end_time` | `time without time zone` | Nullable |
| `location` | `character varying` | Nullable |
| `capacity` | `bigint` | Nullable |
| `status` | `character varying` | Not null; intended values: `DRAFT`, `PUBLISHED`, `CANCELED` (`CANCELED` represents a deleted/canceled event) |

## Relationships

- Each event belongs to one organization and one category.
- An event may have many registrations, announcements, messages, follow-ups, and tasks.
