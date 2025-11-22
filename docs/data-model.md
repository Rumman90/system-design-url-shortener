# Data Model – URL Shortener

## UrlMappings Table
| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT PK | Primary key |
| short_code | VARCHAR(10) | Unique code |
| long_url | TEXT | Original URL |
| created_at | DATETIME | Created time |
| expires_at | DATETIME | Optional expiry |
| click_count | BIGINT | Analytics |
