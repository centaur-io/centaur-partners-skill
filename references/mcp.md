# MCP

Use MCP when the client already has Centaur configured.

## Endpoint

- `https://partners.centaur.io/mcp`

## Auth

- Preferred: client-managed OAuth with Dynamic Client Registration when needed
- Compatibility fallback: bearer partner API key
- Last resort for clients with no header support: `?api_key=...`

## Capability families

- events
- messages
- channel summaries
- positions
- discovery
- stats

Use `list_traders` and `list_assets` as discovery tools when the user does not already know the right IDs.

Use `list_channel_summaries` for privacy-preserving channel insight. Do not use raw messages as the first choice for channel recaps when generated summaries are available.

## Use pattern

- Start with the narrowest matching read for the request.
- If the request depends on knowing a trader or asset first, discover it directly with `list_traders` or `list_assets` before calling detail stats.
- Use explicit `limit` and entity filters when they matter.
- Keep list reads paginated deliberately.
- Use the connected MCP server as the source of truth for exact tools, resources, and argument shapes.
