# API Design - URL Shortener

## Create Short URL

`POST /api/urls`

Request:

```json
{
  "longUrl": "https://example.com/products?id=12345",
  "expiresAt": null,
  "customAlias": null
}
```

Response:

```json
{
  "shortCode": "QwE12R",
  "shortUrl": "https://short.ly/QwE12R",
  "expiresAt": null
}
```

Status codes:

| Status | Meaning |
| --- | --- |
| `201 Created` | Short URL was created. |
| `400 Bad Request` | Invalid URL, invalid alias, or invalid expiration. |
| `409 Conflict` | Custom alias already exists. |
| `429 Too Many Requests` | Rate limit exceeded. |

## Redirect

`GET /{shortCode}`

Behavior:

- Returns `302 Found` with `Location: {longUrl}` for active links.
- Returns `404 Not Found` if the code does not exist.
- Returns `410 Gone` if the link existed but has expired.

## Health Check

`GET /health`

Returns service status for load balancers and uptime checks.
