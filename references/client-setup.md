# Client Setup

Use MCP first whenever possible.

## Claude Code

```bash
claude mcp add --transport http centaur-partners https://partners.centaur.io/mcp \
  --header "Authorization: Bearer <partner-api-key>"
```

## Cursor

Add Centaur to `~/.cursor/mcp.json`:

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

## If MCP is not configured

Export:

```bash
export CENTAUR_PARTNER_API_KEY='<partner-api-key>'
```

Then use the REST examples in [examples-curl.md](examples-curl.md).

## One-time chat fallback

If the user does not want to set an environment variable yet, they may paste a partner API key directly into the current chat session for one-time REST use.

- Treat the pasted key as transient session input only.
- Do not echo it back in responses.
- Do not store it in files, config, or shell history.
