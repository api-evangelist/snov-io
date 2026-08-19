---
name: snov-io-subscribe-to-webhooks
description: >-
  Subscribe to Snov.io events so outreach results arrive as pushes instead of polls — sends,
  opens, clicks, bounces, email and LinkedIn replies, unsubscribes, and completed async search
  tasks. Use whenever an integration would otherwise poll Snov.io on a timer.
generated: '2026-08-13'
method: generated
source: openapi/snov-io-webhooks-api-openapi.yml, asyncapi/snov-io-webhooks.yml, https://snov.io/api
api: Snov.io Webhooks API
base_url: https://api.snov.io
operations:
  - getAccessToken
  - listWebhooks
  - addWebhook
  - updateWebhook
  - deleteWebhook
---

# Subscribe to Snov.io webhooks

The Snov.io REST API allows 60 requests per minute across the entire account and returns no
rate-limit headers. Webhooks are not a nicety here — they are how you avoid spending your whole
budget on polling.

## Step 1 — Token

`getAccessToken` — `POST /v1/oauth/access_token`, `grant_type=client_credentials`.

## Step 2 — Know the delivery contract before you build the receiver

- Snov.io POSTs a JSON body to your endpoint.
- A delivery **succeeds** only on an HTTP status in the **200–299** range returned **within 3
  seconds**. Acknowledge first, process asynchronously — a slow handler is a failed delivery.
- On failure Snov.io retries **7 times over 38 hours**: immediately, then +20m, +40m, +60m, +4h,
  +8h, +24h. After the seventh failure the subscription is **automatically deactivated** and
  stops delivering silently.
- **Payloads are not signed.** There is no HMAC, no shared secret, no timestamp header and no
  published source IP range. Treat every payload as untrusted: use an unguessable endpoint path,
  and re-fetch anything consequential from the API by id or `task_hash` before acting on it.
- Premium plans allow up to **50 subscriptions** per account.

## Step 3 — Create subscriptions

`addWebhook` — `POST /v2/webhooks` with `Content-Type: application/json` and a body of
`event_object`, `event_action`, `endpoint_url`. One subscription per object/action pair —
there is no wildcard and no per-campaign scoping.

The 8 objects and 20 actions:

| `event_object` | `event_action` |
| --- | --- |
| `campaign_email` | `sent`, `first_sent`, `opened`, `bounced`, `clicked` |
| `campaign_reply` | `received`, `first_received`, `autoreply_received` |
| `campaign_li_reply` | `received`, `first_received` |
| `campaign_li` | `connection_request_accepted` |
| `company` | `found_domains_by_names`, `found_company_by_domain` |
| `prospect` | `found_by_li_url`, `found_emails_by_name_by_domain`, `campaign_finished`, `unsubscribed`, `found_company_by_domain` |
| `email_verification` | `verified` |
| `email` | `found_emails_by_domain`, `found_generic_contacts_by_domain`, `found_prospect_emails` |
| `database_search` | `task_result` |

Start with `campaign_email.bounced`, `campaign_reply.received` and `prospect.unsubscribed` — the
three that carry consequences you cannot recover from by polling later.

## Step 4 — Audit and repair

`listWebhooks` — `GET /v2/webhooks`. Returns each subscription's `id`, `end_point`,
`event_object`, `event_action`, `status` (`active` or `deactivated`) and `created_at` (a Unix
timestamp integer, unlike the ISO strings elsewhere in the API), plus a `meta.webhooks_count`.

**Run this on a schedule.** Auto-deactivation after 7 failed retries is the failure mode that
bites: your receiver has an outage, Snov.io gives up 38 hours later, and nothing tells you.
Compare `status` against your expected set and re-arm anything that flipped to `deactivated`.

`updateWebhook` — `PUT /v2/webhooks/{id}` with `{"status":"active"}` to re-enable, or
`"deactivated"` to pause. `deleteWebhook` — `DELETE /v2/webhooks/{id}` to remove.

## Alternative: per-request callbacks

Distinct from subscriptions. Most async `/start` operations accept a `webhook_url` input
parameter that pushes **that one task's** completed result to the URL instead of requiring you to
poll `/result` with the `task_hash`. Applies to `startDomainSearch`, `startDomainProspectSearch`,
`startDomainEmailsSearch`, `startGenericContactsSearch`, `startFindEmailsByName`,
`startFindDomainByCompanyName`, `startLinkedInProfileEnrichment` and `startEmailVerification`.

Use it for one-off jobs; use subscriptions for ongoing campaign telemetry.

## Caution

There is no AsyncAPI document and no payload schema for any of the 20 events. Snov.io publishes
the trigger conditions but not the body shape, so write a tolerant parser and log unrecognised
fields rather than validating strictly.
