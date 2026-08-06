# Auth

Centaur uses different auth patterns by surface.

## Headers

- Official Claude and ChatGPT installs: OAuth, available to active, email-verified signed-up users
- REST: `x-api-key: <api-key>`

## Access shape

- API keys are self-serve REST credentials and do not require manual approval.
- One key format works across the whole read surface — the feed, events, messages, Generated Aggregate Narrative Summaries, Generated Channel Narrative Summaries, positions, discovery, stats, trader rankings, and activity summaries — though individual reads can still be scope-gated by the server.
