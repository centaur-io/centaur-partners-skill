# Client Setup

Use MCP first whenever possible.

Preferred MCP server URL:

```text
https://partners.centaur.io/mcp
```

Official Claude and ChatGPT installs use OAuth for active, email-verified signed-up users. For custom clients, register the plain URL and complete the Centaur browser sign-in flow when prompted. If the client cannot complete OAuth yet, use the compatibility instructions below.

## ChatGPT

Preferred:

```text
https://partners.centaur.io/mcp
```

Use the official ChatGPT App listing when available. For Developer Mode or custom setup, add the plain MCP URL and complete the Centaur browser sign-in flow.

## Claude Code

Preferred:

```bash
claude mcp add --transport http centaur https://partners.centaur.io/mcp
```

Compatibility fallback:

```bash
claude mcp add --transport http centaur https://partners.centaur.io/mcp \
  --header "Authorization: Bearer <api-key>"
```

## Cursor

Preferred `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "centaur": {
      "url": "https://partners.centaur.io/mcp"
    }
  }
}
```

Compatibility fallback:

```json
{
  "mcpServers": {
    "centaur": {
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
export CENTAUR_PARTNER_API_KEY='<api-key>'
```

## Codex

Preferred:

```bash
codex mcp add centaur --url https://partners.centaur.io/mcp
```

Compatibility fallback:

```bash
codex mcp add centaur --url https://partners.centaur.io/mcp \
  --header "Authorization: Bearer <api-key>"
```

Optional persistent config:

```toml
[mcp_servers.centaur]
url = "https://partners.centaur.io/mcp"

[mcp_servers.centaur.headers]
Authorization = "Bearer <api-key>"
```

For interactive Codex clients, prefer the plain MCP URL and let Codex dynamically register the OAuth client when prompted. Some OpenAI API-side MCP integrations may still require the application to supply an access token directly.

## If MCP is not configured

Export:

```bash
export CENTAUR_PARTNER_API_KEY='<api-key>'
```

Then use the REST examples in [examples-curl.md](examples-curl.md).

Once connected, the client can use Centaur across events, messages, generated aggregate summaries, generated channel summaries, positions, discovery, and stats.

## One-time chat fallback

If the user does not want to set an environment variable yet, they may create a self-serve API key in Centaur and paste it directly into the current chat session for one-time REST use.

- Treat the pasted key as transient session input only.
- Do not echo it back in responses.
- Do not store it in files, config, or shell history.
