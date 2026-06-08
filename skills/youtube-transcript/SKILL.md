---
name: youtube-transcript
description: Use when the user wants the transcript, captions, subtitles, or an AI summary of a YouTube video, channel, or playlist — fetches text, translates it, and summarizes it via the Transcript API.
---

# YouTube Transcript API

This skill lets you fetch transcripts, translations, and AI summaries for YouTube
videos, channels, and playlists through the Transcript API.

## Authentication (bring your own key)

This API uses a personal API key, read from the `TRANSCRIPT_API_KEY` environment
variable so the user only provides it once.

1. Check the `TRANSCRIPT_API_KEY` environment variable. If it is set, use it — do
   **not** ask the user.
2. If it is **not** set, ask the user for their key (they can create one in the
   dashboard under **API Keys**, `https://<dashboard-url>/api-keys`). Use it for
   this session, then tell them to make it permanent — so they are never asked
   again — by adding this line to their shell profile (e.g. `~/.zshrc`) and
   reopening their terminal:

   ```
   export TRANSCRIPT_API_KEY=yt_live_...
   ```

3. **Never** print the key back to the user or write it into logs or files.
4. Send it on every request as a header: `Authorization: Bearer $TRANSCRIPT_API_KEY`.

The base URL is `https://yt-transcripts-api-1nwp.onrender.com`.

## When to use which endpoint

| The user wants… | Use |
| --- | --- |
| Transcript/captions of one video | `POST /v1/transcript` |
| A summary (TL;DR + key points) of one video | `POST /v1/summarize` |
| Transcript translated to another language | `POST /v1/transcript` with `translate_to` |
| Transcripts for a whole playlist / channel / list of videos | `POST /v1/transcripts/bulk` |
| To find videos/channels | `GET /v1/search` |
| Recent videos from a channel | `GET /v1/channel/videos` |

Full parameter and credit details: see `references/api.md`.

## The polling pattern (important)

Transcript and summarize calls are asynchronous. A successful create returns
**HTTP 202** with `{ "id": "..." }` — this is success, not an error. Then:

1. Call `GET /v1/transcript/:id`.
2. If `status` is `queued` or `processing`, wait ~2s and poll again.
3. Stop when `status` is `completed` (use `result`) or `failed` (report `error`).
4. A create call may instead return **200** with the full result already attached
   (cache hit) — in that case, skip polling.

## Examples

### 1. Transcript of one video
```bash
# create
curl -s -X POST https://yt-transcripts-api-1nwp.onrender.com/v1/transcript \
  -H "Authorization: Bearer $TRANSCRIPT_API_KEY" -H "Content-Type: application/json" \
  -d '{"url":"https://www.youtube.com/watch?v=VIDEO_ID"}'
# -> {"id":"abc123"}  (HTTP 202)

# poll until completed
curl -s https://yt-transcripts-api-1nwp.onrender.com/v1/transcript/abc123 \
  -H "Authorization: Bearer $TRANSCRIPT_API_KEY"
# -> {"id":"abc123","status":"completed","result":{...transcript...}}
```

### 2. AI summary of one video
```bash
curl -s -X POST https://yt-transcripts-api-1nwp.onrender.com/v1/summarize \
  -H "Authorization: Bearer $TRANSCRIPT_API_KEY" -H "Content-Type: application/json" \
  -d '{"url":"https://youtu.be/VIDEO_ID"}'
# poll GET /v1/transcript/:id -> result.summary = {tldr, key_points, model, generated_at}
```

### 3. Whole playlist (bulk)
```bash
curl -s -X POST https://yt-transcripts-api-1nwp.onrender.com/v1/transcripts/bulk \
  -H "Authorization: Bearer $TRANSCRIPT_API_KEY" -H "Content-Type: application/json" \
  -d '{"playlist":"https://www.youtube.com/playlist?list=PLAYLIST_ID"}'
# -> {"id":"batch_xyz"} ; poll GET /v1/transcripts/batches/batch_xyz
```

## Handling errors

- `401` — the key is missing, invalid, or revoked. Ask the user for a valid key.
- `402` — the account is out of credits or needs a plan upgrade. Tell the user to
  top up or upgrade; do not retry automatically.
- `429` — rate limited. Wait until the time in the `X-RateLimit-Reset` header,
  then retry.

## Alternative: native MCP connection

If the user prefers a native MCP integration instead of REST calls, point them to
the Transcript MCP server (it uses OAuth, not an API key). See the main project's MCP documentation. This
skill's quickstart uses the REST + API-key path above.
