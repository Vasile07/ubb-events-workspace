# Follow-up

Represents follow-up content associated with an event (`public.followups`).

## Fields

| Field | Type | Constraints |
|---|---|---|
| `followup_id` | `bigint` | Primary key; generated always as identity; not null |
| `event_id` | `bigint` | Not null; foreign key to `events.event_id` |
| `content` | `character varying` | Not null |

## Relationships

- Each follow-up belongs to one event; an event may have many follow-ups.
