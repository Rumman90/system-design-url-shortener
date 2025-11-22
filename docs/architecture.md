# Architecture – URL Shortener

```mermaid
graph LR
    User --> API[Web API / Gateway]
    API --> SVC[URL Service]
    SVC --> CACHE[(Redis)]
    SVC --> DB[(Primary Database)]
```

### Components
- API Layer
- URL Service
- Redis Cache
- Database
