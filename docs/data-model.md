# Data Model - URL Shortener

## `url_mappings`

| Column | Type | Notes |
| --- | --- | --- |
| `id` | `BIGINT` | Primary key. |
| `short_code` | `VARCHAR(10)` | Unique, indexed lookup key. |
| `long_url` | `TEXT` | Original destination URL. |
| `created_at` | `DATETIME` | Creation timestamp. |
| `expires_at` | `DATETIME NULL` | Optional expiration timestamp. |
| `click_count` | `BIGINT` | Denormalized counter for basic analytics. |
| `is_active` | `BOOLEAN` | Allows soft disabling links. |

## Recommended Indexes

- Unique index on `short_code`.
- Index on `expires_at` for cleanup jobs.
- Optional index on `created_at` for reporting and retention.

## Cache Representation

```text
url:{shortCode} -> {longUrl, expiresAt, isActive}
```

Redis is used as a cache, not the source of truth. The database record should be used to rebuild cache entries after misses or Redis failures.
