# Client Setup

Use MCP first whenever possible.

Preferred MCP server URL:

```text
https://partners.centaur.io/mcp
```

If the client supports MCP OAuth, register the plain URL, let the client dynamically register itself if needed, and complete the Centaur sign-in and consent flow in the browser. Public Dynamic Client Registration is only available for PKCE public clients. Redirect URIs must use `https`, a native app/private-use scheme, or loopback `http` during local development. OAuth bearer tokens come from the OAuth flow itself rather than a separate generic JWT helper endpoint.

If the client cannot complete OAuth yet, use the compatibility instructions below.

## Claude Code

Preferred:

```bash
claude mcp add --transport http centaur-partners https://partners.centaur.io/mcp
```

Compatibility fallback:

```bash
claude mcp add --transport http centaur-partners https://partners.centaur.io/mcp \
  --header "Authorization: Bearer <partner-api-key>"
```

## Cursor

Preferred `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "centaur-partners": {
      "url": "https://partners.centaur.io/mcp"
    }
  }
}
```

Compatibility fallback:

```json
{
  "mcpServers": {
    "centaur-partners": {
      "url": "https://partners.centaur.io/mcp",
      "headers": {
        "Authorization": "Bearer ${env:CENTAUR_PARTNER_API_KEY}"
      }
    }
  }
}
```

Export the same key for direct REST fallback:

```bash
export CENTAUR_PARTNER_API_KEY='<partner-api-key>'
```

## Codex

Preferred:

```bash
codex mcp add centaur-partners --url https://partners.centaur.io/mcp
```

Compatibility fallback:

```bash
codex mcp add centaur-partners --url https://partners.centaur.io/mcp \
  --header "Authorization: Bearer <partner-api-key>"
```

Optional persistent config:

```toml
[mcp_servers.centaur-partners]
url = "https://partners.centaur.io/mcp"

[mcp_servers.centaur-partners.headers]
Authorization = "Bearer <partner-api-key>"
```

For interactive Codex clients, prefer the plain MCP URL and let Codex dynamically register the OAuth client when prompted. Some OpenAI API-side MCP integrations may still require the application to supply an access token directly.

## If MCP is not configured

Export:

```bash
export CENTAUR_PARTNER_API_KEY='<partner-api-key>'
```

Then use the REST examples in [examples-curl.md](examples-curl.md).

Once connected, the client can use Centaur across events, messages, positions, discovery, and stats.

## One-time chat fallback

If the user does not want to set an environment variable yet, they may paste a partner API key directly into the current chat session for one-time REST use.

- Treat the pasted key as transient session input only.
- Do not echo it back in responses.
- Do not store it in files, config, or shell history.
