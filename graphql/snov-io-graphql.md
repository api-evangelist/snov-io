# Snov.io GraphQL Schema

Snov.io is a sales automation and lead generation platform serving over 300,000 companies across 180+ countries. Its REST API covers the full outreach lifecycle — prospect discovery, email finding, email verification, drip campaign management, and CRM pipeline operations. This GraphQL schema represents a conceptual graph layer over those capabilities.

## Authentication

Snov.io uses OAuth 2.0 client credentials. Clients exchange a `clientId` and `clientSecret` for a short-lived Bearer token via the `authenticate` mutation. All operations consume credits from the account balance.

## Schema Overview

The schema is organized into the following domains:

### Domain & Email Discovery

- `Domain` — a registered internet domain used as a lookup key for email discovery.
- `DomainSearch` — paginated results from a full domain crawl.
- `DomainEmails` — flat list of email addresses found for a domain.
- `DomainStats` — counts of personal vs. generic emails, verification coverage, and crawl freshness.
- `EmailPattern` — the detected naming convention used within a domain (e.g., `{first}.{last}@`).

### Email Details & Confidence

- `Email` — a discovered email address with source count and confidence metadata.
- `EmailDetails` — enriched email record including name, position, company, and social profiles.
- `ProspectEmail` — an email address tied to a prospect record with primary-flag and verification state.
- `EmailConfidence` — numeric score and label indicating how reliable a discovered address is.

### Email Verification

- `VerificationResult` — full verification output: status, MX record presence, SMTP response, and disposable detection.
- `Deliverable` — binary deliverability assessment with reason string.
- `AcceptAll` — catch-all domain detection (emails may exist even if unverifiable individually).
- `SmtpCheck` — low-level SMTP handshake result.
- `Disposable` — whether the address belongs to a throwaway email provider.

### Identity & Enrichment

- `FullName`, `FirstName`, `LastName` — name components with source attribution.
- `Position` — job title, seniority level, department, and tenure dates.
- `Seniority` — enum: C_LEVEL, DIRECTOR, VP, MANAGER, SENIOR, ENTRY, INTERN, OWNER, PARTNER, UNKNOWN.
- `Department` — enum covering Engineering, Marketing, Sales, Finance, Legal, Operations, HR, Product, Design, Customer Success, IT, Research, Other.

### Company

- `Company` — organisation record with domain, industry, size, location, and social links.
- `CompanyDetails` — extended data including legal name, NAICS/SIC codes, tech stack, funding, and revenue range.
- `CompanySize` — enum: SELF_EMPLOYED, MICRO, SMALL, MEDIUM, LARGE, ENTERPRISE, UNKNOWN.
- `Industry` — industry classification with category grouping.

### Social & Contact Channels

- `LinkedIn` — LinkedIn profile URL and username.
- `Twitter` — Twitter/X handle and follower count.
- `Phone` — phone number with type and country code.
- `Social` — aggregated social presence: LinkedIn, Twitter, Facebook, GitHub, personal website.

### Prospects

- `Prospect` — a sales contact with emails, position, company, social profiles, status, and list memberships.
- `ProspectDetails` — prospect plus campaign history and email timeline.
- `ProspectStatus` — lifecycle enum: NEW, CONTACTED, REPLIED, INTERESTED, NOT_INTERESTED, UNSUBSCRIBED, BOUNCED, DO_NOT_EMAIL, ARCHIVED.
- `ProspectList` — a named segment of prospects.
- `ListProspects` — paginated prospects within a list.
- `CustomField` — arbitrary key-value metadata on a prospect.

### Campaigns

- `Campaign` — multi-step outreach campaign with sender, sequence, schedule, and aggregate stats.
- `CampaignStatus` — enum: DRAFT, ACTIVE, PAUSED, COMPLETED, ARCHIVED.
- `CampaignStats` — sent, delivered, opens, clicks, replies, bounces, unsubscribes, and derived rates.
- `CampaignEmail` — an individual email instance sent to one prospect within a campaign.

### Sequences & Steps

- `Sequence` — ordered list of outreach steps attached to a campaign.
- `SequenceStep` — a single step with type, delay, subject, body, and attachments.
- `SequenceStepType` — enum: EMAIL, LINKEDIN_VIEW, LINKEDIN_CONNECT, LINKEDIN_MESSAGE, CALL, TASK.

### Email Content

- `EmailMessage` — a sent email record with full from/to/subject/body and tracking events.
- `MessageBody` — plain-text and HTML variants of an email body.
- `MessageSubject` — subject line with personalization flag.
- `Attachment` — file attached to an email or sequence step.
- `ScheduledTime` — scheduled send timestamp with timezone.
- `SendingSchedule` — time-window and day-of-week constraints for campaign sends.

### Email Tracking

- `EmailTrack` — a tracking event (sent, delivered, opened, clicked, replied, bounced, unsubscribed).
- `Open` — email open event with IP and user agent.
- `Click` — link click event with destination URL.
- `Reply` — inbound reply from a prospect.
- `Bounce` — delivery failure record with bounce type and reason.
- `Unsubscribe` — opt-out event linked to a campaign.

