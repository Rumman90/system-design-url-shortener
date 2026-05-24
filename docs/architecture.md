# Architecture - URL Shortener

This document expands the high-level and low-level architecture described in the README.

## High-Level Architecture

```mermaid
graph LR
    Client[Client / Browser] --> DNS[DNS]
    DNS --> LB[Load Balancer]
    LB --> API[API Gateway / Web API]
    API --> SVC[URL Service]
    SVC --> CACHE[(Redis Cache)]
    SVC --> DB[(Primary Database)]
    SVC --> QUEUE[Event Queue]
    QUEUE --> WORKER[Analytics Worker]
    WORKER --> ANALYTICS[(Analytics Store)]
    API --> OBS[Logs / Metrics / Traces]
```

## Components

| Component | Responsibility |
| --- | --- |
| Client / Browser | Calls create API or opens a short URL. |
| Load Balancer | Distributes traffic across stateless API instances. |
| API Gateway / Web API | Handles routing, validation, rate limiting, and request metadata. |
| URL Service | Owns short code generation, lookup, expiration checks, and persistence. |
| Redis Cache | Stores hot short-code lookups to reduce database reads. |
| Primary Database | Durable source of truth for URL mappings. |
| Event Queue | Buffers click events so redirects are not blocked by analytics writes. |
| Analytics Worker | Consumes click events and updates counters or analytics tables. |
| Observability | Centralized logs, metrics, traces, and alerts. |

## Create Short URL Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API
    participant S as URL Service
    participant DB as Database
    participant R as Redis

    C->>API: POST /api/urls
    API->>S: Validate longUrl, expiresAt, customAlias
    S->>S: Generate or validate shortCode
    S->>DB: Insert URL mapping
    DB-->>S: Persisted mapping
    S->>R: Cache shortCode -> longUrl
    S-->>API: shortUrl response
    API-->>C: 201 Created
```

## Redirect Flow

```mermaid
sequenceDiagram
    participant C as Browser
    participant API as API
    participant R as Redis
    participant DB as Database
    participant Q as Event Queue

    C->>API: GET /{shortCode}
    API->>R: Lookup shortCode
    alt Cache hit
        R-->>API: longUrl
    else Cache miss
        API->>DB: Query active mapping
        DB-->>API: longUrl / not found / expired
        API->>R: Cache active mapping
    end
    API->>Q: Publish click event async
    API-->>C: 302 Redirect
```

## Scaling Strategy

- Keep API servers stateless so they can scale horizontally.
- Use Redis to reduce database reads for redirect traffic.
- Partition database writes by `short_code` hash or creation time if volume grows.
- Publish analytics events asynchronously so redirects do not wait for analytics writes.
- Add CDN or edge caching for extremely hot redirects when expiration handling is acceptable.

## Reliability Notes

- The primary database remains the source of truth.
- Redis can be unavailable or stale; the service should fall back to the database.
- Unique database constraints prevent duplicate `short_code` values.
- Analytics can be eventually consistent because redirect latency is more important.
