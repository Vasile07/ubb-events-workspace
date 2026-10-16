# User

Represents an account in the system (`public.users`).

## Fields

| Field | Type | Constraints |
|---|---|---|
| `user_id` | `bigint` | Primary key; generated always as identity; not null |
| `first_name` | `character varying` | Not null |
| `last_name` | `character varying` | Not null |
| `email` | `character varying` | Not null; unique |
| `password` | `character varying` | Not null; unique |

## Relationships

- A user may have many memberships, registrations, and assigned tasks.
- A user may send many messages.

## Notes

- The schema stores the password in a character-varying field. The application/seed process is expected to store a hashed password; hashing is not expressed as a database constraint.
