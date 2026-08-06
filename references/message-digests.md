# Message digests

How to weight, filter, and structure a digest, daily summary, or channel recap built from Centaur messages and generated summaries.

## Volume weighting

Some traders post many times per day while others share one or two high-value updates. Left unmanaged, high-frequency posters dominate every summary.

- Compress a heavy poster's window into a short note covering their overall theme and conviction rather than enumerating each message.
- Collapse a repetitive sequence on one asset into a single note covering the overall behavior and price range.
- Represent the breadth of thinking across the feed: give each trader space proportional to the substance of what they said, not their volume.

## Identifying substance

- Prioritize messages with original thinking: thesis, reasoning, invalidation levels, market structure reads, or shifts in conviction.
- The most valuable messages often explain why a trader changed their mind, sat out, or took an unexpected position — surface these.
- Deprioritize purely mechanical content: automated position mirrors from bots (rigid templates with entry price, size, notional value), templated signal alerts (ticker + entry + TP + SL + leverage with no context), and one-word confirmations ("longed", "added").

## Structuring the digest

- Build market-wide daily digests from Generated Aggregate Narrative Summaries (`list_aggregate_summaries`) first; use Generated Channel Narrative Summaries (`list_channel_summaries`) for channel-level / Source Window texture or when aggregate summaries are unavailable.
- Interpret "today" and other unqualified calendar-day requests as UTC days. Pass explicit UTC bounds as `startTime` and `endTime` (for example `2026-05-09T00:00:00.000Z` through `2026-05-10T00:00:00.000Z`), using the next UTC midnight as the exclusive end bound; summary time filters select windows that overlap the requested interval.
- Request `limit=50` for daily digests; use up to `limit=200` and paginate with `cursor` when more summaries are needed.
- Lead with the most consequential macro observations: major directional shifts, consensus vs contrarian positioning, notable sentiment changes.
- Group by theme or narrative rather than by trader; end with quick-hit individual calls only where they add information the themes did not cover.
- Point to `list_events` for trade execution details (entry prices, sizes, P&L) instead of restating them.

## Worked example

Given a day where Nihilus posted 20 messages about RESOLV (all variations of "longed more", "keep buying", "dips are for buying"), RunnerXBT posted 2 messages with a detailed macro short thesis citing geopolitical catalysts, and 10 other traders posted 1–3 messages each:

- Compress Nihilus into one line: "Nihilus was aggressively accumulating RESOLV all day, buying dips with high conviction and no stops mentioned."
- Surface RunnerXBT's reasoning in full — it contains original macro thinking no other trader is expressing.
- Give each remaining trader space proportional to the substance of what they said.
