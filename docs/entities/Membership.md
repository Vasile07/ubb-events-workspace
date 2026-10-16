# Membership

Represents a user's relationship with an organization (`public.memberships`). Membership requests are represented through membership status rather than a separate entity.

## Fields

| Field | Type | Constraints |
|---|---|---|
| `membership_id` | `bigint` | Primary key; generated always as identity; not null |
| `user_id` | `bigint` | Not null; foreign key to `users.user_id` |
| `organization_id` | `bigint` | Not null; foreign key to `organizations.organization_id` |
| `role` | `character varying` | Nullable; intended values: `MEMBER`, `MANAGER` |
| `status` | `character varying` | Nullable; intended values: `PENDING` (awaiting approval) and `ACTIVE` (active membership) |

## Relationships

- Each membership belongs to one user and one organization.
- A user and an organization may each be related to many membership rows; the supplied schema does not define a uniqueness constraint on `(user_id, organization_id)`.
