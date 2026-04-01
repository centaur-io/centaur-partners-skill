# Centaur Partners Skill

Public installable skill for Centaur Partners API.

## Install

```bash
npx skills add https://github.com/zap-xyz/centaur-partners-skill
```

## What it covers

- Prefer Centaur MCP when the client already has Centaur configured.
- Fall back to REST only when `CENTAUR_PARTNER_API_KEY` is available.
- Generate correct curl commands for `GET /api/v1/events`.
- Guide Claude, Cursor, and Codex setup for the remote Centaur MCP server.

## Repository layout

- `SKILL.md`: installable skill entrypoint
- `references/auth.md`: auth model and decision tree
- `references/mcp.md`: MCP behavior and tool usage
- `references/rest.md`: REST behavior and query guidance
- `references/examples-curl.md`: curl examples only
- `references/client-setup.md`: Claude, Cursor, and Codex setup

## Product docs

- Docs: [partners.centaur.io/docs](https://partners.centaur.io/docs)
- MCP endpoint: [partners.centaur.io/mcp](https://partners.centaur.io/mcp)
- OpenAPI: [partners.centaur.io/api/v1/openapi.json](https://partners.centaur.io/api/v1/openapi.json)
