# Transcript API — Endpoint Reference

Base URL: `https://yt-transcripts-api-1nwp.onrender.com`. All requests
require `Authorization: Bearer yt_live_...`. Responses are JSON.

Transcript and summary jobs are **asynchronous**: a create call returns
`202 Accepted` with `{ "id": "..." }` (or `200` with the full result on a cache
hit). Poll `GET /v1/transcript/:id` until `status` is `completed` or `failed`.

## POST /v1/transcript
Fetch a transcript for one video.
Body:
- `url` (string, required) — YouTube video URL or ID.
- `language` (string, optional) — preferred caption language code.
- `native_only` (boolean, optional) — only use YouTube's own captions; do not fall back to Whisper.
- `translate_to` (string, optional) — translate the transcript into this language code.
- `include_summary` (boolean, optional) — also generate an AI summary (`tldr` + `key_points`).
Credit cost: 1 (native captions) or per-minute (Whisper audio). Summary adds 0.

## POST /v1/summarize
Convenience wrapper: same as `/v1/transcript` but always returns JSON with a
`summary` object attached.
Body: `url` (required), `language` (optional), `translate_to` (optional).
Credit cost: same as the underlying transcript; the summary itself is free.

## GET /v1/transcript/:id
Poll a transcript or summarize job.
Returns `{ id, status, result?, error? }`. When `status` is `completed`, `result`
holds the transcript metadata, the transcript text/segments, and (if requested)
`result.summary = { tldr, key_points, model, generated_at }`.

## POST /v1/transcripts/bulk
Start a batch job over many videos.
Body — provide exactly ONE source:
- `playlist` (string) — playlist URL/ID, OR
- `channel` (string) — channel URL/ID/handle, OR
- `urls` (string[]) — explicit list of video URLs/IDs.

When the source is `channel`, an optional `channelMode` controls what is pulled:
- `channelMode` (`videos` | `latest` | `search`, default `videos`).
- `channelQuery` (string) — the text to search for within the channel; required only when `channelMode` is `search`.

Plus optional `language`, `native_only`, `translate_to`, and `limit` (integer,
default 50, max 100 — caps how many videos are pulled from a channel/playlist).
Returns a batch `id`; poll `GET /v1/transcripts/batches/:id` for progress.

## GET /v1/search
Search YouTube.
Query params: `q` or `query` (search text), `type` (`video` | `channel` |
`playlist` | `all`, default `video`), `limit` (1–50, default 10).
Credit cost: 1 per call.

## GET /v1/channel/videos
List recent videos for a channel.
Query params: `channel` (required), `limit` (1–50, default 10).
Credit cost: 1 per call.

## Errors
- `401` — missing/invalid/revoked key. Ask the user to re-enter a valid key.
- `402` — out of credits or plan upgrade required. Tell the user to top up or upgrade.
- `429` — rate limited. Respect `X-RateLimit-Reset` and retry after it.
