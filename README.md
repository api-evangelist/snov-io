# Snov.io (snov-io)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Snov.io is a sales automation and lead generation platform trusted by over 300,000 companies across 180+ countries. The platform provides a REST API enabling programmatic access to email finding, domain search, email verification, drip campaign management, CRM contact management, and LinkedIn prospect automation. Authentication uses OAuth 2.0 client credentials to obtain short-lived Bearer tokens, and all API operations consume credits from the account balance.

APIs.json: https://raw.githubusercontent.com/api-evangelist/snov-io/refs/heads/main/apis.yml

Naftiko: https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=snov-io-api-evangelist&utm_content=repo

## Tags

- Sales Automation
- Email Finder
- Email Verification
- Lead Generation
- Drip Campaigns
- CRM
- LinkedIn Automation
- Prospect Management
- Data Enrichment
- Cold Email

## APIs

| Name | Description |
|------|-------------|
| Email Finder API | Find email addresses by domain, name, or LinkedIn URL using async start/result pattern |
| Email Verification API | Verify deliverability of up to 10 email addresses per request |
| Campaigns API | Create and manage multi-channel outreach campaigns with analytics |
| Prospect Management API | Add, search, and manage prospect records, lists, and CRM pipelines |
| Email Warm-up API | Create and manage deliverability warm-up campaigns |
| Webhooks API | Subscribe to real-time platform event notifications |

## Plans, Rate Limits, and FinOps

| Resource | Details |
|----------|---------|
| Plans | [plans/snov-io-plans-pricing.yml](plans/snov-io-plans-pricing.yml) |
| Rate Limits | [rate-limits/snov-io-rate-limits.yml](rate-limits/snov-io-rate-limits.yml) |
| FinOps | [finops/snov-io-finops.yml](finops/snov-io-finops.yml) |

**Rate limit:** 60 requests per minute (all endpoints). OAuth Bearer tokens expire after 3,600 seconds.

**Pricing model:** Credit-based. Plans range from free Trial (50 credits) to Custom Ultra (200,000+ credits). Starter plan at $39/month is the entry point for API access.

## Timestamps

- Created: 2026-06-12
- Modified: 2026-06-12

## Common Properties

| Type | URL |
|------|-----|
| Website | https://snov.io/ |
| Documentation | https://snov.io/api |
| Knowledgebase | https://snov.io/knowledgebase/ |
| GitHub Organization | https://github.com/snovio |
| LinkedIn | https://www.linkedin.com/company/snovio |
| Blog | https://snov.io/blog/ |
| Pricing | https://snov.io/pricing |
| X | https://x.com/snov_io |
| Authentication | https://snov.io/knowledgebase/how-to-use-snov-io-api/ |

## Maintainers

- Kin Lane / kin@apievangelist.com
