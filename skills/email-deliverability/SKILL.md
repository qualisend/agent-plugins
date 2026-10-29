---
name: email-deliverability
description: Check and fix whether a domain's email reaches the inbox - SPF, DKIM, DMARC, MTA-STS, MX and blacklist status. Use when the user asks why their email lands in spam, is setting up a sending domain for transactional or marketing mail, is configuring SendGrid/Postmark/SES/Resend/Mailgun, or wants SPF/DMARC/BIMI records generated or reviewed.
---

# Domain email deliverability

Use the `qualisend` MCP tools to inspect a sending domain and produce the exact
DNS records it needs.

| Question | Tool |
|---|---|
| Is this domain set up correctly to send mail? | `check_domain` |
| What record do I need for SPF / DMARC / BIMI / MTA-STS? | `generate_dns_record` |
| Is my domain or sending IP on a blacklist? | `check_blacklist` |

Run `check_domain` **first**, always. It reports the MX, SPF and DMARC records
actually published, and almost every real problem is visible there. Generating
records before reading the current state produces advice that conflicts with what
the domain already has - which is how domains end up with two SPF records.

## The failure modes that actually cause spam folder placement

Work through these in order; the first two account for most cases.

**More than one SPF record is a hard failure, not a warning.** RFC 7208 §4.5
makes two `v=spf1` TXT records a `permerror`, and receivers treat a permerror as
no SPF at all - so adding a second record to "also allow" a new sender silently
breaks the sender that was already working. A domain gets exactly one SPF record;
new senders are added as extra mechanisms **inside** it. When the user adds a
provider, always edit the existing record rather than publishing another.
`check_domain` shows one SPF record, so confirm there is no second one with
`dig TXT <domain>`.

**SPF has a 10-lookup limit.** Every `include:`, `a`, `mx`, `ptr`, `exists:` and
`redirect=` counts, and nested includes count too. Exceed it and the result is
again `permerror`. A domain with four ESPs chained through `include:` is usually
already over. `check_domain` shows the record but not the count, so count the
lookups yourself, following each `include:`.

**SPF alignment is not SPF passing.** DMARC requires the domain in the `From:`
header to align with the domain SPF authenticated, which is the envelope sender.
An ESP that sends with its own bounce domain passes SPF and still fails DMARC.
This is the single most common "but my SPF is fine" case. DKIM alignment usually
fixes it, because DKIM survives forwarding and SPF does not.

**`p=none` is monitoring, not enforcement.** A DMARC record at `p=none` protects
nobody from spoofing. It is the right place to start - collect `rua` reports for
a few weeks, confirm legitimate mail passes - but the user should know it is a
staging state, not the finished job. Move to `p=quarantine`, then `p=reject`.

**Blacklists.** Only relevant if mail is being rejected or bulk-foldered at
specific providers. `check_blacklist` covers the major zones. A domain-level
listing (such as Spamhaus DBL) is inherited by subdomains, so a listing on the
apex affects `mail.`, `app.` and everything else.

## Producing the fix

After `check_domain`, use `generate_dns_record` for each record that is missing or
wrong, then give the user the literal DNS rows to paste - host, type, value, TTL.
DKIM is the exception: the key comes from the sending provider, not a generator.
Do not paraphrase a record; a DNS value that is retyped is a DNS value that is
wrong.

Two things to say explicitly whenever you hand over records:

- **DNS changes take time to propagate.** Re-run `check_domain` after the TTL,
  not immediately, or you will read the old value and think it failed.
- **Never publish a second SPF record.** If the domain already has one, show the
  merged single record, not an additional one.

## Scope

`check_domain`, `generate_dns_record`, `analyze_email_header`, `check_disposable`,
`check_typo` and `lookup_smtp_code` need a Qualisend Pro plan. `check_blacklist`
needs the Blacklist Monitoring add-on. If a call comes back saying the plan does
not cover it, tell the user which plan it needs and carry on with what you can
determine from plain DNS lookups - `dig TXT`, `dig MX` and a DMARC lookup at
`_dmarc.<domain>` answer a good deal of this without any account at all.
