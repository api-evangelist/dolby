---
name: dolby-wire-webhooks
description: Subscribe to Dolby OptiView event webhooks on THEOlive or Millicast, verify signatures, and confirm delivery.
api: Dolby OptiView Live, Dolby OptiView Real-time Streaming (Millicast)
generated: '2026-09-06'
method: generated
source: openapi/dolby-optiview-live-openapi.yml, openapi/dolby-millicast-api-openapi.yml, asyncapi/dolby-webhooks.yml
operations:
  - create-webhook
  - get-webhook-secret
  - get-webhook-logs
  - Webhooks_AddWebhook
  - Webhooks_TestWebhook
  - Webhooks_ListWebhooks
---

# Wire up Dolby OptiView webhooks

The two streaming products have **different** webhook models. Pick the right one.

## OptiView Live (THEOlive) - base `https://api.theo.live/v2`, HTTP Basic

1. `create-webhook` (`POST /webhooks`) with your receiver `url` and the event types you want.
   Forty event names are published, from `channel.created` and `engine.playing` through
   `distribution.security.key.deleted`; `*` subscribes to everything.
2. `get-webhook-secret` (`GET /webhooks/{id}/secret`) returns the signing secret. Verify every
   inbound payload against it before acting.
3. `get-webhook-logs` (`GET /webhooks/{id}/logs`) is cursor-paginated delivery history - the only
   delivery observability anywhere in the Dolby surface. Use it to confirm a webhook actually
   landed rather than assuming.

## Millicast - base `https://api.millicast.com`, HTTP Bearer

1. `Webhooks_AddWebhook` (`POST /api/webhooks`) with a `url` and boolean toggles
   (`isFeedHooks`, `isRecordingHooks`, ...) rather than an event-name list. Six event types
   exist: `Recordings`, `Thumbnail`, `Transcoder`, `Media`, `Feeds`, `ViewerConnection`.
2. The response carries a base64 signing secret. `Webhooks_UpdateWebhook` with `refreshSecret: true`
   rotates it.
3. `Webhooks_TestWebhook` (`POST /api/webhooks/test/{webhookType}`) fires a synthetic payload of
   the chosen type. **Use it.** It is the only rehearsal mechanism in the entire Dolby OptiView
   surface - there is no dry-run mode on any other operation.

## Rules

- **No retry or backoff policy is published** for webhook delivery on either product. Do not
  assume redelivery; on THEOlive, reconcile against `get-webhook-logs`.
- **No AsyncAPI document exists.** The event catalogue in asyncapi/dolby-webhooks.yml was read
  out of the OpenAPI enums, not published by Dolby.
- **Only these two products emit events.** OptiView Ads, the Ad Engine, the Director API and
  Advanced Analytics have no webhook surface at all.
