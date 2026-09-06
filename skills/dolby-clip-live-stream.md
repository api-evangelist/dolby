---
name: dolby-clip-live-stream
description: Clip a segment out of a live or recorded Millicast stream into a media asset, using the one idempotent write in the Dolby surface.
api: Dolby OptiView Real-time Streaming (Millicast)
base_url: https://api.millicast.com
generated: '2026-09-06'
method: generated
source: openapi/dolby-millicast-api-openapi.yml
operations:
  - MediaAssets_CreateMediaAsset
  - MediaAssets_ListMediaAssets
  - MediaAssets_ReadMediaAsset
  - MediaAssets_UpdateMediaAsset
  - MediaAssets_DeleteMediaAssets
---

# Clip a live Millicast stream

Live clipping captures a partial clip from an active stream and makes it immediately available -
the basis for social sharing, replay and archival. Added 2024-10-15; engine-priority selection
added 2026-05-25.

## Auth
`Authorization: Bearer <API Secret>`.

## Steps

1. **Create the clip.** `MediaAssets_CreateMediaAsset` (`POST /api/v3/media/assets`) with the
   source stream and the time window.
2. **Send an `Idempotency-Key` header.** This is the **only** operation in the entire Dolby
   OptiView surface that accepts one. Dolby documents it explicitly: reuse the same key and a
   repeated request will not generate a second clip. Use a stable key derived from the stream id
   and the clip window, not a fresh UUID per attempt.
3. **Poll for readiness.** `MediaAssets_ReadMediaAsset`
   (`GET /api/v3/media/assets/{mediaAssetId}`).
4. **Find clips later.** `MediaAssets_ListMediaAssets` (`GET /api/v3/media/assets`).

## Rules

- **Do not use `/api/record_files/*`.** The entire legacy recording and clipping family
  (`RecordFiles_CreateRecordClip`, `RecordFiles_ListRecordFiles`, `RecordFiles_GetClipRequest`
  and 20 or so siblings) is flagged `deprecated: true` in the live specification. `/api/v3/media/assets`
  is the current surface.
- **Deletes are permanent.** `MediaAssets_DeleteMediaAssets` and especially
  `MediaAssets_DeleteMediaAssets2` (`DELETE /api/v3/media/assets/all/{type}`, which deletes ALL
  assets of a type) have no restore path and no published recovery window.
- **Recordings expire.** The account carries an expiration policy
  (`GET /api/v3/account/media/expiration`) and clip sources carry an expiry rule. Read them before
  assuming an asset will still be there.
- **Download URLs are moving.** Per the 2026-02-23 changelog, recording downloads are being
  migrated from S3 pre-signed URLs to Dolby's CDN in stages. Do not hard-code the host.
