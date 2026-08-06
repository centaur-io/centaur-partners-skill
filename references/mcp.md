# MCP

Use MCP when the client already has Centaur configured.

## Endpoint

- `https://partners.centaur.io/mcp`

## Auth

- Official Claude and ChatGPT installs use OAuth.
- OAuth is available to active, email-verified signed-up users.

## Capability families

- the feed
- events
- messages
- Generated Aggregate Narrative Summaries
- Generated Channel Narrative Summaries
- positions
- discovery
- stats
- trader rankings
- activity summaries

Use `list_feed` for activity-stream requests. It returns presentation-ready source-message groups ordered by message post time, each with the trader summary, source preview, and curated events carrying direction and embedded asset context — no hydration calls needed. `limit` counts groups (default `20`, max `100`); accepts `traderIds`, `assetIds`, `startTime`/`endTime` on message post time, and forward-only `cursor`. For polling, pass the newest `meta.nextCursor` as `since` (mutually exclusive with `cursor`): results are whole-group upserts keyed by group `id` — an older message that gains a late-recorded event returns again with all its current events, so replace, never append. Keep the latest non-null `nextCursor` between polls, including from empty polls. Feed rows are pre-curated; a remaining `assumed: true` event is a deliberately retained inferred event and safe to present.

If `list_feed` is missing from the connected server's tool inventory or returns a missing-permission error, the connection predates the feed scope — ask the user to disconnect and reconnect the Centaur MCP server once, then retry.

Use `list_traders` and `list_assets` as discovery tools when the user does not already know the right IDs. `list_traders` accepts `sourcePlatforms`, `minTrades`, `startTime`, and `endTime`, and each trader row includes `tradeCount` plus one `source` identity for the trader's platform. `minTrades` defaults to `3`; pass `minTrades=0` when the user needs the full visible trader directory. `startTime` and `endTime` scope `tradeCount` and `minTrades` by position open time. `list_trader_stats` also accepts `sourcePlatforms` and returns the same source shape per trader row.

Use `list_aggregate_summaries` for Generated Aggregate Narrative Summaries across eligible sources. Use `list_channel_summaries` for compact Generated Channel Narrative Summaries. If summary tools return empty pages, there may be no generated summaries for the requested window; fall back to `list_messages` when useful.

Use `list_messages` with `sourcePlatforms` for Telegram-only or X-only source-message requests, and with `ids` to hydrate Source Message IDs referenced by event `messageId`. Source Message IDs are opaque; do not construct them from Telegram or X platform IDs. Message rows expose `source.identity` for account/channel metadata and `source.preview` for timestamp, URL, text, attachments, and platform flags. Message rows do not expose `traderId`, and message reads do not support trader, asset, direction, or event-type filters.

Use `list_positions` for historical position performance or to hydrate event `positionId` references. It accepts `positionIds` and returns `timeBasedPerformances` for `1D`, `7D`, and `30D`; each window exposes `status` and `returnPercentage` only.

Use `rank_traders` for "most active" or "best performing" trader questions instead of paging raw events. It accepts `metric` (`event_count` default, `position_count`, `win_rate`, `avg_return`, `median_return`, `sharpe_ratio`), optional `assetIds`, `direction`, `timeBasedPerformanceWindow` (default `7D`), `minPositions` (default `3`), `startTime`, `endTime` (default last 7 days), and `limit` (default 10, max 50). Results are bounded with no cursor; `meta.totalCandidates` reports how many traders qualified.

Performance metrics evaluate positions opened in the window at the chosen horizon: a position opened at time T is evaluated at T plus the window, so `startTime` must be at least the evaluation window before now. Requests with a more recent `startTime` are rejected with a corrective error — widen the window start or use a shorter `timeBasedPerformanceWindow` and retry. For example, a `30D` ranking needs `startTime` at least 30 days in the past.

Use `summarize_message_activity` for message/event count, volume, and trend questions instead of paging `list_messages` or `list_events`. It accepts `groupBy` (`trader` default, `none`), `interval` (`hour`, `day` default, `week`), `eventTypes`, `traderIds`, `startTime`, `endTime` (default last 7 days), and `limit` (default 10, max 50). It returns per-group totals plus per-bucket `messageCount`/`eventCount`, and `meta.totalMessages`/`meta.totalEvents` across all groups. It is a deterministic count read, not a generated narrative summary. Window/interval combinations above 168 buckets per group are rejected; widen the interval or narrow the window and retry.

For daily summary requests, interpret unqualified days as UTC days. Pass explicit `startTime` and `endTime` bounds such as `2026-05-09T00:00:00.000Z` through `2026-05-10T00:00:00.000Z`; aggregate and channel summary time filters select windows that overlap the requested interval. Use `limit=50` for normal daily digests, increase up to `limit=200` for broader pages, and paginate with `cursor` when needed.

## Use pattern

- Start with the narrowest matching read for the request.
- If the request depends on knowing a trader or asset first, discover it directly with `list_traders` or `list_assets` before calling detail stats.
- For platform-specific trader requests, filter `list_traders` or `list_trader_stats` with `sourcePlatforms` and read `source.platform` from the returned row.
- For active-trader discovery requests, filter `list_traders` with `minTrades`; include `startTime` and `endTime` for bounded periods, and read `tradeCount` from the returned rows.
- For platform-specific source-message requests, filter `list_messages` with `sourcePlatforms`.
- Use explicit `limit` and entity filters when they matter.
- Keep list reads paginated deliberately. Do not page raw lists for ranking, count, or trend questions — use `rank_traders` or `summarize_message_activity` instead.
- Use the connected MCP server as the source of truth for exact tools, resources, and argument shapes.
