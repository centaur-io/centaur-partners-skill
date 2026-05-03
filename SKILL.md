---
name: centaur-partners-api
description: Use Centaur Partners API over MCP when Centaur is already configured in Claude, ChatGPT, Cursor, Codex, or Claude Code; otherwise use REST with CENTAUR_PARTNER_API_KEY or a partner API key pasted into the current chat session to fetch and summarize read-only Centaur partner data or generate correct curl commands.
---

# Centaur Partners API

Use this skill when a user wants to access Centaur partner data directly from an agent, compare MCP versus REST, or generate correct Centaur curl commands.

## Quick checks

1. Determine whether the current client already has Centaur configured as an MCP server.
2. If MCP is available, prefer MCP and use the matching Centaur read family.
3. If MCP is not available, check whether `CENTAUR_PARTNER_API_KEY` is set.
4. If the env var exists, use REST with `x-api-key`.
5. Otherwise ask whether the user wants to paste a partner API key into the current chat for one-time use.
6. If the user pastes a key, use it transiently for this session only and do not persist or echo it back.
7. If none of those paths are available, stop and give setup instructions instead of inventing credentials or unsupported flows.

## Current surface

- MCP endpoint: `https://partners.centaur.io/mcp`
- Capability families: events, messages, channel summaries, positions, discovery, and stats
- Discovery tools: `list_traders` and `list_assets`
- REST fallback uses `https://partners.centaur.io/api/v1/*`
- Preferred MCP auth: client-managed OAuth with Dynamic Client Registration when the client needs it
- MCP compatibility auth: `Authorization: Bearer <partner-api-key>`
- REST auth: `x-api-key: <partner-api-key>`

## Use MCP when available

Prefer MCP for Claude, Cursor, Codex, or any client that already has Centaur configured.

- Use the existing Centaur MCP server rather than rewriting requests as raw HTTP.
- Treat the already-configured client as the owner of the auth flow. The skill should not try to run OAuth or Dynamic Client Registration itself.
- Start with the narrowest matching read for the user's request.
- Keep list reads paginated and bounded.
- Treat the connected server as the source of truth for the exact live tool inventory and request shapes.

Read [references/mcp.md](references/mcp.md) when you need MCP-specific behavior or request shapes.

## Use REST only as fallback

If MCP is not configured, look for `CENTAUR_PARTNER_API_KEY`.

- If present, write or run curl commands against the matching `GET /api/v1/*` read family.
- Always send `x-api-key: $CENTAUR_PARTNER_API_KEY`.
- If the env var is not present but the user pastes a partner API key in chat, use that key only for the current session.
- Keep list reads bounded and paginated.
- Do not invent OAuth flows inside the skill. Either the client is already connected to Centaur MCP or the skill should fall back to REST.

Read [references/rest.md](references/rest.md) and [references/examples-curl.md](references/examples-curl.md) for concrete request patterns.

## Working with messages

Messages are the raw voice of each trader's channel — thesis, macro thinking, sentiment, conviction, and context that cannot be derived from structured event or position data. When using `list_messages`, treat messages as a window into how traders think, not as a second source of trade data.

For private Telegram channel insight, prefer generated channel summaries instead of raw messages. Use `list_channel_summaries` when the user asks for private-channel recaps, paid-channel insight, or privacy-preserving Telegram summaries. Channel summaries are generated server-side and are designed to avoid exposing raw private source messages, source message IDs, source URLs, and private attachment URLs.

### Messages vs events

Trade execution details (what was opened, closed, at what price, with what return) belong to `list_events`, `list_open_positions`, and `list_valid_closed_positions`. Do not use messages to reconstruct or re-explain trades. When summarizing messages, focus on the reasoning, narrative, and sentiment — the why and the worldview — not the what. If a user asks about specific trades, direct them to the events and positions tools instead.

### Volume awareness

Some traders post many times per day while others share one or two high-value updates. Without active management, high-frequency posters will dominate every summary.

