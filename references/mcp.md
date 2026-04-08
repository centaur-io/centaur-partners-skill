# MCP

Use MCP when the client already has Centaur configured.

## Endpoint

- `https://partners.centaur.io/mcp`

## Capability families

- events
- messages
- positions
- stats

## Use pattern

- Start with the narrowest matching read for the request.
- Use explicit `limit` and entity filters when they matter.
- Keep list reads paginated deliberately.
- Use the connected MCP server as the source of truth for exact tools, resources, and argument shapes.
