# REST

Endpoint paths and query parameters for direct HTTP access.

## Endpoint shape

- Use the matching read family under `GET https://partners.centaur.io/api/v1/*`.

## Header

```text
x-api-key: $CENTAUR_API_KEY
```

## Query guidance

- `GET /api/v1/feed` — the activity-stream read. `limit` counts message groups (default `20`, max `100`); accepts `traderIds`, `assetIds`, `startTime`/`endTime` on message post time, `cursor` for scroll-back, and `since` for change polling (mutually exclusive with `cursor`; pass the newest `meta.nextCursor`).
- `GET /api/v1/traders` and `GET /api/v1/assets` — discovery when the right IDs are not known yet.
- `GET /api/v1/traders` accepts `sourcePlatforms`, `minTrades` (default `3`; `0` returns the full visible trader directory), and `startTime`/`endTime` (scope `tradeCount` and `minTrades` by position open time); each row includes `tradeCount` plus one `source` identity for the trader's platform.
- `GET /api/v1/traders/stats` also accepts `sourcePlatforms` and returns one `source` identity per trader stats row.
- `GET /api/v1/aggregate-summaries` — Generated Aggregate Narrative Summaries across eligible sources. `GET /api/v1/channel-summaries` — compact Generated Channel Narrative Summaries. Both accept `includeLowSignal` (default safe summaries only); `startTime`/`endTime` select windows that overlap the requested interval.
- `GET /api/v1/messages` accepts `sourcePlatforms`, `ids` (hydrates Source Message IDs referenced by event `messageId`), time bounds, `limit`, and `cursor`. Rows expose `source.identity` (account/channel metadata) and `source.preview` (timestamp, URL, text, attachments, and platform flags); no `traderId`, and no trader, asset, direction, or event-type filters.
- `GET /api/v1/positions` accepts `positionIds` and returns time-based performance for `1D`, `7D`, and `30D`.
- `GET /api/v1/traders/rankings` — trader rankings: `metric` (`event_count` default, `position_count`, `win_rate`, `avg_return`, `median_return`, `sharpe_ratio`), optional `assetIds`, `direction`, `timeBasedPerformanceWindow` (default `30D`), `minPositions` (default `3`), UTC bounds (default year-to-date), and `limit` (default `10`, max `50`). Results are bounded with no cursor. Performance rows expose sampled and scored coverage. Structurally invalid combinations return `422` (`PERFORMANCE_WINDOW_NOT_ELAPSED`) with requested and resolved values plus valid retry alternatives; the server does not substitute or retry.
- `GET /api/v1/activity-summaries` — deterministic message/event counts and trends: `groupBy` (`trader` default, `none`), `interval` (`hour`, `day` default, `week`), `eventTypes`, `traderIds`, explicit UTC bounds (default last 7 days), and `limit` (default `10`, max `50`). Requests above 168 buckets per group return `422`.
- List reads use forward-only cursor pagination via `cursor`; historical list reads take `startTime` and `endTime`. A page walk is complete only when the final page's `meta.hasMore` is `false`.
- Every list and stats response carries `meta.serverTime`, the server's current UTC time when the response was produced (ISO-8601). List responses echo the applied range in `meta.appliedTimeRange`, stats responses in `filtersApplied.appliedTimeRange`.
- Reads accept only explicit ISO-8601 bounds and do not parse relative-time phrases. Resolve those to explicit UTC bounds before calling, anchored on `meta.serverTime` from your most recent response, or on the current UTC time before the first one.
- Prices are expressed in the row's `quoteSymbol`, which is `null` when unknown; never assume US dollars.
- Stats reads return aggregate subject and summary data instead of list pages.
- The Centaur docs site (`https://partners.centaur.io/docs`) is the source of truth for the exact live endpoint, query, and response contract.
