# Submission notes

Paste-ready values for each store. Publish this repo as `github.com/qualisend/agent-plugins`
(public, under the `qualisend` org) before submitting anything below.

## 1. Cursor Marketplace

Form: https://cursor.com/marketplace/publish. The same listing serves Grok Bot.

| Field | Value |
| --- | --- |
| Organization name | Qualisend |
| Organization handle | qualisend |
| Contact email | hello@qualisend.com |
| Logotype URL | https://raw.githubusercontent.com/qualisend/agent-plugins/main/assets/logo.png |
| Description | Verify email addresses and check deliverability from your agent. Check one address or a list of up to 1,000, see your credits, and on Pro review SPF and DMARC, generate DNS records, read headers and explain SMTP codes. Remote MCP with OAuth sign-in; 100 free credits, no card. |
| GitHub repository | https://github.com/qualisend/agent-plugins |
| Owner | Organization |
| Website URL | https://www.qualisend.com |

The description is 276 characters. The logo is a 512×512 PNG with an opaque brand-blue plate.

### Email to marketplace-publishing@cursor.com

> **Subject:** Qualisend plugin: please also enable for Grok Bot
>
> Hi Cursor team,
>
> We've submitted the Qualisend plugin (github.com/qualisend/agent-plugins) through the
> marketplace form. Could you also enable the listing for Grok Bot (the `grokbot` / `sand`
> clients)? The manifest already sets `minClientVersions` to cursor 3.13.0, grokbot 0.49.0 and
> sand 0.49.0, matching other remote OAuth plugins such as Questrade.
>
> It is OAuth-only remote MCP: one Streamable HTTP server at https://app.qualisend.com/api/mcp,
> OAuth 2.1 with PKCE and dynamic client registration, no API key, client ID or variables to
> configure. No hooks or scripts.
>
> Thanks,
> Qualisend — hello@qualisend.com

## 2. xAI Grok Build marketplace

PR to https://github.com/xai-org/plugin-marketplace. Append this entry to the `plugins` array
in `.grok-plugin/marketplace.json`, after replacing the `sha` with the commit to ship:

```bash
git ls-remote https://github.com/qualisend/agent-plugins.git HEAD
```

```json
{
  "name": "qualisend",
  "description": "Qualisend email verification and deliverability (hosted MCP server + skills). Verify one address or a list of up to 1,000, check credits, and on Pro review a domain's SPF and DMARC, generate DNS records, read message headers and explain SMTP codes. Sign in with your Qualisend account; no API key needed.",
  "category": "development",
  "source": {
    "source": "url",
    "url": "https://github.com/qualisend/agent-plugins.git",
    "sha": "0000000000000000000000000000000000000000"
  },
  "homepage": "https://www.qualisend.com/mcp",
  "keywords": [
    "qualisend",
    "qualisend email verification",
    "qualisend email verifier",
    "qualisend mcp",
    "qualisend deliverability"
  ],
  "domains": [
    "qualisend.com",
    "www.qualisend.com",
    "app.qualisend.com"
  ]
}
```

Then, in the fork:

```bash
python3 scripts/generate-plugin-index.py
python3 scripts/validate-catalog.py
python3 scripts/generate-plugin-index.py --check
```

Commit `.grok-plugin/marketplace.json` and the regenerated `.grok-plugin/plugin-index.json`.

### PR title

`Add qualisend plugin`

### PR body

```markdown
## What this PR does

Adds the Qualisend plugin: email verification and deliverability through Qualisend's hosted MCP server, plus four skills.

- Plugin name: `qualisend`
- Type: remote source
- Source URL + pinned SHA (remote), or vendored path (local): https://github.com/qualisend/agent-plugins.git @ `<sha>`
- Homepage: https://www.qualisend.com/mcp

## Ownership

- [x] I own this plugin or have the right to distribute it.
- [x] The `source` repo is published under our official org (or I've explained why not below).

## Checklist

- [x] Added/updated exactly one entry in `.grok-plugin/marketplace.json` (valid JSON, kebab-case `name`).
- [x] Remote source pins a full 40-char lowercase commit `sha`, and that commit is public + reachable.
- [x] Regenerated `.grok-plugin/plugin-index.json` (`python3 scripts/generate-plugin-index.py`).
- [x] `python3 scripts/validate-catalog.py` passes locally.
- [x] `python3 scripts/generate-plugin-index.py --check` passes locally.
- [x] `homepage` + clear `description` set; local plugins include `README.md` + `.grok-plugin/plugin.json`.
- [x] License is stated.

## Security

- [x] No `curl | bash`, remote-code download/exec, or `postinstall` RCE.
- [x] No reading/exfiltration of secrets, tokens, `.env`, or env vars.
- [x] Hooks and MCP scope are least-privilege.
- Network endpoints this plugin calls (and why): `https://app.qualisend.com/api/mcp` (Streamable HTTP MCP, the tool calls) and the OAuth endpoints under `https://app.qualisend.com` (sign-in, dynamic client registration, token refresh).
- Credentials/permissions it requires (and why): a Qualisend account, via OAuth 2.1 (PKCE, scopes `verify` and `read`). No API key, env var or local file is read. An API key is optional for headless use and never stored in the repo.

## Notes for reviewers

- Components: one remote MCP server (`.mcp.json`) and four Markdown skills. No hooks, commands, agents, scripts or LSP servers.
- The repo is the official `qualisend` GitHub org; app.qualisend.com and www.qualisend.com are our domains.
- The same repo carries `.cursor-plugin/`, `.claude-plugin/` and Agent Plugins manifests for other clients; `.grok-plugin/plugin.json` is read first.
- Keywords and domains are brand-scoped only.
```

## 3. Claude Code

The live Claude Code marketplace is `qualisend/claude-plugins` (`/plugin install qualisend@qualisend`).
This repo carries the same plugin with `.claude-plugin/plugin.json` at the root, so the marketplace
can later point at it instead of a vendored copy:

```json
{
  "name": "qualisend",
  "source": { "source": "github", "repo": "qualisend/agent-plugins" }
}
```

Check with `claude plugin validate .` before switching.
