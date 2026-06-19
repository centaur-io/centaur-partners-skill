# REST

Use REST when MCP is not configured or the user asks for direct HTTP access and `CENTAUR_API_KEY` is available.

## Endpoint shape

- Use the matching read family under `GET https://partners.centaur.io/api/v1/*`.
- Common families include events, messages, Generated Aggregate Narrative Summaries, Generated Channel Narrative Summaries, positions, discovery, and stats.

## Header

```text
x-api-key: $CENTAUR_API_KEY
```

## Query guidance

- Use `GET /api/v1/traders` and `GET /api/v1/assets` for direct discovery when the right IDs are not known yet.
- Use `GET /api/v1/aggregate-summaries` for Generated Aggregate Narrative Summaries across eligible sources.
- Use `GET /api/v1/channel-summaries` for compact Generated Channel Narrative Summaries.
- Use `ids` on `GET /api/v1/messages` to hydrate Source Message IDs referenced by event `messageId`.
- Message rows do not expose `traderId`, and message reads do not support trader, asset, direction, or event-type filters.
- Use explicit filters rather than broad fetches when possible.
- Historical list reads use `startTime` and `endTime` when explicit bounds are needed.
- Position history reads accept `positionIds` and return time-based performance for `1D`, `7D`, and `30D`.
- For Generated Aggregate Narrative Summaries and Generated Channel Narrative Summaries, `startTime` and `endTime` select windows that overlap the requested interval. For "today" or other unqualified daily summary requests, use UTC day bounds and request `limit=50`; use up to `limit=200` for broader pages.
- List reads use forward-only cursor pagination via `cursor`.
- Stats reads return aggregate subject and summary data instead of list pages.
- Use the main Centaur docs site for the exact live endpoint, query, and response contract.
