# REST

Use REST when MCP is not configured and the compatibility env var `CENTAUR_PARTNER_API_KEY` is available.

## Endpoint shape

- Use the matching read family under `GET https://partners.centaur.io/api/v1/*`.
- Common families include events, messages, aggregate summaries, channel summaries, positions, discovery, and stats.

## Header

```text
x-api-key: $CENTAUR_PARTNER_API_KEY
```

## Query guidance

- Use `GET /api/v1/traders` and `GET /api/v1/assets` for direct discovery when the right IDs are not known yet.
- Use `GET /api/v1/aggregate-summaries` for market-wide generated insight across eligible sources.
- Use `GET /api/v1/channel-summaries` for compact generated channel summaries.
- Use explicit filters rather than broad fetches when possible.
- Historical list reads use `startTime` and `endTime` when explicit bounds are needed.
- Position history reads return time-based performance for `1D`, `7D`, and `30D`.
- For aggregate and channel summaries, `startTime` and `endTime` select windows that overlap the requested interval. For "today" or other unqualified daily summary requests, use UTC day bounds and request `limit=50`; use up to `limit=200` for broader pages.
- List reads use forward-only cursor pagination via `cursor`.
- Stats reads return aggregate subject and summary data instead of list pages.
- Use the main Centaur docs site for the exact live endpoint, query, and response contract.
