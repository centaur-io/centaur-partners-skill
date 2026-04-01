# REST

Use REST when MCP is not configured and `CENTAUR_PARTNER_API_KEY` is available.

## Endpoint

- `GET https://partners.centaur.io/api/v1/events`

## Header

```text
x-api-key: $CENTAUR_PARTNER_API_KEY
```

## Query guidance

- Use explicit filters rather than broad fetches when possible.
- `after` and `before` are mutually exclusive.
- Use `sortOrder` when order matters.
- Respect the history floor: `2026-01-01T00:00:00.000Z`.
- If `to` is before the floor, expect an empty page.

## Supported filters

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
