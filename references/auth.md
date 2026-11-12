# Auth

Centaur uses different auth patterns by surface.

## Headers

- Official Claude and ChatGPT installs: OAuth, available to active, email-verified signed-up users
- REST: `x-api-key: <api-key>`

## Access shape

- API keys are self-serve REST credentials and do not require manual approval.
- One key format works across the whole read surface — the feed, events, trade views (beta), messages, Generated Aggregate Narrative Summaries, Generated Channel Narrative Summaries, positions, discovery, stats, trader rankings, and activity summaries — though individual reads can still be scope-gated by the server.
- Trade views (beta) need `views.read` on `GET /v2/views` and `list_views`. The feed carries trade views under `feed.read` without `views.read`. An older key or AI client that gets `403` on the views read lacks the scope: create a new key, or reconnect the client and approve the new scope.
