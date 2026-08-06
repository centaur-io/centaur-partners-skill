---
name: centaur-api
description: Use the Centaur API over MCP when it is already configured in Claude, ChatGPT, Cursor, Codex, or Claude Code; use REST with CENTAUR_API_KEY or a Centaur API key pasted into the current chat session to fetch and summarize read-only Centaur data or generate correct curl commands.
---

# Centaur API

Use this skill when a user wants to access Centaur data directly from an agent, compare Centaur MCP versus REST, or generate correct curl commands against the Centaur API.

## Quick checks

1. Determine whether the current client already has Centaur configured as an MCP server.
2. If MCP is available, prefer MCP and use the matching Centaur read family.
3. If MCP is not available or the user asks for REST, check whether `CENTAUR_API_KEY` is set.
4. If the env var exists, use REST with `x-api-key`.
5. Otherwise ask whether the user wants to paste a Centaur API key into the current chat for one-time use.
6. If the user pastes a key, use it transiently for this session only and do not persist or echo it back.
7. If none of those paths are available, stop and give setup instructions instead of inventing credentials or unsupported flows.

## Current surface

- MCP endpoint: `https://partners.centaur.io/mcp`
- Capability families: the feed, events, messages, Generated Aggregate Narrative Summaries, Generated Channel Narrative Summaries, positions, discovery, stats, trader rankings, and activity summaries
- Feed tool: `list_feed` returns presentation-ready source-message groups with curated trade events; it is the default read for "what is happening" and activity-stream requests
- Discovery tools: `list_traders` and `list_assets`; `list_traders` can filter by `sourcePlatforms`, `minTrades`, `startTime`, and `endTime`
- Message tools: `list_messages` can filter source messages by `sourcePlatforms`
- Aggregate tools: `rank_traders` ranks traders by activity or performance without trader IDs; `summarize_message_activity` returns deterministic message/event counts and trends
- REST uses `https://partners.centaur.io/api/v1/*`
- Official Claude and ChatGPT installs use OAuth.
- OAuth is available to active, email-verified signed-up users.
- API keys are self-serve REST credentials.
- REST auth: `x-api-key: <api-key>`
- Generated Aggregate Narrative Summaries and Generated Channel Narrative Summaries are standard read families.

## Use MCP when available

Prefer MCP for Claude, Cursor, Codex, or any client that already has Centaur configured.

- Use the existing Centaur MCP server rather than rewriting requests as raw HTTP.
- Treat the already-configured client as the owner of the auth flow. The skill should not try to run OAuth or Dynamic Client Registration itself.
- Start with the narrowest matching read for the user's request.
- Keep list reads paginated and bounded.
- Treat the connected server as the source of truth for the exact live tool inventory and request shapes.

Read [references/mcp.md](references/mcp.md) when you need MCP-specific behavior or request shapes.

## Use REST for direct HTTP access

If MCP is not configured or the user asks for REST, look for `CENTAUR_API_KEY`.

- If present, write or run curl commands against the matching `GET /api/v1/*` read family.
- Always send `x-api-key: $CENTAUR_API_KEY`.
- If the env var is not present but the user pastes a Centaur API key in chat, use that key only for the current session.
- Keep list reads bounded and paginated.
- Do not invent OAuth flows inside the skill. Either the client is already connected to Centaur MCP or the skill should use REST.

Read [references/rest.md](references/rest.md) and [references/examples-curl.md](references/examples-curl.md) for concrete request patterns.

## Working with discovery

Use `list_traders` or `GET /api/v1/traders` when a request depends on resolving a trader ID before stats or detail reads. Trader discovery rows include `tradeCount` plus one `source` identity for the trader's platform, with `platform`, handle, display name, profile URL, avatar URL, and audience count when available. Use `minTrades` to require a minimum eligible visible position count; it defaults to `3`, and `minTrades=0` returns the full visible trader directory. Use `startTime` and `endTime` when the user asks for active traders in a bounded period; those bounds scope `tradeCount` and `minTrades` by position open time. Trader stats rows expose the same `source` shape.

Each trader has one source platform. Use `sourcePlatforms` to filter trader discovery or trader stats to `TELEGRAM`, `X`, or both, then read the returned row's `source.platform` before answering platform-specific questions.

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

Messages are the raw voice of each trader's source account or channel: thesis, macro thinking, sentiment, conviction, and context that cannot be derived from structured event or position data. When using `list_messages`, treat messages as a window into how traders think, not as a second source of trade data.

`list_messages` supports source-platform filtering with `sourcePlatforms`, direct source-message hydration with `ids`, plus time bounds, limit, and cursor. For Telegram-only or X-only source-message requests, pass `sourcePlatforms` to the tool rather than fetching all platforms and filtering client-side. It does not expose `traderId` on message rows and does not support trader, asset, direction, or event-type filters.

Source Message IDs are opaque. Use IDs returned by message `id` or event `messageId`; never synthesize IDs from Telegram channel/message components or X account/tweet components.

Message rows use a nested source payload. Read source account/channel metadata from `source.identity` and message display data from `source.preview`. Do not expect old flat `url`, `text`, `attachments`, or `originalCreatedAt` fields on the message row.

For market-wide insight, prefer Generated Aggregate Narrative Summaries when `list_aggregate_summaries` is available. Use Generated Channel Narrative Summaries for Source Window-specific texture when `list_channel_summaries` is available. If summary tools return empty pages, there may be no generated summaries for the requested window; fall back to `list_messages` when useful. Summaries are generated server-side and return concise market context without the full source material.

