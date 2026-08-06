---
name: centaur-api
description: Read-only Centaur trading data (partners.centaur.io) - the activity feed, traders, trade events, positions, source messages, and generated summaries. Use when fetching or summarizing Centaur data over a configured Centaur MCP server or via REST with a Centaur API key, when generating Centaur API curl commands, or when a client needs Centaur MCP setup.
---

# Centaur API

Read-only Centaur trading data over MCP (`https://partners.centaur.io/mcp`) or REST (`GET https://partners.centaur.io/api/v1/*`). Read families: the feed, events, messages, Generated Aggregate Narrative Summaries, Generated Channel Narrative Summaries, positions, discovery, stats, trader rankings, and activity summaries.

## Choosing access

1. If the current client already has Centaur configured as an MCP server, use MCP with the matching read family.
2. Otherwise, if `CENTAUR_API_KEY` is set, use REST with the `x-api-key` header.
3. Otherwise, offer a one-time path: the user pastes a Centaur API key into the chat and you use it for this session only — never echo or persist it.
4. If none of those paths exist, say Centaur is not configured yet, point to [references/client-setup.md](references/client-setup.md), and stop rather than inventing credentials or auth flows.

Read [references/auth.md](references/auth.md) for the auth model: OAuth availability, key provisioning, and scope notes.

## Use MCP when available

Prefer the configured Centaur MCP server over rewriting requests as raw HTTP. The client owns the auth flow — never run OAuth or Dynamic Client Registration from the skill. Start with the narrowest matching read, keep list reads bounded with explicit `limit` and pagination, and treat the connected server as the source of truth for the live tool inventory and request shapes.

Read [references/mcp.md](references/mcp.md) for per-tool arguments, defaults, and limits.

## Use REST for direct HTTP access

If MCP is not configured or the user asks for REST, call the matching `GET /api/v1/*` read family with `x-api-key: $CENTAUR_API_KEY` (or the pasted session key) on every request.

Read [references/rest.md](references/rest.md) for endpoint paths and query parameters, and [references/examples-curl.md](references/examples-curl.md) for copy-ready curl commands.

## Working with discovery

Use `list_traders` or `GET /api/v1/traders` when a request depends on resolving a trader ID before stats or detail reads. `minTrades` requires a minimum eligible visible position count and defaults to `3` — pass `minTrades=0` for the full visible trader directory. `startTime` and `endTime` scope `tradeCount` and `minTrades` by position open time for "active traders in a bounded period" requests.

Each trader has one source platform. Filter trader discovery or trader stats with `sourcePlatforms` (`TELEGRAM`, `X`), then read the returned row's `source.platform` before answering platform-specific questions.

## Working with the feed

The feed is the presentation-ready view of recent trading activity: source-message groups ordered by message post time, each carrying the trader summary, the source-message preview, and curated trade events with embedded asset context and direction. Use `list_feed` or `GET /api/v1/feed` when the user asks what is happening, wants an activity stream, or wants to follow new activity over time — one feed call replaces composing events, messages, traders, assets, and positions.

- Feed rows are curated server-side: fabricated system events (assumed closes, garbage-collected closes, and bookkeeping or duplicate-story assumed opens) are already removed. Do not re-apply the event-flag filtering rules from the events section below; present feed groups as returned.
- A feed event with `assumed: true` is a deliberately retained inferred event (for example an inferred open). It is safe to present; do not mention the flag unless the user asks.
- `limit` counts message groups (default `20`, max `100`). Scroll back with `cursor`.
- Poll for new activity with `since` (mutually exclusive with `cursor`): pass the newest `meta.nextCursor`. Results are whole-group upserts — replace any previously seen group by its `id`, because a group returns with all of its current events, including when an older message gains a late-recorded event. Keep the latest non-null `nextCursor` between polls, including from empty polls.
- Filters: `traderIds` and `assetIds`, plus `startTime`/`endTime` on message post time.
- Some sources may be editorially excluded from the feed while remaining visible on raw reads like `list_events`; this is expected, not missing data.
- For flat event queries, counts, rankings, or system-event visibility, use `list_events` and the stats tools instead of the feed.

## Working with messages

Messages are the raw voice of each trader's source account or channel: thesis, macro thinking, sentiment, conviction, and context that cannot be derived from structured event or position data. Treat messages as a window into how traders think, not as a second source of trade data.

`list_messages` supports `sourcePlatforms` filtering — pass it for Telegram-only or X-only requests rather than filtering client-side — plus direct hydration with `ids`, time bounds, limit, and cursor. Message rows carry a nested source payload (`source.identity` for account/channel metadata, `source.preview` for display data) and expose no `traderId`; message reads have no trader, asset, direction, or event-type filters.

