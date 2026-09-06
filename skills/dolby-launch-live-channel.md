---
name: dolby-launch-live-channel
description: Stand up a Dolby OptiView Live (THEOlive) channel end to end - channel, ingest, engine, distribution - and start it.
api: Dolby OptiView Live
base_url: https://api.theo.live/v2
generated: '2026-09-06'
method: generated
source: openapi/dolby-optiview-live-openapi.yml and https://optiview.dolby.com/docs/theolive/api/full-example/
operations:
  - get-regions
  - get-abr-ladders
  - create-channel
  - create-channel-ingest
  - create-channel-engine
  - create-channel-distribution
  - start-channel
  - get-channel-monitoring
  - stop-channel
---

# Launch a Dolby OptiView Live channel

OptiView Live V2 is a **component model**. A channel is only a container; the pipeline is
`Channel -> Ingest -> Engine -> Distribution` and each part is a separate create call wired by id.
Four calls minimum before anything streams. (V1's single-call channel create no longer exists -
see https://optiview.dolby.com/docs/theolive/api/migration-from-v1/.)

## Auth
HTTP Basic. Base64-encode `key:secret` from the dashboard's API Tokens page and send
`Authorization: Basic <base64>`. The secret is displayed once at creation and cannot be retrieved
again.

## Steps

1. **Pick a region and an ABR ladder.** `get-regions` (`GET /regions`) and `get-abr-ladders`
   (`GET /abr`). Both are cursor-paginated. Keep the `id` values.
2. **Create the channel.** `create-channel` (`POST /channels`) with a `name`. Keep `channelId`.
3. **Create the ingest.** `create-channel-ingest` (`POST /channels/{id}/ingests`) with a `name`
   and a `type` of `rtmp-push`, `rtmp-pull` or `srt-pull`. Keep `ingestId`.
4. **Create the engine.** `create-channel-engine` (`POST /channels/{id}/engines`) with
   `ingestId`, `region` and `quality.abrLadderId`. Keep `engineId`.
5. **Create the distribution.** `create-channel-distribution`
   (`POST /channels/{id}/distributions`) referencing the engine. This is the viewer-facing
   output and holds security, delivery and player settings.
6. **Start it.** `start-channel` (`POST /channels/{id}/start`) starts every connected engine.
7. **Watch it come up.** Poll `get-channel-monitoring` (`GET /monitoring/channels/{channelId}`).
   A 404 here returns a `MonitoringError` `{code, message}` object.

## Rules

- **There is no idempotency on this API.** No `Idempotency-Key` header is accepted on any
  THEOlive operation. If step 2 times out and you retry, you get a second channel. Read back with
  `get-channels` and match on `name` before retrying a create.
- **No rate-limit headers.** Nothing tells you how long to back off. Only one 429 is documented
  in the whole API, on `send-channel-instream-metadata`, and it carries no `Retry-After`.
- **Reversal.** `stop-channel` (`POST /channels/{id}/stop`) reverses `start-channel` and stops
  all connected engines. There is **no undo for delete** - `delete-channel`, `delete-ingest`,
  `delete-engine` and `delete-distribution` are permanent and Dolby publishes no restore window.
  Stop before you delete, and confirm with `get-channel` first.
- **Pagination.** Every list endpoint takes `limit` (default 20) and `cursor`. Read
  `pagination.hasMore` and pass `pagination.cursor` back. Never assume one page.
- **Errors** are bare descriptive strings, not RFC 9457 problem documents. See
  errors/dolby-problem-types.yml.