### Messages vs events

Trade execution details (what event happened, which position it belongs to, and how positions performed) belong to `list_events`, `list_positions`, and `list_open_positions`. Event rows expose `positionId` and `messageId`; use `list_positions` with `positionIds` when you need to hydrate referenced position details or time-based performance, and `list_messages` with `ids` when you need referenced source-message text. Do not use messages to reconstruct or re-explain trades. When summarizing messages, focus on the reasoning, narrative, and sentiment — the why and the worldview — not the what. If a user asks about specific trades, direct them to the events and positions tools instead.

### Digests

Weight traders by the substance of what they said, not their message volume — compress high-frequency posters, surface original thinking, and group by theme rather than by trader. Before writing any digest, daily summary, or channel recap, read [references/message-digests.md](references/message-digests.md) for volume weighting, substance filtering, digest structure, and a worked example.

## Working with events

Events are the structured record of trade activity — opens, closes, increases, and decreases with prices, position references, source-message references, and trader attribution. When using `list_events`, apply these rules to present event data cleanly.

### Filtering out system events

These rules apply to `list_events`, which returns the raw record. Feed rows are already curated server-side — present them as returned.

Each event carries three boolean flags: `assumed`, `retrospective`, and `autoGenerated`. These are internal metadata — they should not be visible to the user by default.

**Default behavior — hide and skip:**

- When presenting events (e.g. "show me the last 10 trades for RunnerXBT"), skip events where `assumed: true` or `autoGenerated: true`. These are system-generated artifacts, not trader actions. Present only genuine trader-initiated events.
- If skipping these events means fewer results than the user asked for, fetch more from the API to fill the requested count with real events.
- Never mention `assumed`, `retrospective`, or `autoGenerated` as attributes in event listings. Do not say "not assumed" or "not auto-generated" — these flags do not exist from the user's perspective.
- The `retrospective` flag should never be surfaced, even when true. Whether a trader posted in real time or after the fact is not relevant to the user unless they specifically ask about it.

**Only surface if explicitly asked:**

- If a user specifically asks to see assumed closes, auto-generated events, or system events — then include them and explain what they are.
- If a user asks "why are there gaps" or "where are the missing closes" — then explain that some closes are system-inferred and are hidden by default, and offer to show them.
- If a user asks about retrospective trades or timing of posts vs trades — then the `retrospective` flag becomes relevant and can be mentioned.

**What each flag means (for your understanding, not for presenting to users):**

- `assumed` — the system inferred this event. Common case: a trader flips from long to short, so the system assumes the prior long was closed.
- `autoGenerated` — the system created this event automatically, as a garbage-collection close for stale positions. These usually have `messageId: null` and no price data.
- `retrospective` — the trader posted about this after the fact, not in real time.

### Choosing the right tool

- Use `list_feed` when the user wants an activity stream — what is happening, message-grouped, ready to present, with system events already removed. Prefer it over composing `list_events` + `list_messages` + discovery calls for feed-style requests.
- Use `list_traders` or `list_assets` first when the user does not already know the right trader ID or asset ID.
- Use `rank_traders` when the user asks who is most active or best performing — it ranks traders server-side by `event_count`, `position_count`, `win_rate`, `avg_return`, `median_return`, or `sharpe_ratio` over an explicit time window (default last 7 days) without requiring trader IDs. Do not page through raw events to build rankings. For performance metrics, qualify thin samples using the returned `timeBasedPerformance.positionsCount`.
- Use `summarize_message_activity` when the user asks how much activity happened — message/event counts, volumes, or trends — grouped by trader or overall and bucketed by hour, day, or week. Its counts are deterministic evidence over raw data; it is not a generated narrative summary. Do not page through `list_messages` or `list_events` to count activity.
- Use `list_events` when the user wants to see trade activity — what happened, in what order, with compact source-message references. Remember to skip assumed and auto-generated events by default.
- Use `list_positions` when the user wants historical position performance or needs to hydrate event `positionId` references. It accepts `positionIds` and returns open and closed positions with `timeBasedPerformances` for `1D`, `7D`, and `30D`; each window exposes `status` and `returnPercentage` only.
- Use `list_open_positions` when the user wants current exposure — what is held right now and how it is performing.
- Use `list_trader_stats` when the user wants aggregate metrics — win rate, average time-based return, asset focus.
- Use `list_asset_stats` when the user wants aggregate positioning metrics for one asset after discovery.
- Do not use `list_events` to manually compute win rates, ROI, rankings, or activity counts by scanning rows. The stats, positions, ranking, and activity-summary tools exist precisely for that.

## When blocked

If the agent cannot find MCP config and `CENTAUR_API_KEY` is not set:

- say that Centaur is not configured yet
- offer a one-time path where the user can paste a Centaur API key into the current chat
- if the user pastes a key, do not echo it back or persist it anywhere
- point the user to [references/client-setup.md](references/client-setup.md)
- ask them to either configure MCP, export `CENTAUR_API_KEY`, or paste a key for the current session

## References

- [references/auth.md](references/auth.md)
- [references/mcp.md](references/mcp.md)
- [references/rest.md](references/rest.md)
- [references/examples-curl.md](references/examples-curl.md)
- [references/client-setup.md](references/client-setup.md)
