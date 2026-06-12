# Snov.io (snov-io)

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
