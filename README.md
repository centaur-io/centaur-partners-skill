# Centaur API Skill

Public installable skill for the Centaur API.

## Install

```bash
npx skills add https://github.com/centaur-io/centaur-partners-skill
```

## What it covers

- Prefer Centaur MCP when the client already has Centaur configured.
- Use REST when `CENTAUR_API_KEY` is available or when the user provides a one-time key in chat.
- Help with the broader read-only trading data surface across the feed, events, messages, Generated Aggregate Narrative Summaries, Generated Channel Narrative Summaries, positions, discovery, stats, trader rankings, and activity summaries.
- Support direct trader and asset discovery before detail stats calls.
- Generate correct curl commands for the matching `GET /api/v1/*` read family, not only events.
- Guide Claude, ChatGPT, Cursor, and Codex setup for the remote Centaur MCP server.

## Current contract at a glance

- REST surface: `https://partners.centaur.io/api/v1/*`
- MCP endpoint: `https://partners.centaur.io/mcp`
- Time-bounded reads use `startTime` and `endTime`
- List-style reads use forward-only cursor pagination
- Official Claude and ChatGPT installs use OAuth
- OAuth is available to active, email-verified signed-up users
- API keys are self-serve REST credentials
- Generated Aggregate Narrative Summaries and Generated Channel Narrative Summaries are standard read families
- The feed (`GET /api/v1/feed`, `list_feed`) returns presentation-ready source-message groups with curated events; `since` polls for changes with whole-group upserts
- Summary tools may return empty pages when no generated summaries match the requested window
- Source Message IDs are opaque; hydrate only IDs returned by message rows or event `messageId`
- Exact live request shapes and examples live in the product docs below

## Repository layout

- `SKILL.md`: installable skill entrypoint
- `references/auth.md`: auth model and decision tree
- `references/mcp.md`: MCP behavior and tool usage
- `references/rest.md`: REST behavior and query guidance
- `references/examples-curl.md`: curl examples only
- `references/client-setup.md`: Claude, ChatGPT, Cursor, and Codex setup
- `references/message-digests.md`: weighting, filtering, and structuring message digests

## Product docs

- Docs: [partners.centaur.io/docs](https://partners.centaur.io/docs)
- MCP endpoint: [partners.centaur.io/mcp](https://partners.centaur.io/mcp)
- OpenAPI: [partners.centaur.io/api/v1/openapi.json](https://partners.centaur.io/api/v1/openapi.json)
