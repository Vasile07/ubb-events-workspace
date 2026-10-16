# Organization

Represents a student organization (`public.organizations`). The `organization_id` is its identifier.

## Fields

| Field | Type | Constraints |
|---|---|---|
| `organization_id` | `bigint` | Primary key; generated always as identity; not null |
| `name` | `character varying` | Not null |
| `description` | `character varying` | Nullable |
| `logo_url` | `character varying` | Nullable |

## Relationships

- An organization may have many memberships and events.
