# Client Setup

Use MCP first whenever possible.

Preferred MCP server URL:

```text
https://partners.centaur.io/mcp
```

Official Claude and ChatGPT installs use OAuth for active, email-verified signed-up users. For custom clients, register the plain URL and complete the Centaur browser sign-in flow when prompted.

## ChatGPT

Preferred:

```text
https://partners.centaur.io/mcp
```

Use the official ChatGPT App listing when available. For Developer Mode or custom setup, add the plain MCP URL and complete the Centaur browser sign-in flow.

## Claude Code

```bash
claude mcp add --transport http centaur https://partners.centaur.io/mcp
```

## Cursor

Add this to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "centaur": {
      "url": "https://partners.centaur.io/mcp"
    }
  }
}
```

## Codex

```bash
codex mcp add centaur --url https://partners.centaur.io/mcp
```

Optional persistent config:

```toml
[mcp_servers.centaur]
url = "https://partners.centaur.io/mcp"
```

For interactive Codex clients, prefer the plain MCP URL and let Codex dynamically register the OAuth client when prompted. Some OpenAI API-side MCP integrations may still require the application to supply an access token directly.

## If MCP is not configured

Export:

```bash
export CENTAUR_API_KEY='<api-key>'
```

Then use the REST examples in [examples-curl.md](examples-curl.md).

## One-time chat key

If the user does not want to set an environment variable yet, they may create a self-serve API key in Centaur and paste it directly into the current chat session for one-time REST use. Treat the pasted key as transient session input: keep it out of responses, files, config, and shell history.
