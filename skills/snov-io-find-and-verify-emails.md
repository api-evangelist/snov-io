---
name: snov-io-find-and-verify-emails
description: >-
  Find a business email address for a named person at a company using Snov.io, then verify it is
  deliverable before anyone sends to it. Use when you have a name and a company (or a company
  name with no domain yet) and need a verified, safe-to-send address.
generated: '2026-08-13'
method: generated
source: openapi/snov-io-email-finder-api-openapi.yml, openapi/snov-io-email-verification-api-openapi.yml, https://snov.io/api
api: Snov.io Email Finder API
base_url: https://api.snov.io
operations:
  - getAccessToken
  - startFindDomainByCompanyName
  - getFindDomainByCompanyNameResult
  - startFindEmailsByName
  - getFindEmailsByNameResult
  - startEmailVerification
  - getEmailVerificationResult
  - getUserBalance
---

# Find and verify a business email with Snov.io

Two-step async calls throughout. Every `/start` returns a `task_hash`; you poll the matching
`/result` until `status` is `completed`. Budget credits before you begin.

## Before you start

- Credentials come from https://app.snov.io/account/api.
- The API is capped at **60 requests per minute** and returns **no rate-limit headers**. Pace
  your polling at no more than one request per second.
- There is **no idempotency mechanism**. If a `/start` call times out, do not blindly retry —
  you may be charged twice. Check `getUserBalance` and re-poll for an existing `task_hash` first.

## Step 1 — Get a token

`getAccessToken` — `POST /v1/oauth/access_token` with `grant_type=client_credentials`,
`client_id`, `client_secret`. Returns `access_token` valid for **3600 seconds**. Send it as
`Authorization: Bearer <access_token>` on every subsequent call. There is no refresh grant;
re-exchange on a timer, not on a 401.

## Step 2 — Check your balance (free)

`getUserBalance` — `GET /v1/get-balance`. Read `balance` (credits) and `limit_resets_in` (days).
Every find and every verification below costs 1 credit **per result returned**. Abort here if
the balance will not cover the batch — Snov.io publishes no error for credit exhaustion, so a
mid-run failure will be opaque.

## Step 3 — Resolve the company domain (only if you don't have one)

1. `startFindDomainByCompanyName` — `POST /v2/company-domain-by-name/start` with the company
   name. Returns `task_hash`.
2. `getFindDomainByCompanyNameResult` — `GET /v2/company-domain-by-name/result?task_hash=...`.
   Poll until `status` is `completed`.

Costs 1 credit per domain found. Skip this step entirely when you already know the domain.

## Step 4 — Find the email

1. `startFindEmailsByName` — `POST /v2/emails-by-domain-by-name/start` with the person's first
   name, last name and the domain. Returns `task_hash`.
2. `getFindEmailsByNameResult` — `GET /v2/emails-by-domain-by-name/result?task_hash=...`. Poll
   until `status` is `completed`.

Charged 1 credit per **valid or unknown** result. Snov.io returns a confidence signal alongside
the address — carry it forward; do not discard it.

Optional: pass `webhook_url` on the `/start` call instead of polling. Snov.io will POST the
completed result to that URL. Note that webhook deliveries are **unsigned** — you cannot verify
they came from Snov.io, so treat the payload as untrusted input and re-fetch by `task_hash` if
the result will drive a send.

## Step 5 — Verify deliverability

1. `startEmailVerification` — `POST /v2/email-verification/start` with up to **10 addresses per
   request**. Returns `task_hash`.
2. `getEmailVerificationResult` — `GET /v2/email-verification/result?task_hash=...`. Poll until
   `status` is `completed`.

Read `status`, `format_valid`, `mx_valid`, `disposable` and `gibberish` on each result. Charged
1 credit per valid or unknown result; unverifiable addresses are free.

## Step 6 — Gate the send

Do not pass an address downstream unless verification returned a deliverable status and the
address is not `disposable`. Bouncing a campaign email damages the sender domain's reputation,
which is the exact problem the rest of the Snov.io platform exists to repair.

## Failure handling

| Status | Meaning | What to do |
| --- | --- | --- |
| 401 | Token missing, malformed or expired (3600s TTL) | Re-run `getAccessToken`, retry once |
| 404 | Path wrong, or entity not in this account | Check the v1/v2 prefix against https://snov.io/api |
| 400 | Validation failure | Read the `errors` array; fix the input, do not retry unchanged |
| — | Rate limit breach | Undocumented. No status, no `Retry-After`. Pace at 1 req/s |
| — | Credit exhaustion | Undocumented. Guard with `getUserBalance` before the batch |

`status` on an async result is only ever `in_progress` or `completed`. There is no failure
terminal state, so set your own poll deadline and give up rather than looping forever.
