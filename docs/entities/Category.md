# Category

Represents a category assigned to events (`public.categories`).

## Fields

| Field | Type | Constraints |
|---|---|---|
| `category_id` | `bigint` | Primary key; generated always as identity; not null |
| `name` | `character varying` | Not null |

## Relationships

- A category may be referenced by many events. Each event references one category through `events.category_id`.
