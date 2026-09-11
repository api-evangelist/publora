---
name: Schedule a post across multiple platforms
description: Publish or schedule one post to several connected social accounts in a single Publora API call.
api: openapi/publora-openapi-original.json
operations: [getPlatformConnections, createPost, getPost]
---

# Schedule a cross-platform post

Authenticate every request with the header `x-publora-key: sk_YOUR_API_KEY`
(base URL `https://api.publora.com/api/v1`).

1. **List connections** — call `getPlatformConnections` (`GET /platform-connections`).
   Copy the exact `platformId` values (form `<platform>-<id>`, e.g. `linkedin-abc123`).
   Never invent or guess IDs.
2. **Create the post** — call `createPost` (`POST /create-post`) with `content`,
   `platforms` (the verbatim IDs), and an ISO-8601 UTC `scheduledTime` in the
   future. Omit `scheduledTime` to create a draft instead.
   - Media-requiring platforms (Instagram, TikTok, YouTube) need media in the
     same call — pass `mediaUrls` (public https direct-file URLs) alongside
     `scheduledTime`, or upload first (see the media skill).
   - Send an `Idempotency-Key` header so a retried create does not double-post.
     A conflicting replay returns `IDEMPOTENCY_KEY_CONFLICT` (422); an in-flight
     duplicate returns `IDEMPOTENCY_IN_FLIGHT` (409) — retry the identical body.
3. **Confirm** — call `getPost` (`GET /get-post/{postGroupId}`) with the returned
   `postGroupId` to read status.

## Rules
- A past `scheduledTime` (>=5 min) returns `400 SCHEDULED_TIME_IN_PAST` in strict mode.
- Match errors on `code`, not the human message (see errors/publora-error-codes.yml).
- To undo: `deletePost` works only while the group is still draft/scheduled;
  once publishing starts it cannot be deleted (`POST_PUBLISH_IN_PROGRESS`).
- Test without publishing: set `platforms` to exactly `['publora-playground']`.
