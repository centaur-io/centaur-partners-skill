# Centaur Partners Skill

Public installable skill for Centaur Partners API.

## Install

```bash
npx skills add https://github.com/centaur-io/centaur-partners-skill
```

## What it covers

- Prefer Centaur MCP when the client already has Centaur configured.
- Fall back to REST when `CENTAUR_PARTNER_API_KEY` is available or when the user provides a one-time key in chat.
- Help with the broader read-only trading data surface across events, messages, generated aggregate summaries, generated channel summaries, positions, discovery, and stats.
- Support direct trader and asset discovery before detail stats calls.
- Generate correct curl commands for the matching `GET /api/v1/*` read family, not only events.
- Guide Claude, ChatGPT, Cursor, and Codex setup for the remote Centaur MCP server.

## Current contract at a glance

- REST surface: `https://partners.centaur.io/api/v1/*`
- MCP endpoint: `https://partners.centaur.io/mcp`
- Time-bounded reads use `startTime` and `endTime`
- List-style reads use forward-only cursor pagination
- Official Claude and ChatGPT installs use OAuth
- API keys are custom/manual fallback only
- Generated Aggregate Summaries and Generated Channel Summaries are standard read families
- Exact live request shapes and examples live in the product docs below

## Repository layout

- `SKILL.md`: installable skill entrypoint
- `references/auth.md`: auth model and decision tree
- `references/mcp.md`: MCP behavior and tool usage
- `references/rest.md`: REST behavior and query guidance
- `references/examples-curl.md`: curl examples only
- `references/client-setup.md`: Claude, ChatGPT, Cursor, and Codex setup

## Product docs

- Docs: [partners.centaur.io/docs](https://partners.centaur.io/docs)
- MCP endpoint: [partners.centaur.io/mcp](https://partners.centaur.io/mcp)
- OpenAPI: [partners.centaur.io/api/v1/openapi.json](https://partners.centaur.io/api/v1/openapi.json)