Source Message IDs are opaque. Use only IDs returned by message `id` or event `messageId`; never synthesize IDs from Telegram channel/message components or X account/tweet components.

For market-wide insight, prefer Generated Aggregate Narrative Summaries (`list_aggregate_summaries`); use Generated Channel Narrative Summaries (`list_channel_summaries`) for Source Window-specific texture. They return concise server-generated market context without the full source material. An empty page means no generated summaries for the requested window — fall back to `list_messages` when useful.

### Messages vs events

Trade execution details (what event happened, which position it belongs to, and how positions performed) belong to `list_events`, `list_positions`, and `list_open_positions`. Event rows expose `positionId` and `messageId`; hydrate referenced positions with `list_positions` and `positionIds`, and referenced source text with `list_messages` and `ids`. When summarizing messages, focus on the reasoning, narrative, and sentiment — the why and the worldview; direct questions about specific trades to the events and positions tools.

### Digests

Weight traders by the substance of what they said, not their message volume — compress high-frequency posters, surface original thinking, and group by theme rather than by trader. Before writing any digest, daily summary, or channel recap, read [references/message-digests.md](references/message-digests.md) for volume weighting, substance filtering, digest structure, and a worked example.

## Working with events

Events are the structured record of trade activity — opens, closes, increases, and decreases with prices, position references, source-message references, and trader attribution.

### Filtering out system events

These rules apply to `list_events`, which returns the raw record; feed rows are already curated — present them as returned. Each event carries three boolean flags, internal metadata that stays invisible to the user by default:

- `assumed` — the system inferred the event. Common case: a trader flips from long to short, so the system assumes the prior long was closed.
- `autoGenerated` — the system created the event automatically, as a garbage-collection close for stale positions; these usually have `messageId: null` and no price data.
- `retrospective` — the trader posted about the event after the fact, not in real time.

When presenting events, include only genuine trader-initiated events: skip rows where `assumed: true` or `autoGenerated: true`, and fetch more pages if that leaves fewer results than the user asked for. Present the remaining events without mentioning the flags — from the user's perspective they do not exist, so phrases like "not assumed" or "not auto-generated" never appear.

Surface hidden events only on an explicit ask: if the user asks for assumed closes, system events, or why there are gaps or missing closes, explain that some closes are system-inferred and hidden by default, and offer to show them. Mention `retrospective` (posted in real time vs after the fact) only when the user asks about post timing.

### Choosing the right tool

- `list_feed` — activity-stream requests: message-grouped, presentation-ready, system events already removed. Prefer it over composing `list_events` + `list_messages` + discovery calls.
- `list_traders` / `list_assets` — first, when the right trader or asset ID is not known yet.
- `rank_traders` — "most active" or "best performing" questions: ranks traders server-side by `event_count`, `position_count`, `win_rate`, `avg_return`, `median_return`, or `sharpe_ratio` over an explicit time window (default last 7 days) without requiring trader IDs. For performance metrics, qualify thin samples using the returned `timeBasedPerformance.positionsCount`.
- `summarize_message_activity` — "how much activity happened" questions: deterministic message/event counts, volumes, and trends, grouped by trader or overall and bucketed by hour, day, or week. It is a count read over raw data, not a generated narrative summary.
- `list_events` — trade activity in order, with compact source-message references; skip assumed and auto-generated events by default.
- `list_positions` — historical position performance and hydrating event `positionId` references. It accepts `positionIds` and returns open and closed positions with `timeBasedPerformances` for `1D`, `7D`, and `30D`; each window exposes `status` and `returnPercentage` only.
- `list_open_positions` — current exposure: what is held right now and how it is performing.
- `list_trader_stats` — aggregate per-trader metrics: win rate, average time-based return, asset focus.
- `list_asset_stats` — aggregate positioning metrics for one asset after discovery.

Ranking, count, and trend questions are served by `rank_traders`, `summarize_message_activity`, and the stats tools — never by paging `list_events` or `list_messages` rows to compute them.

## References

- [references/auth.md](references/auth.md) — auth model, key provisioning, and scope notes
- [references/mcp.md](references/mcp.md) — per-tool arguments, defaults, and limits
- [references/rest.md](references/rest.md) — endpoint paths and query parameters
- [references/examples-curl.md](references/examples-curl.md) — copy-ready curl commands
- [references/client-setup.md](references/client-setup.md) — Claude, ChatGPT, Cursor, and Codex MCP setup
- [references/message-digests.md](references/message-digests.md) — weighting, filtering, and structuring message digests