### Senders & Accounts

- `Sender` — a configured identity (name + email) used for campaign delivery.
- `SenderAccount` — sender inbox with SMTP/IMAP settings and warm-up status.
- `EmailAccount` — general inbox record (warm-up campaigns, reply tracking).
- `SMTP` — outbound mail server configuration.
- `IMAP` — inbound mail server configuration.

### Auth & Usage

- `APIKey` — static API key for authentication.
- `Token` — OAuth 2.0 Bearer token with expiry.
- `UsageStats` — credit balance, monthly request counts by operation type, and reset date.

### Webhooks

- `WebhookSubscription` — subscription to a platform event delivered to an HTTPS endpoint.
- `WebhookEvent` — enum: EMAIL_OPENED, EMAIL_CLICKED, EMAIL_REPLIED, EMAIL_BOUNCED, PROSPECT_UNSUBSCRIBED, PROSPECT_STATUS_CHANGED, CAMPAIGN_COMPLETED.

## Root Operations

### Queries

| Field | Description |
|---|---|
| `domainSearch` | Paginated email discovery for a domain |
| `domainEmails` | All emails found for a domain |
| `domainStats` | Coverage statistics for a domain |
| `emailPattern` | Detected naming pattern for a domain |
| `email` | Enriched details for a single email address |
| `verifyEmail` | Verification result for one address |
| `verifyEmails` | Bulk verification (up to 10 addresses) |
| `prospect` | Detailed prospect record by ID |
| `searchProspects` | Search prospects by name or email |
| `prospectLists` | All prospect lists in the account |
| `listProspects` | Paginated prospects within a list |
| `campaign` | Campaign by ID |
| `campaigns` | All campaigns, optionally filtered by status |
| `campaignStats` | Aggregate stats for a campaign |
| `sequence` | Sequence for a campaign |
| `senders` | Configured sender accounts |
| `usageStats` | Credit and request usage for the account |
| `apiKeys` | API keys on the account |
| `webhooks` | Active webhook subscriptions |
| `company` | Company record by domain |

### Mutations

| Field | Description |
|---|---|
| `startDomainSearch` | Kick off an async domain email search, returns job ID |
| `startEmailFinder` | Async name + domain lookup, returns job ID |
| `startEmailVerification` | Async multi-email verification job |
| `createProspect` | Add a new prospect |
| `updateProspect` | Update prospect fields |
| `deleteProspect` | Remove a prospect |
| `createProspectList` | Create a new list segment |
| `addProspectsToList` | Add prospects to an existing list |
| `createCampaign` | Create a new campaign |
| `updateCampaignStatus` | Change campaign state |
| `deleteCampaign` | Delete a campaign |
| `addSequenceStep` | Append a step to a sequence |
| `updateSequenceStep` | Edit an existing sequence step |
| `deleteSequenceStep` | Remove a step from a sequence |
| `addCampaignRecipients` | Add prospects to a campaign's recipient list |
| `createWebhook` | Register a new webhook endpoint |
| `updateWebhook` | Update webhook URL or active state |
| `deleteWebhook` | Remove a webhook subscription |
| `authenticate` | Exchange client credentials for Bearer token |
| `addDoNotEmail` | Add an address to the suppression list |

## Named Types Summary

Scalars (3): `DateTime`, `URL`, `EmailAddress`

Enums (8): `Seniority`, `Department`, `CompanySize`, `ProspectStatus`, `CampaignStatus`, `VerificationStatus`, `SequenceStepType`, `EmailTrackEvent`, `WebhookEvent`

Object Types (55): `Domain`, `DomainSearch`, `DomainEmails`, `DomainStats`, `EmailPattern`, `Email`, `ProspectEmail`, `EmailConfidence`, `EmailDetails`, `VerificationResult`, `Deliverable`, `AcceptAll`, `SmtpCheck`, `Disposable`, `FullName`, `FirstName`, `LastName`, `Position`, `Company`, `CompanyDetails`, `CompanySize` (enum), `Industry`, `LinkedIn`, `Twitter`, `Phone`, `Social`, `Prospect`, `ProspectDetails`, `ProspectList`, `ListProspects`, `CustomField`, `Campaign`, `CampaignStats`, `CampaignEmail`, `Sequence`, `SequenceStep`, `EmailMessage`, `MessageBody`, `MessageSubject`, `Attachment`, `ScheduledTime`, `SendingSchedule`, `EmailTrack`, `Open`, `Click`, `Reply`, `Bounce`, `Unsubscribe`, `Sender`, `SenderAccount`, `EmailAccount`, `SMTP`, `IMAP`, `APIKey`, `Token`, `UsageStats`, `WebhookSubscription`

Input Types (4): `ProspectInput`, `CustomFieldInput`, `CampaignInput`, `SequenceStepInput`

## Source

Schema derived from public Snov.io API documentation at https://snov.io/api covering email finder, email verifier, prospect management, drip campaigns, and webhooks endpoints.
