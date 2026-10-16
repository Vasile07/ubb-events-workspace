# Message

Represents a message sent by a user in an event (`public.messages`). This is the entity referred to as `ChatMessage` in the original task description.

## Fields

| Field | Type | Constraints |
|---|---|---|
| `message_id` | `bigint` | Primary key; generated always as identity; not null |
| `event_id` | `bigint` | Not null; foreign key to `events.event_id` |
| `sender_id` | `bigint` | Not null; foreign key to `users.user_id` |
| `content` | `character varying` | Not null |

## Relationships

- Each message belongs to one event and has one sender (a user).
- An event and a user may each be associated with many messages.
