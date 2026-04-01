---
name: centaur-partners-api
description: Use Centaur Partners API over MCP when Centaur is already configured in Claude, Cursor, or Codex; otherwise use REST with the CENTAUR_PARTNER_API_KEY environment variable to fetch and summarize partner trading events or generate correct curl commands.
---

# Centaur Partners API

Use this skill when a user wants to access Centaur partner events directly from an agent, compare MCP versus REST, or generate correct Centaur curl commands.

## Quick checks

1. Determine whether the current client already has Centaur configured as an MCP server.
2. If MCP is available, prefer MCP and call `list_events`.
3. If MCP is not available, check whether `CENTAUR_PARTNER_API_KEY` is set.
4. If the env var exists, use REST with `x-api-key`.
5. If neither MCP nor the env var is available, stop and give setup instructions instead of inventing credentials or unsupported flows.

## Current surface

- MCP endpoint: `https://partners.centaur.io/mcp`
- MCP tool: `list_events`
- REST endpoint: `GET https://partners.centaur.io/api/v1/events`
- Required permission: `events.read`
- MCP auth: `Authorization: Bearer <partner-api-key>`
- REST auth: `x-api-key: <partner-api-key>`

## Use MCP when available

Prefer MCP for Claude, Cursor, Codex, or any client that already has Centaur configured.

- Use the existing Centaur MCP server rather than rewriting requests as raw HTTP.
- Call `list_events` with the narrowest valid arguments first.
- Keep pagination explicit. `after` and `before` are mutually exclusive.
- Respect the event-history floor of `2026-01-01T00:00:00.000Z`.

Read [references/mcp.md](references/mcp.md) when you need MCP-specific behavior or request shapes.

## Use REST only as fallback

If MCP is not configured, look for `CENTAUR_PARTNER_API_KEY`.

- If present, write or run curl commands against `GET /api/v1/events`.
- Always send `x-api-key: $CENTAUR_PARTNER_API_KEY`.
- Keep filters and pagination explicit.
- Do not claim SDKs, OAuth flows, or endpoints that Centaur does not expose.

Read [references/rest.md](references/rest.md) and [references/examples-curl.md](references/examples-curl.md) for concrete request patterns.

## When blocked

If the agent cannot find MCP config and `CENTAUR_PARTNER_API_KEY` is not set:

- say that Centaur is not configured yet
- point the user to [references/client-setup.md](references/client-setup.md)
- ask them to either configure MCP or export `CENTAUR_PARTNER_API_KEY`

## References

- [references/auth.md](references/auth.md)
- [references/mcp.md](references/mcp.md)
- [references/rest.md](references/rest.md)
- [references/examples-curl.md](references/examples-curl.md)
- [references/client-setup.md](references/client-setup.md)
