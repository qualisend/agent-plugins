# Changelog

All notable changes to this plugin will be documented here.

## 1.0.0 — initial release

- Added the `qualisend` MCP server pointing at Qualisend's hosted Streamable HTTP endpoint
  (`https://app.qualisend.com/api/mcp`).
- Auth uses OAuth 2.1 with PKCE and dynamic client registration — no API key or client ID to
  configure. A `qs_live_` API key also works for headless use.
- Four skills: `verify-emails`, `email-deliverability`, `add-email-validation`,
  `diagnose-delivery`.
- Manifests for Cursor (`3.13.0`), Grok Bot (`0.49.0`), Grok Build, Claude Code and the Agent
  Plugins standard.
- Logo: the Qualisend mark on a 512×512 brand-blue tile.
