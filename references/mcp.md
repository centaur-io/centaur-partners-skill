# MCP

Use MCP when the client already has Centaur configured.

## Endpoint

- `https://partners.centaur.io/mcp`

## Tool inventory

- `list_events`

## Resource inventory

- `centaur://partners-api/capabilities`
- `centaur://partners-api/event-filters`

## Use pattern

- Start with `list_events` and the narrowest valid arguments for the request.
- Use explicit `limit`, `from`, `to`, and entity filters when they matter.
- Keep pagination deliberate. `after` and `before` cannot be used together.
- Respect the history floor: `2026-01-01T00:00:00.000Z`.

## Supported arguments

- `traderIds`
- `assetIds`
- `directions`
- `eventTypes`
- `from`
- `to`
- `limit`
- `after`
- `before`
- `sortOrder`
