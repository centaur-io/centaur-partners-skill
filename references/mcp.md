# MCP

Per-tool arguments, defaults, and limits for the Centaur MCP server.

## Endpoint

- `https://partners.centaur.io/mcp`
- Official Claude and ChatGPT installs authenticate with OAuth (see [auth.md](auth.md)).

## Tools

`list_feed` — the activity-stream read. `limit` counts message groups (default `20`, max `100`); accepts `traderIds`, `assetIds`, `startTime`/`endTime` on message post time, forward-only `cursor` for scroll-back, and `since` for change polling (mutually exclusive with `cursor`; pass the newest `meta.nextCursor`). If `list_feed` is missing from the connected server's tool inventory or returns a missing-permission error, the connection predates the feed scope — ask the user to disconnect and reconnect the Centaur MCP server once, then retry (API keys are unaffected).

`list_traders` / `list_assets` — discovery when the right IDs are not known yet. `list_traders` accepts `sourcePlatforms`, `minTrades` (default `3`; `0` returns the full visible trader directory), and `startTime`/`endTime` (scope `tradeCount` and `minTrades` by position open time); each row includes `tradeCount` plus one `source` identity for the trader's platform. `list_trader_stats` also accepts `sourcePlatforms` and returns the same `source` shape per row.

`list_aggregate_summaries` — Generated Aggregate Narrative Summaries across eligible sources; returns source coverage counts but never trader, channel, or raw source identifiers. `list_channel_summaries` — compact Generated Channel Narrative Summaries; returns `traderId` for over-time continuity, but accepts no trader or channel filters and returns no channel identity. Both accept `startTime`, `endTime`, `limit`, `cursor`, and `includeLowSignal`. Time filters select windows that overlap the requested interval (`windowEnd > startTime` and `windowStart < endTime`). Both default to safe summaries; pass `includeLowSignal=true` to include low-signal windows.

`list_messages` — accepts `sourcePlatforms`, `ids` (hydrates Source Message IDs referenced by event `messageId`), time bounds, `limit`, and `cursor`. Rows expose `source.identity` (account/channel metadata) and `source.preview` (timestamp, URL, text, attachments, and platform flags); no `traderId`, and no trader, asset, direction, or event-type filters.

`list_positions` — accepts `positionIds` and returns `timeBasedPerformances` for `1D`, `7D`, and `30D`; each window exposes `status` and `returnPercentage` only.

`rank_traders` — accepts `metric` (`event_count` default, `position_count`, `win_rate`, `avg_return`, `median_return`, `sharpe_ratio`), optional `assetIds`, `direction`, `timeBasedPerformanceWindow` (default `30D`), `minPositions` (default `3`), `startTime`, `endTime` (default UTC year-to-date), and `limit` (default `10`, max `50`). Results are bounded with no cursor; `meta.totalCandidates` reports how many traders qualified. Performance rows expose sampled and scored coverage. Structurally invalid combinations return requested and resolved values plus valid alternatives. The tool does not alter the request. Choose an alternative, retry explicitly, and disclose the change.

`summarize_message_activity` — accepts `groupBy` (`trader` default, `none`), `interval` (`hour`, `day` default, `week`), `eventTypes`, `traderIds`, `startTime`, `endTime` (default last 7 days), and `limit` (default `10`, max `50`). Returns per-group totals plus per-bucket `messageCount`/`eventCount`, and `meta.totalMessages`/`meta.totalEvents` across all groups. Window/interval combinations above 168 buckets per group are rejected — widen the interval or narrow the window and retry.

## Resources

The server exposes its own guidance as MCP resources. Read them when the connected host surfaces resources; they are the live source of truth for behavior:

- `centaur://partners-api/filter-guide` — grounding, failure handling, history and relative-time resolution, pagination, and per-tool visibility rules. It carries no timestamp of its own; anchor relative-time phrases on `meta.serverTime` from your most recent tool result, and on your own current UTC time only before the first result.
- `centaur://partners-api/capabilities` — the server tool inventory, the read scope each tool requires, and connection and auth details. It is the same static text for every caller, not a per-key entitlement list.
