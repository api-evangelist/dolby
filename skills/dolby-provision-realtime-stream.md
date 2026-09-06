---
name: dolby-provision-realtime-stream
description: Provision a Millicast (OptiView Real-time) publish token and subscribe token, then stop the stream cleanly.
api: Dolby OptiView Real-time Streaming (Millicast)
base_url: https://api.millicast.com
generated: '2026-09-06'
method: generated
source: openapi/dolby-millicast-api-openapi.yml and openapi/dolby-millicast-director-openapi.yml
operations:
  - PublishTokenV1_CreateToken
  - PublishTokenV1_ListTokens
  - PublishTokenV1_GetActiveTokenByStreamId
  - SubscribeTokenV1_CreateToken
  - PublishTokenV1_DisableTokens
  - Stream_StopStream
  - PublishTokenV1_DeleteToken
---

# Provision a Millicast real-time stream

Millicast is the sub-second WebRTC surface. The REST API mints the credentials; the actual
publish and subscribe happen against the **Director API** on a different host
(`https://director.millicast.com`), which speaks IETF **WHIP** (`POST /api/whip/{streamName}`)
and **WHEP** (`POST /api/whep/{streamAccountId}/{streamName}`).

## Auth
`Authorization: Bearer <API Secret>`. Create the secret in the streaming dashboard under
Settings -> Security -> API Secrets.

## Steps

1. **Create a publish token.** `PublishTokenV1_CreateToken` (`POST /api/publish_token`) naming
   the stream. The response carries the token used by the broadcaster.
2. **Create a subscribe token** if the stream is not public.
   `SubscribeTokenV1_CreateToken` (`POST /api/subscribe_token`).
3. **Publish** with the Director API: `POST https://director.millicast.com/api/whip/{streamName}`
   (or `/api/director/publish` for the legacy signalling path).
4. **Subscribe** with `POST https://director.millicast.com/api/whep/{streamAccountId}/{streamName}`.
5. **Check what is live.** `PublishTokenV1_GetActiveTokenByStreamId`
   (`GET /api/publish_token/active`) or `PublishTokenV1_GetAllActiveTokensByAccount`.
6. **Wind down.** `Stream_StopStream` (`POST /api/stream/stop`) for one stream, or
   `Stream_StopByAccount` (`POST /api/stream/stop/all`) for everything on the account.

## Rules

- **Disable, do not delete, to pause access.** `PublishTokenV1_DisableTokens`
  (`PATCH /api/publish_token/disable`) is reversible. `PublishTokenV1_DeleteToken`
  (`DELETE /api/publish_token/{tokenId}`) is not, and since 2025-05-30 it **immediately
  terminates every live stream using that token** - previously they ran until the publisher
  disconnected. Confirm no active stream with `PublishTokenV1_GetActiveTokenByStreamId` first.
- **Avoid the deprecated update path.** `PublishTokenV1_UpdateToken`
  (`PUT /api/publish_token/{tokenId}`) and `SubscribeTokenV1_UpdateToken` are flagged
  `deprecated: true`. Use `PublishTokenV2_UpdateToken` (`PUT /api/v2/publish_token/{tokenId}`)
  and `SubscribeTokenV2_UpdateToken`. Thirty of this API's 129 operations are deprecated and
  none carries a sunset date - see lifecycle/dolby-lifecycle.yml before you build on any of them.
- **No idempotency on token creation.** A retried `POST /api/publish_token` mints a second token.
  List by name (`PublishTokenV1_ListTokensByName`) before retrying.
- **Error envelope** is `{"status": "fail", "data": {"message": "..."}}`, not RFC 9457. Check
  `status`, not just the HTTP code.
