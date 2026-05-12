# Auth

Centaur uses different auth patterns by surface.

## Decision order

1. Prefer MCP if the current client already has a Centaur MCP server configured.
2. Otherwise check for `CENTAUR_PARTNER_API_KEY`.
3. If the env var exists, use REST.
4. Otherwise the user may paste a Centaur API key directly into the current chat for one-time use.
5. If a pasted key is provided, use REST for the current session only and do not persist the key.
6. If neither is available, stop and ask the user to configure Centaur rather than inventing credentials.

## Headers

- Official Claude and ChatGPT installs: OAuth
- Custom/manual MCP fallback: `Authorization: Bearer <api-key>`
- REST: `x-api-key: <api-key>`

## Access shape

- The same key format works across events, messages, aggregate summaries, channel summaries, positions, discovery, and stats.
- MCP OAuth is available to active, email-verified signed-up users.
- API keys are self-serve REST/custom-client credentials and do not require manual approval.
- Summary reads require normal summary read authorization and may return empty pages when no generated summaries match the requested window.
- Individual reads can still be scope-gated by the server.

## Notes

- Do not try to drive an OAuth flow or Dynamic Client Registration from the skill itself. Either the client is already connected to the Centaur MCP server or the skill should fall back to REST.
- Do not log, echo, or persist a real API key in output unless the user explicitly asks for that behavior.
- If the user pastes a key in chat, treat it as transient session input only.
