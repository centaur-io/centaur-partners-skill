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

## Answering with Centaur data

These rules mirror the behavior the Centaur MCP server states in its `filter-guide` resource. They apply to every read family below, over MCP and REST alike.

### Ground every claim in a read

- Any claim about live or current Centaur data — positions, trades, messages, rankings, summaries, performance — must be grounded in a result returned by a Centaur read in this session.
- Do not answer data questions from general or prior knowledge, memory, or assumptions about markets, traders, or assets. Call the matching read, or say the data is unavailable.
- When an answer mixes read results with general knowledge or inference, label which parts came from a Centaur read and which did not.

### Resolve relative time before calling

Reads accept only explicit ISO-8601 `startTime` and `endTime`; none of them parse relative-time phrases such as "today" or "this week". Resolve the phrase to explicit UTC bounds yourself, anchored on the current server time. Over MCP, the `filter-guide` resource carries that timestamp on every resource read — use it as the anchor when the connected host surfaces resources. When it does not, and over REST, anchor on the current UTC time.

- "last 24h" — the rolling 24 hours ending at the anchor.
- "today" — the current UTC calendar day ending at the anchor.
- "this week" — the current UTC calendar week, starting Monday `00:00:00Z`, ending at the anchor. This is a calendar week, not a rolling 7 days.
- "last N days" — the rolling N days ending at the anchor.
- State the resolved window in the answer so the user can see what range was queried.
- If a phrase is ambiguous and these rules do not resolve it, ask one clarifying question instead of guessing a window.

Omitting both bounds may apply a bounded default history window, and an explicit `startTime` earlier than the accessible range may be clamped forward. List responses echo the applied range in `meta.appliedTimeRange`, stats responses in `filtersApplied.appliedTimeRange`; when the applied range differs from the requested one, report the applied range.

### Separate failures from empty results

- If a read fails, times out, or is rejected, do not present any claim that depends on it as confirmed.
- Never invent, estimate, or backfill data for a read that failed or returned no rows.
- Distinguish the two cases when reporting. A failed, timed-out, or rejected read means the data could not be retrieved. A successful read with no rows means no matching data was found for the requested filters and window — that is a valid result, not an error.
- A failed or partial read only qualifies the claims that depended on it. Claims backed by other successful reads in the same answer stand normally.

### Finish or qualify page walks

- List reads use forward-only cursor pagination. Pass `cursor` from `meta.nextCursor` to advance.
- A page walk is complete only when the final page's `meta.hasMore` is `false`. Otherwise qualify the answer as partial, and never present partial pages as a complete ranking, count, or total.
- Page walks are for itemized reads only. Ranking, count, and trend questions are served by `rank_traders`, `summarize_message_activity`, and the stats reads — never by paging `list_events` or `list_messages` rows to compute them.
- Ordering is fixed per read; there is no client sort parameter.
- Time-ordered reads (`list_feed`, `list_messages`, `list_events`, `list_positions`, `list_open_positions`, `list_channel_summaries`, `list_aggregate_summaries`) return newest rows first: canonical timestamp descending with `id` as the tiebreak.
- Discovery reads (`list_traders`, `list_assets`) return rows alphabetically: case-insensitive name ascending with `id` as the tiebreak.
- The aggregate reads order differently: `rank_traders` rows follow the requested metric descending, not timestamp, and `summarize_message_activity` buckets ascend by bucket start within each group.

### Host-controlled behavior

The connected client, not Centaur, owns the model and the conversation. Centaur neither selects the model nor observes conversation history.

