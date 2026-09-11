---
name: Upload media and publish a post
description: Upload an image or video to Publora via pre-signed URL, finalize it, and attach it to a scheduled post.
api: openapi/publora-openapi-original.json
operations: [getUploadUrl, completeMedia, createPost, updatePost]
---

# Upload media and publish

Authenticate with `x-publora-key: sk_YOUR_API_KEY`.

Two paths:

**A. Public URL (fastest).** Pass `mediaUrls` (public https direct-file URLs)
directly to `createPost` with `scheduledTime`; Publora downloads and validates
them server-side. Batch is all-or-nothing; per-URL failures come back in
`mediaResults[].code` (e.g. `MEDIA_URL_TOO_LARGE`, `MEDIA_URL_UNSUPPORTED_FORMAT`).
Ingestion is capped at 60 URLs/hour (`429 MEDIA_URL_RATE_LIMITED`, honor `Retry-After`).

**B. Local file (pre-signed upload).**
1. `getUploadUrl` (`POST /get-upload-url`) with `fileName`, `contentType`, `type` —
   returns a pre-signed S3 URL and a `mediaId`.
2. HTTP `PUT` the raw bytes to that URL (not a Publora endpoint).
3. `completeMedia` (`POST /complete-media/{mediaFileId}`) to probe and finalize.
   Transient `503 PROBE_*` codes are retryable after a few seconds.
4. Create a draft with `createPost` (omit `scheduledTime`), then `updatePost`
   (`PUT /update-post/{postGroupId}`) with `status: scheduled` and `scheduledTime`.

## Rules
- Limits: images <=25MB, videos <=150MB (URL ingest); formats jpeg/png/gif/webp/tiff/avif/mp4/mov/webm.
- Instagram/TikTok/YouTube require media before they can be scheduled.
- Use an `Idempotency-Key` on create/update; edits only apply while draft/scheduled.
