# REST

Use REST when MCP is not configured and `CENTAUR_PARTNER_API_KEY` is available.

## Endpoint shape

- Use the matching read family under `GET https://partners.centaur.io/api/v1/*`.
- Common families include events, messages, positions, and stats.

## Header

```text
x-api-key: $CENTAUR_PARTNER_API_KEY
```

## Query guidance

- Use `GET /api/v1/traders` and `GET /api/v1/assets` for direct discovery when the right IDs are not known yet.
- Use explicit filters rather than broad fetches when possible.
- Historical list reads use `startTime` and `endTime` when explicit bounds are needed.
- List reads use forward-only cursor pagination via `cursor`.
- Stats reads return aggregate subject and summary data instead of list pages.
- Use the main Centaur docs site for the exact live endpoint, query, and response contract.
