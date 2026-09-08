# Centaur API Skill

Public installable skill for the Centaur API.

## Install

```bash
npx skills add https://github.com/centaur-io/centaur-partners-skill
```

## What it covers

- Prefer Centaur MCP when the client already has Centaur configured.
- Use REST when `CENTAUR_API_KEY` is available or when the user provides a one-time key in chat.
- Help with the broader read-only trading data surface across the feed, events, messages, Generated Aggregate Narrative Summaries, Generated Channel Narrative Summaries, positions, discovery, stats, trader rankings, and activity summaries.
- Support direct trader and asset discovery before detail stats calls.
- Generate correct curl commands for the matching `GET /api/v1/*` read family, not only events.
- Guide Claude, ChatGPT, Cursor, and Codex setup for the remote Centaur MCP server.

## Current contract at a glance

- REST surface: `https://partners.centaur.io/api/v1/*`
- MCP endpoint: `https://partners.centaur.io/mcp`
- Time-bounded reads use `startTime` and `endTime`
- List-style reads use forward-only cursor pagination
- Official Claude and ChatGPT installs use OAuth
- OAuth is available to active, email-verified signed-up users
- API keys are self-serve REST credentials
- Generated Aggregate Narrative Summaries and Generated Channel Narrative Summaries are standard read families
- The feed (`GET /api/v1/feed`, `list_feed`) returns presentation-ready source-message groups with curated events; `since` polls for changes with whole-group upserts
- Summary tools may return empty pages when no generated summaries match the requested window; an empty page is a valid result, not a failed read
- Reads accept only explicit ISO-8601 `startTime`/`endTime` and never parse relative-time phrases
- A page walk is complete only when the final page's `meta.hasMore` is `false`
- Prices are expressed in the row's `quoteSymbol` and are `null` when unknown; US dollars are never assumed
- Generated narrative summary reads are selected only on an explicit ask for insights or narrative analysis
- Source Message IDs are opaque; hydrate only IDs returned by message rows or event `messageId`
- Exact live request shapes and examples live in the product docs below

## Behavior parity

Grounding, relative-time semantics, generated-summary selection, pagination completion, and failure qualification in `SKILL.md` are aligned with the behavior the Partners API/MCP server states in its `centaur://partners-api/filter-guide` resource, and with the behavior the Centaur chat assistant enforces on the same data questions. The server resource is the live source of truth; this skill restates those rules so they also apply to REST callers and to hosts that do not read MCP resources.

### Intentional differences

These are deliberate, not drift:

- **No injected time anchor.** The server resource and the chat assistant stamp the current server time into their instructions on every read or generation. A static skill document cannot, so it names the host's own current UTC clock as the anchor, with the `filter-guide` server timestamp as a fallback over MCP (it may be stale because hosts cache resources) and the current UTC time over REST.
- **No `"recent"` default.** The chat assistant resolves an unqualified `"recent"` to a rolling 7 days. That is a product-surface UX default; a skill running in an arbitrary host should not invent a window the user did not ask for, so `"recent"` falls under the ambiguous-phrase rule and prompts a clarifying question.
- **Quote-symbol presentation stops at the unit.** The chat assistant additionally formats USD-pegged quote symbols as dollars and others with a native currency symbol. That presentation layer is product-specific; the skill states only the underlying rule — price is in the row's `quoteSymbol`, never assume dollars, omit the unit when `quoteSymbol` is `null`.
- **Both surfaces, one set of rules.** The `filter-guide` resource is MCP-only. The skill covers MCP and REST, so it states the shared rules once and notes where the two surfaces differ (the time anchor, the auth header).
- **No documentation-search, page-context, or conversation-history guidance.** Those chat assistant rules depend on tools and UI state that no skill host is guaranteed to have, so the skill omits them rather than instructing against unavailable capabilities.
- **Model and conversation are host-owned.** An MCP host picks its own model and owns its own history; Centaur neither selects the model nor observes the conversation. The skill therefore forbids claiming model identity unless the host explicitly reports it, and forbids claiming that Centaur remembers earlier turns.

## Repository layout

- `SKILL.md`: installable skill entrypoint
- `references/auth.md`: auth model and decision tree
- `references/mcp.md`: MCP behavior and tool usage
- `references/rest.md`: REST behavior and query guidance
- `references/examples-curl.md`: curl examples only
- `references/client-setup.md`: Claude, ChatGPT, Cursor, and Codex setup
- `references/message-digests.md`: weighting, filtering, and structuring message digests

## Product docs

- Docs: [partners.centaur.io/docs](https://partners.centaur.io/docs)
- MCP endpoint: [partners.centaur.io/mcp](https://partners.centaur.io/mcp)
- OpenAPI: [partners.centaur.io/api/v1/openapi.json](https://partners.centaur.io/api/v1/openapi.json)
