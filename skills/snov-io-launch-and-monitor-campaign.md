---
name: snov-io-launch-and-monitor-campaign
description: >-
  Stand up a Snov.io multichannel drip campaign end to end — connect a sender mailbox, create the
  campaign and its email steps, start it, then read the analytics and replies. Use when outreach
  needs to be launched or monitored programmatically rather than through the Snov.io UI.
generated: '2026-08-13'
method: generated
source: openapi/snov-io-campaigns-api-openapi.yml, openapi/snov-io-email-accounts-api-openapi.yml, https://snov.io/api
api: Snov.io Campaigns API
base_url: https://api.snov.io
operations:
  - getAccessToken
  - listEmailAccounts
  - addEmailAccount
  - checkSenderStatus
  - listEmailSchedules
  - createCampaign
  - createEmailStepContent
  - listCampaigns
  - getCampaign
  - updateCampaign
  - changeCampaignState
  - deleteCampaign
  - getCampaignAnalytics
  - getCampaignProgress
  - getCampaignRecipientsActivity
  - getSentEmails
  - getCampaignOpens
  - getCampaignClicks
  - getCampaignReplies
  - addToDoNotEmailList
  - listDoNotEmailLists
---

# Launch and monitor a Snov.io campaign

Campaigns are the sending side of Snov.io and everything hangs off a connected mailbox. Do not
create a campaign before the sender account reports a valid SMTP status — a campaign attached to
a broken mailbox fails silently at send time.

## Step 1 — Token

`getAccessToken` — `POST /v1/oauth/access_token`, `grant_type=client_credentials`. 3600 seconds.

## Step 2 — Get a working sender mailbox

1. `listEmailAccounts` — `GET /v2/sender-accounts/emails`. **Free.** Look for an existing
   account before adding one.
2. If none: `addEmailAccount` — `POST /v2/sender-accounts/emails` with `sender_name`,
   `email_from`, `password`, `smtp` (host/port/encryption) and optionally `imap`, plus
   `limitation` (daily send cap) and `delay` settings.
3. `checkSenderStatus` — `GET /v2/sender-accounts/check-sender-status?sender_account_id=...`.
   Proceed only when `data.smtp.status` is `valid`. When it is `invalid`, `data.smtp.errors` is
   an array of strings (for example `"Connection refused"`) — fix the mailbox first.

## Step 3 — Suppress before you send

`listDoNotEmailLists` — `GET /v2/blacklists` and `addToDoNotEmailList` —
`POST /v1/do-not-email-list`. Push every known opt-out into the suppression list **before**
launch. Snov.io will not retroactively unsend.

## Step 4 — Create the campaign and its steps

1. `listEmailSchedules` — `GET /v2/campaigns/schedules`. Pick a `schedule_id`; a campaign needs
   one to know when it may send.
2. `createCampaign` — `POST /v2/campaigns/create` with `name`, `schedule_id`,
   `sender_account_id` (from Step 2), `track_opens`, `track_clicks`. Keep the returned `id`.
3. `createEmailStepContent` for each step — subject, body, `step_number` and `delay_days`.
   Note: the published path for this operation is
   `POST /v2/campaigns/{campaign_id}/steps/{step_id}/content/create`; confirm the current shape
   against https://snov.io/api before wiring it, as the spec in this repo models fewer path
   parameters than the docs require.

## Step 5 — Start it

`changeCampaignState` — `POST /v2/campaigns/{id}/action` with `{"action":"start"}`.

The response is `{"success": true}` or `{"success": false, "errors": [...]}` — this operation
uses the array-shaped error envelope, not the object-shaped one the router returns. Handle both.

## Step 6 — Monitor

| What | Operation | Endpoint |
| --- | --- | --- |
| Rollup metrics | `getCampaignAnalytics` | `GET /v2/statistics/campaign-analytics` |
| Delivery progress | `getCampaignProgress` | `GET /v2/campaigns/{campaign_id}/progress` |
| Per-recipient activity | `getCampaignRecipientsActivity` | `GET /v2/campaigns/{campaign_id}/recipients-activity` |
| Sent messages | `getSentEmails` | `GET /v1/emails-sent` |
| Opens | `getCampaignOpens` | `GET /v1/get-emails-opened` |
| Clicks | `getCampaignClicks` | `GET /v1/get-emails-clicked` |
| Replies | `getCampaignReplies` | `GET /v1/get-emails-replies` |

`getCampaignAnalytics` returns `total_recipients`, `sent`, `delivered`, `opened`, `clicked`,
`replied`, `bounced`, `unsubscribed` plus `open_rate`, `click_rate` and `reply_rate`.

**Prefer webhooks over polling.** Subscribe to `campaign_email.bounced`, `campaign_reply.received`
and `prospect.unsubscribed` at `POST /v2/webhooks` rather than looping on these read endpoints —
you are capped at 60 requests per minute across the whole account and polling will starve your
discovery calls. See `asyncapi/snov-io-webhooks.yml` for all 20 event actions.

## Step 7 — Wind down

`updateCampaign` — `PATCH /v2/campaigns/{id}` to edit. `deleteCampaign` —
`DELETE /v2/campaigns/{id}`, but deletion is permitted **only** from the `new` (draft),
`complete` or `archived` states. An `active`, `pause` or `scheduled` campaign must be stopped via
`changeCampaignState` first.

## Cautions

- Recipients, not credits, are the campaign currency. A unique contact costs one recipient slot
  per billing period; follow-ups to that contact are free.
- No idempotency. Do not retry `createCampaign` on a timeout — call `listCampaigns`
  (`GET /v2/campaigns/`) and check whether it already exists.
- Campaign endpoints are split across v1 and v2 with no pattern. Read the path column above
  literally; the prefixes are not interchangeable.
