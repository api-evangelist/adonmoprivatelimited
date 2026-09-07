---
name: adonmo-list-spots
description: Page through Acumen ops-portal spot records on the Adonmo API, with optional free-text search and status filtering.
api: Adonmo API
generated: '2026-09-07'
method: generated
source: openapi/adonmoprivatelimited-adonmo-api.json
operations:
  - listSpots
---

# List Adonmo spots

Retrieves a page of spot records from the Acumen ops portal.

## Before you start

- Base URL is `https://api.adonmo.com`. It is not in the published OpenAPI document; it was
  confirmed by probing.
- Every call needs a bearer access token: `Authorization: Bearer <access_token>`.
- **There is no self-service way to get a token.** Adonmo publishes no signup, no key page,
  and the Swagger UI at `https://api.adonmo.com/apidocs/` renders without an Authorize
  control. Ask Adonmo (`dev@adonmo.com`) for credentials before attempting this flow.

## Steps

1. Call `listSpots` — `GET /ops_portal/api/spots`.
2. `page` and `page_size` are **both required**. Omitting either is a client error, not a
   defaulted request. `page` starts at 1; `page_size` accepts 1–10000.
3. Optionally narrow with `search` (free text) and `status` (a string; Adonmo does not
   publish the allowed values, so discover them from live data rather than assuming).
4. Advance `page` until a page comes back short or empty.

```
GET /ops_portal/api/spots?page=1&page_size=100 HTTP/1.1
Host: api.adonmo.com
Authorization: Bearer <access_token>
```

## Reading the response

The 200 response references `SpotListSchema`, but that schema is **not defined** in the
published document — `definitions` is empty. You cannot rely on a documented envelope.
Inspect the first live response and code against what you actually observe, and do not
assume a `total`, `next`, or `has_more` field exists.

## Error handling

| Status | Body | Meaning |
|---|---|---|
| 400 | `{"errors":["access_token is required."]}` | No `Authorization` header. Note this is 400, not 401. |
| 401 | HTML Werkzeug page | Token present but invalid or expired. Not JSON — do not try to parse it. |
| 404 | `{"error":"404 Not Found"}` | Wrong path. Note the singular `error` key here versus the plural `errors` array above. |

Match on status codes, not on message text: the API publishes no stable machine-readable
error codes.

## Rate limits

None are published and no rate-limit headers are returned. Back off on your own schedule
and keep `page_size` moderate rather than requesting the 10000 maximum.