- When a trader has posted heavily in a given window, compress their activity into a short summary noting their overall theme and conviction rather than enumerating each message.
- Avoid letting any single trader dominate the output. Prioritize diversity of voices — a summary should represent the breadth of thinking across the feed, not the depth of one trader's posting habit.
- When a trader posts a sequence of repetitive messages on the same asset, collapse the sequence into one note covering the overall behavior and price range.

### Identifying substance

Not all messages carry equal weight. Traders share a wide range of content — macro views, market structure analysis, personal reflections, conviction calls, risk management thinking, humor — and not all of it ties directly to a trade.

- Prioritize messages that contain original thinking: thesis, reasoning, invalidation levels, market structure reads, or shifts in conviction.
- Deprioritize messages that are purely mechanical: automated position mirrors from bots (rigid templates with entry price, size, notional value), templated signal alerts (ticker + entry + TP + SL + leverage with no context), or one-word confirmations ("longed", "added").
- The most valuable messages often explain why a trader changed their mind, sat out, or took an unexpected position — surface these.

### Structuring digests

When asked for a daily summary, channel recap, or "what happened yesterday":

- Lead with the most consequential macro-level observations: major directional shifts, consensus vs contrarian positioning, notable changes in sentiment.
- Group by theme or narrative rather than by trader — "thesis 1" or "thesis 2" rather than trader-by-trader recaps.
- End with quick-hit individual calls only where they add information not covered by the thematic grouping.

### Example

Given a day where Nihilus posted 20 messages about RESOLV (all variations of "longed more", "keep buying", "dips are for buying"), RunnerXBT posted 2 messages with a detailed macro short thesis citing geopolitical catalysts, and 10 other traders posted 1–3 messages each:

- Compress Nihilus into one line: "Nihilus was aggressively accumulating RESOLV all day, buying dips with high conviction and no stops mentioned."
- Surface RunnerXBT's reasoning in full since it contains original macro thinking that other traders are not expressing.
- Give each of the remaining traders space proportional to the substance of what they said, not their volume.
- Do not restate trade execution details (entry prices, sizes, P&L) — point to `list_events` for that.

## Working with events

Events are the structured record of trade activity — opens, closes, increases, and decreases with prices, positions, and trader attribution. When using `list_events`, apply these rules to present event data cleanly.

### Filtering out system events

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
- `autoGenerated` — the system created this event automatically, as a garbage-collection close for stale positions. These usually have `source: null` and no price data.
- `retrospective` — the trader posted about this after the fact, not in real time.

### Choosing the right tool

- Use `list_traders` or `list_assets` first when the user does not already know the right trader ID or asset ID.
- Use `list_events` when the user wants to see trade activity — what happened, in what order, with source context. Remember to skip assumed and auto-generated events by default.
- Use `list_valid_closed_positions` when the user wants performance data — realized returns on trades that have clean open and close prices. This tool already excludes unscoreable trades.
- Use `list_open_positions` when the user wants current exposure — what is held right now and how it is performing.
- Use `get_trader_stats` when the user wants aggregate metrics — win rate, average return, asset focus.
- Use `get_asset_stats` when the user wants aggregate metrics for one asset after discovery.
- Do not use `list_events` to manually compute win rates or ROI by scanning close events. The stats and positions tools exist precisely for that.

## When blocked

If the agent cannot find MCP config and `CENTAUR_PARTNER_API_KEY` is not set:

- say that Centaur is not configured yet
- offer a one-time path where the user can paste a partner API key into the current chat
- if the user pastes a key, do not echo it back or persist it anywhere
- point the user to [references/client-setup.md](references/client-setup.md)
- ask them to either configure MCP, export `CENTAUR_PARTNER_API_KEY`, or paste a key for the current session

## References

- [references/auth.md](references/auth.md)
- [references/mcp.md](references/mcp.md)
- [references/rest.md](references/rest.md)
- [references/examples-curl.md](references/examples-curl.md)
- [references/client-setup.md](references/client-setup.md)