- Do not state which model is answering unless the host has explicitly reported it. Never infer model identity from the client name, the user agent, or the fact that Centaur is connected.
- Do not claim that Centaur remembers earlier turns or persists context between sessions. Earlier turns are available only when the host supplies them.

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
- Feed event prices carry the same `quoteSymbol` unit rule as raw event rows — see [Working with events](#working-with-events).
- Some sources may be editorially excluded from the feed while remaining visible on raw reads like `list_events`; this is expected, not missing data.
- For flat event queries, counts, rankings, or system-event visibility, use `list_events` and the stats tools instead of the feed.

## Working with messages

Messages are the raw voice of each trader's source account or channel: thesis, macro thinking, sentiment, conviction, and context that cannot be derived from structured event or position data. Treat messages as a window into how traders think, not as a second source of trade data.

`list_messages` supports `sourcePlatforms` filtering — pass it for Telegram-only or X-only requests rather than filtering client-side — plus direct hydration with `ids`, time bounds, limit, and cursor. Message rows carry a nested source payload (`source.identity` for account/channel metadata, `source.preview` for display data) and expose no `traderId`; message reads have no trader, asset, direction, or event-type filters.

Source Message IDs are opaque. Use only IDs returned by message `id` or event `messageId`; never synthesize IDs from Telegram channel/message components or X account/tweet components.

Generated narrative summaries describe generated market narratives for their Source Window or Aggregate Window, returning concise server-generated market context without the full source material. Select them only when the user explicitly asks for insights, info, summaries, or similar generated narrative analysis: `list_aggregate_summaries` for cross-source market-wide narrative, `list_channel_summaries` for Source Window-specific texture.

- They are not evidence for exact trade counts, public activity rankings, or current open-position skew. Use `rank_traders` for rankings, `summarize_message_activity` for counts and trends, and the event, message, position, open-position, and stats reads for other trade facts and positioning claims.
- The word "summary" alone does not select a read: the narrative summary reads answer insight and narrative requests, while `summarize_message_activity` answers count, volume, and trend requests.
- If only generated narrative summaries are available, answer in narrative terms and avoid count, ranking, or current-positioning language.
- Both default to safe summaries; pass `includeLowSignal=true` to include low-signal windows.
- An empty page means no generated summaries for the requested window, not a failed read — fall back to `list_messages` when useful.

### Messages vs events

Trade execution details (what event happened, which position it belongs to, and how positions performed) belong to `list_events`, `list_positions`, and `list_open_positions`. Event rows expose `positionId` and `messageId`; hydrate referenced positions with `list_positions` and `positionIds`, and referenced source text with `list_messages` and `ids`. When summarizing messages, focus on the reasoning, narrative, and sentiment — the why and the worldview; direct questions about specific trades to the events and positions tools.

### Digests

Weight traders by the substance of what they said, not their message volume — compress high-frequency posters, surface original thinking, and group by theme rather than by trader. Before writing any digest, daily summary, or channel recap, read [references/message-digests.md](references/message-digests.md) for volume weighting, substance filtering, digest structure, and a worked example.

## Working with events

Events are the structured record of trade activity — opens, closes, increases, and decreases with prices, position references, source-message references, and trader attribution.

Prices on event, feed, position, and open-position rows are expressed in the row's `quoteSymbol` (for example `USDT`, `USDC`, `CAD`), the quote currency of the asset's preferred market — never assume US dollars. `quoteSymbol` is `null` when unknown; omit the unit rather than guessing one.

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
- `rank_traders` — "most active" or "best performing" questions: ranks traders server-side by `event_count`, `position_count`, `win_rate`, `avg_return`, `median_return`, or `sharpe_ratio` without requiring trader IDs. The sample defaults to UTC year-to-date and the performance evaluation window defaults to `30D`. For performance metrics, qualify thin samples using the returned scored-versus-sampled coverage. Invalid combinations return retry alternatives; choose one explicitly and disclose any retry.
- `summarize_message_activity` — "how much activity happened" questions: deterministic message/event counts, volumes, and trends, grouped by trader or overall and bucketed by hour, day, or week. It is a count read over raw data, not a generated narrative summary.
- `list_events` — trade activity in order, with compact source-message references; skip assumed and auto-generated events by default.
- `list_positions` — historical position performance and hydrating event `positionId` references. It accepts `positionIds` and returns open and closed positions with `timeBasedPerformances` for `1D`, `7D`, and `30D`; each window exposes `status` and `returnPercentage` only.
- `list_open_positions` — current exposure: what is held right now and how it is performing.
- `list_trader_stats` — aggregate per-trader metrics: win rate, average time-based return, asset focus.
- `list_asset_stats` — aggregate positioning metrics for one asset after discovery.

## References

- [references/auth.md](references/auth.md) — auth model, key provisioning, and scope notes
- [references/mcp.md](references/mcp.md) — per-tool arguments, defaults, and limits
- [references/rest.md](references/rest.md) — endpoint paths and query parameters
- [references/examples-curl.md](references/examples-curl.md) — copy-ready curl commands
- [references/client-setup.md](references/client-setup.md) — Claude, ChatGPT, Cursor, and Codex MCP setup
- [references/message-digests.md](references/message-digests.md) — weighting, filtering, and structuring message digests
