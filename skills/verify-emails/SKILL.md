---
name: verify-emails
description: Check whether email addresses are real and safe to send to. Use when the user has a list, CSV, database column, array, or single address of emails and wants them validated, cleaned, deduplicated, or checked for bounces, typos, disposable domains, or role accounts before sending or importing.
---

# Verify email addresses

Use the `qualisend` MCP tools to find out which addresses are real before the user
sends to them. A list that has never been cleaned typically carries 10-25% dead
addresses, and mailbox providers read bounces as a spam signal against the sender,
so the cost of sending to a bad list is not the wasted send - it is the reputation
damage that lowers inbox placement for the good addresses too.

## Which tool

| Situation | Tool |
|---|---|
| One address | `verify_email` |
| 2-50 addresses | `verify_emails` (one call, synchronous) |
| 51-1000 addresses | `verify_list` (async, returns `job_id`) |
| Checking on an async job | `get_verification` with the `job_id` |

Never loop `verify_email` over a list - batch it. Each address costs one credit
either way, but a loop is far slower and burns the per-call rate limit.

## Reading the verdict

Every address comes back with one of four statuses. They are not a quality ranking
and should not be collapsed into "good/bad":

- **`deliverable`** - the mailbox exists and accepted the probe. Safe to send.
- **`undeliverable`** - the mailbox does not exist. **Remove it.** This is the
  category that causes hard bounces.
- **`risky`** - the server answered, but not in a way that confirms the mailbox.
  Most often a catch-all domain (accepts everything, so nothing can be proven),
  a full mailbox, or a role address. Sending is a judgement call - see below.
- **`unknown`** - no verdict could be reached, usually because the receiving
  server refused the probe or timed out. **`unknown` is not a bounce prediction.**
  Do not remove these; re-check later or treat them as unverified.

`score` (0-100) is a confidence figure, and `sub_flags` carries the specific
findings: `role`, `free_mail`, `disposable`, `catch_all`, `full_mailbox`,
`disabled`.

## What to do with the results

Report the counts first, then act. When cleaning a list:

1. **Drop `undeliverable`.** No ambiguity here.
2. **Keep `deliverable`.**
3. **Ask about `risky`** rather than deciding silently. A catch-all domain is
   common at companies and often perfectly valid; a `disposable` address rarely
   is. The right call depends on whether the user is sending transactional mail
   (keep more) or cold outreach (drop more).
4. **Leave `unknown` in place** and say so. Deleting an address we could not
   check is destroying data on no evidence.

If `did_you_mean` is set, the address has a likely typo (`gmial.com` →
`gmail.com`). Surface the suggestion; do not silently rewrite a user's data.

## Writing results back

When the addresses came from a file the user owns, write the cleaned output to a
**new** file and leave the original untouched, unless they asked otherwise. Keep
the verdict columns - a list that has been cleaned once is worth re-checking in a
few months, and the previous verdict is the baseline.

## Account and cost

The tools need a Qualisend account; the first tool call opens a browser to sign
in. A new account starts with 100 free credits, no card. Verification costs one
credit per address. If the user has no account, say that plainly before running
anything on a large list - do not start a 1000-address job and surprise them with
a sign-up wall. `get_credit_balance` shows what is left.
