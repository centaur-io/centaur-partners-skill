# MCP

Use MCP when the client already has Centaur configured.

## Endpoint

- `https://partners.centaur.io/mcp`

## Auth

- Official Claude and ChatGPT installs use OAuth.
- Custom/manual fallback: bearer partner API key.
- Last-resort custom/manual fallback: `?api_key=...`.

## Capability families

- events
- messages
- channel summaries
- positions
- discovery
- stats

Use `list_traders` and `list_assets` as discovery tools when the user does not already know the right IDs.

Use `list_channel_summaries` for compact channel insight when authorized. It requires `summaries.read`; if that permission is missing, fall back to `list_messages`.

## Use pattern

- Start with the narrowest matching read for the request.
- If the request depends on knowing a trader or asset first, discover it directly with `list_traders` or `list_assets` before calling detail stats.
- Use explicit `limit` and entity filters when they matter.
- Keep list reads paginated deliberately.
- Use the connected MCP server as the source of truth for exact tools, resources, and argument shapes.
