---
name: adonmo-delete-spot
description: Delete a spot record on the Adonmo API by uuid — an irreversible operation with no published recovery path.
api: Adonmo API
generated: '2026-09-07'
method: generated
source: openapi/adonmoprivatelimited-adonmo-api.json
operations:
  - deleteSpot
---

# Delete an Adonmo spot

Removes a single spot record from the Acumen ops portal.

## Read this first

**This operation is destructive and there is no published way to undo it.** Adonmo ships no
restore, undelete, or trash operation, and states no recovery window anywhere in its
documentation. Treat a successful call as permanent.

There is also **no idempotency mechanism** — no `Idempotency-Key` header, no dedupe window.
If a call times out you cannot safely establish whether it landed by retrying it. Confirm
with `listSpots` instead.

Do not call this without explicit, specific confirmation from the person you are acting
for, naming the uuid being deleted.

## Before you start

- Base URL `https://api.adonmo.com`.
- `Authorization: Bearer <access_token>` is required. Credentials are not self-service;
  contact `dev@adonmo.com`.

## Steps

1. Locate the target record with `listSpots` and capture its `uuid`. Verify it is the
   intended record before proceeding.
2. Call `deleteSpot` — `DELETE /ops_portal/api/spot/{uuid}`.
3. A `204 No Content` means the delete succeeded. There is no response body.
4. Re-run `listSpots` to confirm the record is gone rather than assuming from the status.

```
DELETE /ops_portal/api/spot/<uuid> HTTP/1.1
Host: api.adonmo.com
Authorization: Bearer <access_token>
```

## Error handling

| Status | Body | Meaning |
|---|---|---|
| 204 | empty | Deleted. |
| 400 | `{"errors":["access_token is required."]}` | No `Authorization` header. |
| 401 | HTML Werkzeug page | Invalid or expired token. |

Note that the published contract declares **only** the 204 response. Every error above was
observed by probing, not read from the spec, so treat any other status as undocumented and
stop rather than retrying.
