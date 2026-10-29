# Qualisend

Plugin that connects agents to [Qualisend](https://www.qualisend.com) through Qualisend's official
remote [Model Context Protocol](https://modelcontextprotocol.io/) server.

Verify email addresses before you send to them, clean lists, and check whether a domain is set up
to reach the inbox. One repo, packaged for Cursor, Grok Bot, Grok Build and Claude Code.

## Install

### Cursor

1. Open **Cursor Settings → Plugins**.
2. Search for **Qualisend**.
3. Click **Install**, then sign in to Qualisend when prompted.

Or run `/add-plugin qualisend` in chat.

### Grok Bot

In Grok Bot (0.49 or newer), connect **Qualisend** from the connector list, then sign in.

### Grok Build

Install **qualisend** from the xAI plugin marketplace.

### Claude Code

```
/plugin marketplace add qualisend/claude-plugins
/plugin install qualisend@qualisend
```

To load a clone of this repo for one session: `claude --plugin-dir .`

### Any other MCP client

```json
{
  "mcpServers": {
    "qualisend": {
      "type": "http",
      "url": "https://app.qualisend.com/api/mcp"
    }
  }
}
```

Per-client guides: https://www.qualisend.com/mcp

## Auth

OAuth 2.1 with PKCE and dynamic client registration. An unauthenticated tool call returns `401`
with `resource_metadata` pointing at `https://app.qualisend.com/.well-known/oauth-protected-resource`;
the client then opens a Qualisend sign-in and registers itself. There is no client ID, key or
secret to configure, and none is stored in this repo.

For headless use, send an API key instead: `Authorization: Bearer qs_live_...`, created in the
Qualisend app under **Verification API → Keys**. Keep it out of version control.

## Tools

| Tool | What it does | Plan |
| --- | --- | --- |
| `verify_email` | Verify one address | All plans, 1 credit |
| `verify_emails` | Verify up to 50 addresses, synchronously | All plans, 1 credit each |
| `verify_list` | Verify up to 1,000 addresses as a background job; returns a `job_id` | All plans, 1 credit each |
| `get_verification` | Result or progress for a `job_id` | All plans |
| `get_credit_balance` | Credits left in the workspace | All plans |
| `check_domain` | MX, SPF and DMARC setup of a sending domain | Pro / Enterprise |
| `generate_dns_record` | SPF, DMARC, BIMI or MTA-STS record | Pro / Enterprise |
| `analyze_email_header` | Route, per-hop delay and SPF/DKIM/DMARC results from raw headers | Pro / Enterprise |
| `check_disposable` | Throwaway, role or free-mail address | Pro / Enterprise |
| `check_typo` | Suggest a fix for a mistyped domain | Pro / Enterprise |
| `lookup_smtp_code` | Explain an SMTP reply code | Pro / Enterprise |
| `check_blacklist` | Check a domain or IPv4 address against major DNS blacklists | Blacklist Monitoring add-on |

The hosted server is the source of truth for tool names and schemas.

## Skills

| Skill | Use it when you're... |
| --- | --- |
| `verify-emails` | cleaning a list, CSV or database column before sending or importing |
| `email-deliverability` | setting up a sending domain, or asking why mail lands in spam |
| `add-email-validation` | adding address validation to a signup form, checkout or API |
| `diagnose-delivery` | holding a bounce message, an SMTP code or a raw header block |

## Credits and plans

One credit verifies one address; `unknown` verdicts are refunded. A free account starts with 100
credits, no card. More credits come as one-time packs or a monthly Pro plan; see
https://www.qualisend.com/pricing. The deliverability tools need Pro or Enterprise and use no
credits.

## Network and data

The plugin has no hooks, scripts or local server. Its only endpoint is
`https://app.qualisend.com/api/mcp` (plus the OAuth endpoints under `https://app.qualisend.com`).
Addresses, domains and headers you pass to a tool are sent there to be checked.

- Privacy: https://www.qualisend.com/privacy-policy
- Terms: https://www.qualisend.com/terms-of-service
- Security: [SECURITY.md](SECURITY.md)
- Support: hello@qualisend.com

## Docs

- MCP overview: https://www.qualisend.com/mcp
- Cursor guide: https://www.qualisend.com/mcp/cursor
- Grok guide: https://www.qualisend.com/mcp/grok
- REST API: https://www.qualisend.com/docs
- Server URL: https://app.qualisend.com/api/mcp

## License

MIT
