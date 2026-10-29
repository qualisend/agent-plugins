---
name: add-email-validation
description: Add email address validation to an application - signup forms, waitlists, checkout, CRM imports, or an API endpoint. Use when the user wants to stop fake, mistyped, disposable, or undeliverable addresses entering their database, or asks how to validate emails server-side beyond a regex.
---

# Add email validation to an application

## First decide which layer the user actually needs

These are three different jobs, and reaching for the paid one when the free one
would do is bad engineering:

**1. Syntax.** Is it shaped like an address? Free, instant, local, no network.
Use the platform's own validator (`zod`'s `z.email()`, `validator.isEmail`,
Python's `email-validator`). **Never hand-write an email regex** - the grammar in
RFC 5322 is not regular, and every hand-rolled pattern rejects valid addresses
like `user+tag@` or `firstname.o'brien@`. If the user only asked to catch
`notanemail`, stop here.

**2. Domain.** Does the domain exist and accept mail? One MX lookup, free, ~30ms.
Catches `@gmial.com` and dead company domains. Worth doing inline on any signup
form. `check_typo` also suggests the correction for a near-miss domain.

**3. Mailbox.** Does this specific mailbox exist? Requires an SMTP conversation
with the receiving server, takes a second or two, and costs a credit. This is the
only layer that catches `definitely.not.real@gmail.com`. Use it where a wrong
address has a real cost: paid signups, password reset flows, transactional
receipts, list imports.

A good default for a signup form is **1 + 2 inline, 3 asynchronously** after
submit - so the user is never left waiting on a network call mid-form.

## Integrating the API

Qualisend's REST API is at `https://app.qualisend.com`. Auth is a bearer API key
(`qs_live_...`) the user creates in their dashboard under Verification API → Keys.
**The key is a server-side secret** - it must never reach the browser bundle, a
mobile app, or a public repo. Read it from the environment.

```
POST /api/v1/verify
Authorization: Bearer qs_live_...
Content-Type: application/json
Idempotency-Key: <uuid>        # optional; makes retries safe, no double charge

{ "email": "someone@example.com" }
```

### The part that is easy to get wrong

`POST /api/v1/verify` returns **immediately with the local verdict** - syntax,
domain, disposable, role, typo - while the mailbox probe is still running. The
response carries `smtp: "pending"` and a `job_id`.

If you treat that first response as final, a perfectly deliverable address reads
as `unknown`, because the mailbox check had not finished yet. Poll
`GET /api/v1/jobs/{job_id}` until the probe settles - usually a few seconds. For
lists, use `POST /api/v1/verify/bulk` with a `callback_url` and let it call you back.

```js
const res = await fetch("https://app.qualisend.com/api/v1/verify", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.QUALISEND_API_KEY}`,
    "Content-Type": "application/json",
    "Idempotency-Key": crypto.randomUUID(),
  },
  body: JSON.stringify({ email }),
});
let result = await res.json();

// The mailbox probe is asynchronous. Without this, a deliverable address
// comes back "unknown" simply because the probe had not landed yet.
if (result.smtp === "pending") {
  const deadline = Date.now() + 8000;
  while (Date.now() < deadline) {
    await new Promise((r) => setTimeout(r, 300));
    // `include=results` is required - without it the response carries only
    // status and progress counters, and `results` is simply absent.
    const poll = await fetch(
      `https://app.qualisend.com/api/v1/jobs/${result.job_id}?include=results`,
      { headers: { Authorization: `Bearer ${process.env.QUALISEND_API_KEY}` } },
    );
    const job = await poll.json();
    if (job.status === "completed") { result = job.results[0]; break; }
  }
}
```

Other endpoints: `POST /api/v1/verify/bulk` for lists, `GET /api/v1/jobs/{id}` for
status and progress (add `?include=results` for the per-address rows),
`POST /api/v1/jobs/{id}/cancel` to stop a running bulk job, and `GET /api/v1/credits`
for the balance.

## How the result should affect the product

Map the four statuses to behaviour deliberately, and **do not block on
`unknown`**:

| Status | Suggested handling |
|---|---|
| `deliverable` | Accept. |
| `undeliverable` | Reject with a specific message: "We couldn't find that mailbox - check the spelling." |
| `risky` | Accept, but flag it. Blocking catch-all domains locks out large numbers of legitimate corporate users. |
| `unknown` | **Accept.** No verdict was reached. Rejecting here turns an outage at the receiving server, or a timeout, into a lost signup. |

If `did_you_mean` is set, show it as a confirmation prompt - "Did you mean
`user@gmail.com`?" - rather than silently correcting what the user typed.

## Build it to survive the API being down

Verification is an enhancement to a signup flow, not a dependency of it. Set a
short timeout, and on timeout or error **let the signup through** and record it
for later checking. A verification provider outage must never become an outage of
the user's own product. Say this to the user explicitly if they are putting the
call in a blocking path.
