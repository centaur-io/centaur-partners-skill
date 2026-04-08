# Auth

Centaur uses the same partner API key across MCP and REST.

## Decision order

1. Prefer MCP if the current client already has a Centaur MCP server configured.
2. Otherwise check for `CENTAUR_PARTNER_API_KEY`.
3. If the env var exists, use REST.
4. Otherwise the user may paste a partner API key directly into the current chat for one-time use.
5. If a pasted key is provided, use REST for the current session only and do not persist the key.
6. If neither is available, stop and ask the user to configure Centaur rather than inventing credentials.

## Headers

- MCP: `Authorization: Bearer <partner-api-key>`
- REST: `x-api-key: <partner-api-key>`

## Access shape

- The same key format works across events, messages, positions, and stats.
- Individual reads can still be scope-gated by the server.

## Notes

- Do not assume OAuth, session auth, or SDK-specific auth flows exist.
- Do not log, echo, or persist a real API key in output unless the user explicitly asks for that behavior.
- If the user pastes a key in chat, treat it as transient session input only.
