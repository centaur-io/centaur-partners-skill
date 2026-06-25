# MCP

Use MCP when the client already has Centaur configured.

## Endpoint

- `https://partners.centaur.io/mcp`

## Auth

- Official Claude and ChatGPT installs use OAuth.
- OAuth is available to active, email-verified signed-up users.

## Capability families

- events
- messages
- Generated Aggregate Narrative Summaries
- Generated Channel Narrative Summaries
- positions
- discovery
- stats

Use `list_traders` and `list_assets` as discovery tools when the user does not already know the right IDs. `list_traders` accepts `sourcePlatforms` and each trader row includes one `source` identity for the trader's platform.

Use `list_aggregate_summaries` for Generated Aggregate Narrative Summaries across eligible sources. Use `list_channel_summaries` for compact Generated Channel Narrative Summaries. If summary tools return empty pages, there may be no generated summaries for the requested window; fall back to `list_messages` when useful.

Use `list_messages` with `ids` to hydrate Source Message IDs referenced by event `messageId`. Source Message IDs are opaque; do not construct them from Telegram or X platform IDs. Message rows expose `source.identity` for account/channel metadata and `source.preview` for timestamp, URL, text, attachments, and platform flags. Message rows do not expose `traderId`, and message reads do not support trader, asset, direction, or event-type filters.

Use `list_positions` for historical position performance or to hydrate event `positionId` references. It accepts `positionIds` and returns `timeBasedPerformances` for `1D`, `7D`, and `30D`; each window exposes `status` and `returnPercentage` only.

For daily summary requests, interpret unqualified days as UTC days. Pass explicit `startTime` and `endTime` bounds such as `2026-05-09T00:00:00.000Z` through `2026-05-10T00:00:00.000Z`; aggregate and channel summary time filters select windows that overlap the requested interval. Use `limit=50` for normal daily digests, increase up to `limit=200` for broader pages, and paginate with `cursor` when needed.

## Use pattern

- Start with the narrowest matching read for the request.
- If the request depends on knowing a trader or asset first, discover it directly with `list_traders` or `list_assets` before calling detail stats.
- For platform-specific trader requests, filter `list_traders` with `sourcePlatforms` and read `source.platform` from the returned row.
- Use explicit `limit` and entity filters when they matter.
- Keep list reads paginated deliberately.
- Use the connected MCP server as the source of truth for exact tools, resources, and argument shapes.
