# Security

## What this plugin contains

- `mcp.json` / `.mcp.json`: one remote MCP server, `https://app.qualisend.com/api/mcp`
  (Streamable HTTP).
- `skills/`: four Markdown skills. Instructions only; no code runs from them.
- Manifests and a logo.

There are no hooks, no install or postinstall scripts, no local binaries and no bundled
dependencies.

## Network endpoints

| Endpoint | Why |
| --- | --- |
| `https://app.qualisend.com/api/mcp` | Tool calls |
| `https://app.qualisend.com/.well-known/oauth-*`, `/oauth/authorize`, `/api/oauth/*` | OAuth 2.1 sign-in, client registration and token refresh |

## Credentials

None are stored in this repo. The client signs in over OAuth 2.1 (PKCE S256, dynamic client
registration); access tokens last 1 hour and refresh tokens rotate. Scopes are `verify` (spends
credits) and `read`. You can revoke a connection in the Qualisend app under
**Settings → Account → Connected apps**.

An optional API key (`qs_live_...`) works for headless use. Treat it as a secret.

## Data

Addresses, domains and headers passed to a tool are sent to Qualisend to be checked, under the
[privacy policy](https://www.qualisend.com/privacy-policy).

## Reporting a vulnerability

Email hello@qualisend.com with "Security" in the subject. Please do not open a public issue for a
vulnerability. More at https://www.qualisend.com/security.
