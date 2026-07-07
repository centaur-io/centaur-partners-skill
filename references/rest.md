# REST

Use REST when MCP is not configured or the user asks for direct HTTP access and `CENTAUR_API_KEY` is available.

## Endpoint shape

- Use the matching read family under `GET https://partners.centaur.io/api/v1/*`.
- Common families include events, messages, Generated Aggregate Narrative Summaries, Generated Channel Narrative Summaries, positions, discovery, stats, trader rankings, and activity summaries.

## Header

```text
x-api-key: $CENTAUR_API_KEY
```

## Query guidance

- Use `GET /api/v1/traders` and `GET /api/v1/assets` for direct discovery when the right IDs are not known yet.
- `GET /api/v1/traders` accepts `sourcePlatforms` and `minTrades`, and each trader row includes `tradeCount` plus one `source` identity for the trader's platform.
- `minTrades` defaults to `3`; pass `minTrades=0` when the user needs the full visible trader directory.
- `GET /api/v1/traders` also accepts `startTime` and `endTime` to scope `tradeCount` and `minTrades` by position open time.
- `GET /api/v1/traders/stats` also accepts `sourcePlatforms` and returns one `source` identity per trader stats row.
- Use `GET /api/v1/aggregate-summaries` for Generated Aggregate Narrative Summaries across eligible sources.
- Use `GET /api/v1/channel-summaries` for compact Generated Channel Narrative Summaries.
- Use `sourcePlatforms` on `GET /api/v1/messages` for Telegram-only or X-only source-message requests.
- Use `ids` on `GET /api/v1/messages` to hydrate Source Message IDs referenced by event `messageId`.
- Source Message IDs are opaque; do not construct them from Telegram or X platform IDs.
- Message rows expose `source.identity` for account/channel metadata and `source.preview` for timestamp, URL, text, attachments, and platform flags.
- Message rows do not expose `traderId`, and message reads do not support trader, asset, direction, or event-type filters.
- Use explicit filters rather than broad fetches when possible.
- Historical list reads use `startTime` and `endTime` when explicit bounds are needed.
- Position history reads accept `positionIds` and return time-based performance for `1D`, `7D`, and `30D`.
- For Generated Aggregate Narrative Summaries and Generated Channel Narrative Summaries, `startTime` and `endTime` select windows that overlap the requested interval. For "today" or other unqualified daily summary requests, use UTC day bounds and request `limit=50`; use up to `limit=200` for broader pages.
- List reads use forward-only cursor pagination via `cursor`.
- Stats reads return aggregate subject and summary data instead of list pages.
- Use `GET /api/v1/traders/rankings` for trader ranking questions: `metric` (`event_count` default, `position_count`, `win_rate`, `avg_return`, `median_return`, `sharpe_ratio`), optional `assetIds`, `direction`, `timeBasedPerformanceWindow` (default `7D`), `minPositions` (default `3`), explicit UTC bounds (default last 7 days), and `limit` (default 10, max 50). Results are bounded with no cursor.
- Performance-metric rankings evaluate positions opened in the window at the chosen horizon, so `startTime` must be at least the evaluation window before now; more recent windows return `422` (`PERFORMANCE_WINDOW_NOT_ELAPSED`) with a corrective message.
- Use `GET /api/v1/activity-summaries` for deterministic message/event counts and trends: `groupBy` (`trader` default, `none`), `interval` (`hour`, `day` default, `week`), `eventTypes`, `traderIds`, explicit UTC bounds (default last 7 days), and `limit` (default 10, max 50). Requests above 168 buckets per group return `422`.
- Do not page raw event or message lists to build rankings or counts; use the rankings and activity-summary reads instead.
- Use the main Centaur docs site for the exact live endpoint, query, and response contract.
