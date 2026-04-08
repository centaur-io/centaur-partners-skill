---
name: centaur-partners-api
description: Use Centaur Partners API over MCP when Centaur is already configured in Claude, Cursor, or Codex; otherwise use REST with CENTAUR_PARTNER_API_KEY or a partner API key pasted into the current chat session to fetch and summarize read-only Centaur partner data or generate correct curl commands.
---

# Centaur Partners API

Use this skill when a user wants to access Centaur partner data directly from an agent, compare MCP versus REST, or generate correct Centaur curl commands.

## Quick checks

1. Determine whether the current client already has Centaur configured as an MCP server.
2. If MCP is available, prefer MCP and use the matching Centaur read family.
3. If MCP is not available, check whether `CENTAUR_PARTNER_API_KEY` is set.
4. If the env var exists, use REST with `x-api-key`.
5. Otherwise ask whether the user wants to paste a partner API key into the current chat for one-time use.
6. If the user pastes a key, use it transiently for this session only and do not persist or echo it back.
7. If none of those paths are available, stop and give setup instructions instead of inventing credentials or unsupported flows.

## Current surface

- MCP endpoint: `https://partners.centaur.io/mcp`
- Capability families: events, messages, positions, and stats
- REST fallback uses `https://partners.centaur.io/api/v1/*`
- MCP auth: `Authorization: Bearer <partner-api-key>`
- REST auth: `x-api-key: <partner-api-key>`

## Use MCP when available

Prefer MCP for Claude, Cursor, Codex, or any client that already has Centaur configured.

- Use the existing Centaur MCP server rather than rewriting requests as raw HTTP.
- Start with the narrowest matching read for the user's request.
- Keep list reads paginated and bounded.
- Treat the connected server as the source of truth for the exact live tool inventory and request shapes.

Read [references/mcp.md](references/mcp.md) when you need MCP-specific behavior or request shapes.

## Use REST only as fallback

If MCP is not configured, look for `CENTAUR_PARTNER_API_KEY`.

- If present, write or run curl commands against the matching `GET /api/v1/*` read family.
- Always send `x-api-key: $CENTAUR_PARTNER_API_KEY`.
- If the env var is not present but the user pastes a partner API key in chat, use that key only for the current session.
- Keep list reads bounded and paginated.
- Do not claim SDKs, OAuth flows, or endpoints that Centaur does not expose.

Read [references/rest.md](references/rest.md) and [references/examples-curl.md](references/examples-curl.md) for concrete request patterns.

## When blocked

If the agent cannot find MCP config and `CENTAUR_PARTNER_API_KEY` is not set:

- say that Centaur is not configured yet
- offer a one-time path where the user can paste a partner API key into the current chat
- if the user pastes a key, do not echo it back or persist it anywhere
- point the user to [references/client-setup.md](references/client-setup.md)
- ask them to either configure MCP, export `CENTAUR_PARTNER_API_KEY`, or paste a key for the current session

## References

- [references/auth.md](references/auth.md)
- [references/mcp.md](references/mcp.md)
- [references/rest.md](references/rest.md)
- [references/examples-curl.md](references/examples-curl.md)
- [references/client-setup.md](references/client-setup.md)
