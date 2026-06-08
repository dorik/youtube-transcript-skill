# YouTube Transcript — Agent Skill (Claude Code)

A Claude Code Agent Skill that teaches the assistant to fetch YouTube
transcripts, translations, and AI summaries through the Transcript API, using
your own API key (bring-your-own-key).

## What you need
- A Transcript API key (`yt_live_...`). Create one in the dashboard under
  **API Keys**: `https://<dashboard-url>/api-keys`.

## Install (copy-paste)

First get the files — clone (or download) the skill repo:

```
git clone https://github.com/dorik/youtube-transcript-skill
cd youtube-transcript-skill
```

Then copy the skill folder into a skills directory and restart Claude Code:

```
# project-local (one project only)
cp -r skills/youtube-transcript <your-project>/.claude/skills/youtube-transcript

# or personal (all your projects)
cp -r skills/youtube-transcript ~/.claude/skills/youtube-transcript
```

That's it. The skill auto-loads and triggers when you ask for a YouTube
transcript or summary. On first use, the agent asks for your API key and stores
it locally.

## Files
- `skills/youtube-transcript/SKILL.md` — what the agent reads (capabilities, auth, examples).
- `skills/youtube-transcript/references/api.md` — endpoint reference.

## Optional: install as a plugin

This folder is also packaged as a Claude Code plugin (see `.claude-plugin/`), so
it can be installed via a marketplace once published to a public repo:

```
/plugin marketplace add dorik/youtube-transcript-skill
/plugin install youtube-transcript@dorik
```

(The `/plugin marketplace add` form needs the public repo to exist first.)

## Supported agents
This is a **Claude Code** skill (the `SKILL.md` format). Other tools differ:
- The **Anthropic Agent SDK** supports Agent Skills, but in a different format — not this package as-is.
- **Cursor and most other agents do not support the `SKILL.md` format.** For those,
  call the REST API directly (see `skills/youtube-transcript/references/api.md`) or use the MCP server.

## Native MCP alternative
Prefer a native MCP connection (OAuth, no manual key)? Connect to the Transcript
MCP server instead — see the main project docs for the MCP endpoint.
