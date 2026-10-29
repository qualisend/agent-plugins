---
name: diagnose-delivery
description: Work out why a specific email bounced, was delayed, or landed in spam. Use when the user has a bounce message, an SMTP error code like 550 or 4.7.1, a raw email header block, or asks why one particular message failed or took a long time to arrive.
---

# Diagnose a single message

This is for one message that misbehaved. If the question is about a domain's
setup in general, use the `email-deliverability` skill instead.

| What the user has | Tool |
|---|---|
| An error code or bounce string (`550 5.1.1`, `4.7.1`, `421`) | `lookup_smtp_code` |
| A raw header block (`Received:`, `Authentication-Results:`, ...) | `analyze_email_header` |

## Read the first digit before anything else

It determines whether there is a problem to fix at all:

- **`4xx` is temporary.** The sending server will retry, typically for a day or
  more. A `4xx` is usually greylisting, rate limiting, or a busy receiver, and
  most resolve without intervention. Do not "fix" a `4xx` by removing the
  address - it is not a bounce.
- **`5xx` is permanent.** The message will not be delivered on retry.
  - `550 5.1.1` - mailbox does not exist. Remove the address.
  - `550 5.7.x` - **policy**, not a missing mailbox. Authentication failed,
    reputation, or a blocklist. The address is fine; the *sender* is the problem.
    This is the important distinction, because deleting the recipient does
    nothing and the next send fails the same way.
  - `552` / `5.2.2` - mailbox full. Temporary in practice despite the 5.

A `5.7.x` against many recipients at one provider means a sender-side problem.
Follow it with `check_domain` and `check_blacklist` on the sending domain and IP.

## Headers

`analyze_email_header` parses a raw header block and reports the hop-by-hop route
with per-hop delay, plus the SPF, DKIM and DMARC results. Ask the user for the
**full unmodified header block** - the "show original" or "view source" output
from their mail client. A truncated paste loses exactly the hops that explain the
delay.

Two things headers answer that nothing else does:

- **Where the time went.** `Received:` hops are listed newest-first, so read from
  the bottom up. One hop with a large gap is the culprit - usually greylisting or
  a queue at the receiver, not the sender.
- **Whether authentication actually passed for this message.**
  `Authentication-Results:` gives the receiver's own verdict. This beats any
  amount of reasoning about the DNS, because it is what the receiving server
  concluded at delivery time.

Watch for **SPF passing while DMARC fails**. That is alignment: SPF authenticated
the envelope sender, and DMARC requires the `From:` header domain to match. An
ESP sending with its own bounce domain produces exactly this, and it is the most
commonly misread line in a header block.

## Headers contain personal data

A raw header block carries real addresses, subject lines, internal hostnames and
IPs. Do not paste header content into a commit, an issue, or any file in the
user's repository while diagnosing. Keep it in the conversation.

## Scope

`lookup_smtp_code` and `analyze_email_header` need a Qualisend Pro plan. If the
plan does not cover it, you can still get a long way unaided: the RFC 3463
enhanced status code in the bounce (`5.1.1`, `4.7.1`) is self-describing once you
know the first digit rule above, and `Authentication-Results:` can be read
directly.
