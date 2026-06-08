# Pre-publish validation checklist

Run once against a live API key before publishing. Each example in `SKILL.md`
must succeed end-to-end. Do NOT publish if any item fails — fix the docs first.

- [ ] `POST /v1/transcript` for a known video returns 202 + id (or 200 cache hit).
- [ ] Polling `GET /v1/transcript/:id` reaches `status: completed` with a transcript.
- [ ] `POST /v1/summarize` result contains `result.summary.tldr` and `result.summary.key_points`.
- [ ] `POST /v1/transcripts/bulk` with a small playlist returns a batch id that progresses.
- [ ] `GET /v1/search?q=...` returns results.
- [ ] An invalid key returns `401`; a zero-credit account returns `402`.
- [ ] Base URL and dashboard URL placeholders in `SKILL.md` / `README.md` are replaced with real values.
- [ ] `cd backend && npx vitest run src/skill/skillContract.test.ts` passes.
- [ ] From the published repo: `/plugin marketplace add dorik/youtube-transcript-skill` then `/plugin install youtube-transcript@dorik` installs cleanly.
