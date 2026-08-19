---
name: snov-io-build-prospect-list-from-domain
description: >-
  Crawl a company domain with Snov.io to discover its people and email addresses, then persist
  the good ones into a named prospect list. Use when you have a company domain and need a
  durable, reusable list of contacts rather than a one-off lookup.
generated: '2026-08-13'
method: generated
source: openapi/snov-io-domain-search-api-openapi.yml, openapi/snov-io-prospects-api-openapi.yml, https://snov.io/api
api: Snov.io Domain Search API
base_url: https://api.snov.io
operations:
  - getAccessToken
  - getDomainEmailsCount
  - startDomainSearch
  - getDomainSearchResult
  - startDomainProspectSearch
  - getDomainProspectSearchResult
  - startDomainEmailsSearch
  - getDomainEmailsSearchResult
  - startGenericContactsSearch
  - getGenericContactsSearchResult
  - createProspectList
  - addProspect
  - getProspectCustomFields
  - viewProspectsInList
  - listProspectLists
---

# Build a prospect list from a company domain

Nothing in Snov.io's discovery flow is durable until you write it to a list. Task results expire
with their `task_hash` — if you lose the hash, you lose the credits you spent. Create the list
first, write as you go.

## Step 1 — Token

`getAccessToken` — `POST /v1/oauth/access_token`, `grant_type=client_credentials`. 3600-second
Bearer token, no refresh grant.

## Step 2 — Size the job for free

`getDomainEmailsCount` — `POST /v1/get-domain-emails-count` with the domain. **This endpoint is
free.** It tells you how many addresses Snov.io holds for the domain before you spend anything.
Use it to decide whether the crawl is worth the credits at all.

## Step 3 — Create the destination list

1. `getProspectCustomFields` — `GET /v1/prospect-custom-fields`. Read the account's custom-field
   definitions **before** you write anything. You must supply the account's own field keys;
   there is no generic metadata bag.
2. `createProspectList` — `POST /v1/lists` with the list name. Keep the returned `id`.

## Step 4 — Choose the right search

Four independent crawls, each with its own start/result pair and its own credit cost. Pick one;
do not run all four out of habit.

| Goal | Start | Result |
| --- | --- | --- |
| Company profile + summary | `startDomainSearch` | `getDomainSearchResult` |
| Named people at the domain | `startDomainProspectSearch` | `getDomainProspectSearchResult` |
| All addresses at the domain | `startDomainEmailsSearch` | `getDomainEmailsSearchResult` |
| Role addresses (info@, sales@) | `startGenericContactsSearch` | `getGenericContactsSearchResult` |

Each `/start` returns `{"data":{"task_hash":"<32 hex>"}}`. Poll the matching `/result` with
`?task_hash=...` until `status` is `completed`. Persist the `task_hash` somewhere durable the
moment you receive it — it is the only handle on work you have already paid for.

Each `/start` also accepts `webhook_url`. Supply it to be pushed the completed result instead of
polling. Deliveries are unsigned; do not treat the payload as authenticated.

## Step 5 — Write results into the list

For each prospect worth keeping, call `addProspect` — `POST /v1/add-prospect-to-list` with the
list `id` from Step 3, the email, name, position, company and any custom fields whose keys you
read in Step 3.

Write incrementally as results arrive. There is **no idempotency key**, so build your own guard:
keep a client-side set of emails already written and skip duplicates rather than relying on the
API to dedupe.

## Step 6 — Confirm

- `viewProspectsInList` — `POST /v1/prospect-list` with the list id, paged with `page` and
  `per_page` (allowed values 20, 50, 100).
- `listProspectLists` — `GET /v1/get-user-lists` to confirm the list count landed as expected.

## Pacing and cost

- 60 requests per minute, account-wide, with no rate-limit headers. One request per second.
- Credits are charged **per result**, not per request — a crawl that returns 500 people costs
  far more than the one call implies. Size with Step 2 first.
- Follow-on verification of the addresses you collected is a separate skill:
  `snov-io-find-and-verify-emails`.

## Failure handling

401 means the 3600-second token aged out — re-exchange and retry once. 404 on a list id means
the list does not belong to this account. 400 returns an `errors` array; fix the input rather
than retrying. Rate-limit and credit-exhaustion responses are undocumented — guard both
proactively.
