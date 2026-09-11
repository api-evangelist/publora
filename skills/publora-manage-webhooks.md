---
name: Subscribe to post-lifecycle webhooks
description: Create and verify Publora webhooks for post.scheduled / published / failed / demoted events.
api: openapi/publora-openapi-original.json
operations: [createWebhook, listWebhooks, updateWebhook, regenerateWebhookSecret, deleteWebhook]
---

# Subscribe to post-lifecycle webhooks

Authenticate with `x-publora-key: sk_YOUR_API_KEY`.

1. **Create** — `createWebhook` (`POST /webhooks`) with `name`, `url` (your
   https receiver), and `events` (any of `post.scheduled`, `post.published`,
   `post.failed`, `post.demoted`). The response includes a `secret` **shown only
   once** — store it. `token.expiring` is subscribable but not dispatched.
2. **Verify each delivery** — Publora POSTs `{ version, event, timestamp, data }`.
   Recompute `HMAC-SHA256` over the raw request bytes with your secret and
   compare to the `X-Publora-Signature` header; read `X-Publora-Event` for the type.
3. **Manage** — `listWebhooks` (`GET /webhooks`), `updateWebhook`
   (`PATCH /webhooks/{id}`; empty strings are ignored, not applied),
   `regenerateWebhookSecret` (`POST /webhooks/{id}/regenerate-secret`),
   `deleteWebhook` (`DELETE /webhooks/{id}`).

## Rules
- On `post.published`, all seven data keys are always present (nulls, never omitted).
- Use `data.postedId` to map an event to the live post on the platform.
- Invalid event names return a flat `{ "error": "Invalid events" }` (names not echoed).
