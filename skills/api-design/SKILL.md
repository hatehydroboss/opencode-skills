---
name: api-design
description: Design, review, and implement REST and GraphQL APIs. Use when building endpoints, defining routes, designing request/response schemas, implementing pagination, versioning, rate limiting, or reviewing API contracts. Trigger on "API", "endpoint", "route", "REST", "GraphQL", "webhook", "HTTP status codes", or "API versioning".
---

# API Design

Design APIs that are consistent, predictable, and pleasant to consume.

## REST Design Principles

### Resource Naming
- Use nouns, not verbs: `/users` not `/getUsers`
- Plural for collections: `/articles`, `/comments`
- Nest for hierarchy: `/articles/{id}/comments`
- Max 2 levels of nesting; use query params beyond that

### HTTP Methods
| Method | Purpose | Idempotent | Body |
|--------|---------|-----------|------|
| GET | Read | Yes | No |
| POST | Create | No | Yes |
| PUT | Full replace | Yes | Yes |
| PATCH | Partial update | Yes | Yes |
| DELETE | Remove | Yes | No |

### Status Codes
- `200` OK, `201` Created, `204` No Content
- `400` Bad Request, `401` Unauthenticated, `403` Forbidden, `404` Not Found
- `409` Conflict, `422` Unprocessable, `429` Rate Limited
- `500` Internal Server Error (never leak internals)

### Response Envelope
```json
{
  "data": { ... },
  "meta": { "page": 1, "per_page": 20, "total": 150 },
  "error": null
}
```

### Error Format
```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Email is required",
    "details": [{ "field": "email", "issue": "required" }]
  }
}
```

## Pagination

Always use cursor-based for infinite scroll, offset for admin tables:

```
GET /items?cursor=abc123&limit=20    → cursor-based
GET /items?page=3&per_page=20       → offset-based
```

Never return unbounded lists. Default limit 20, max 100.

## Versioning

- URL path versioning for breaking changes: `/v1/users`
- Header versioning for additive changes: `Accept: application/vnd.api.v2+json`
- Never break existing contracts; deprecate with `Deprecation` header and `Sunset` date

## Rate Limiting

Always include in responses:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1700000000
Retry-After: 30  (on 429)
```

## Authentication

- Bearer tokens for stateless APIs (JWT)
- OAuth 2.0 for third-party access
- API keys for server-to-server (identify, not authenticate)
- Never put tokens in URLs or query strings

## GraphQL Specifics

- Use connections for pagination (Relay spec): `edges { node }`, `pageInfo { hasNextPage, endCursor }`
- Depth-limit queries to prevent abuse (max depth 10)
- Persisted queries in production
- Separate mutations from queries by naming convention

## Checklist for New APIs

- [ ] Consistent naming (nouns, plural, snake_case or camelCase — pick one)
- [ ] All error responses follow the envelope format
- [ ] Pagination on every collection endpoint
- [ ] Rate limiting configured
- [ ] Input validation with clear error messages
- [ ] Idempotency keys on POST endpoints that create resources
- [ ] OpenAPI/Swagger spec maintained alongside code
