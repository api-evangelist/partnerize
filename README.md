# Partnerize (partnerize)

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

Partnerize is an affiliate and partner marketing platform with a REST API for managing publisher partnerships, tracking sales, processing commissions, and accessing performance analytics. The platform provides end-to-end partnership automation for brands and publishers across retail, travel, finance, direct-to-consumer, and subscription industries, tapping into a network of over 750,000 partners and 250,000 influencers.

- APIs.json: https://raw.githubusercontent.com/api-evangelist/partnerize/refs/heads/main/apis.yml
- Naftiko: https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=partnerize-api-evangelist&utm_content=repo

## Tags

- Affiliate Marketing
- Partner Marketing
- Partnerships
- Performance Marketing
- Commissions
- Tracking
- Analytics

## APIs

### Partnerize Brands API

REST API for brands to manage networks, campaigns, payments, commissions, attribution, and partner relationships across the Partnerize platform. Covers Networks and Brands Management (v1), Campaign Management (v1), Payments (v1), Commissions (v2), and Attribution (v2).

- Documentation: https://api-docs.partnerize.com/brand/

### Partnerize Partners API

REST API for publishers and partners to manage campaigns, tracking links, commissions, payments, reporting, and analytics. Covers Partner Management (v1), Campaign Management (v1/v2), Payments (v1), Tracking Links (v2), Reporting (v1), Analytics (v3), Participations (v3), and Partnerize Tags (v3).

- Documentation: https://api-docs.partnerize.com/partner/

## Plans, Rate Limits, and FinOps

- Plans/Pricing: [plans/partnerize-plans-pricing.yml](plans/partnerize-plans-pricing.yml) — Custom enterprise pricing with License (fixed fee) and Performance (percentage-based) models; no publicly listed tiers.
- Rate Limits: [rate-limits/partnerize-rate-limits.yml](rate-limits/partnerize-rate-limits.yml) — Rate limiting enforced; HTTP 429 returned on excess; dual API key authentication required (X-Application-Key, X-User-Api-Key).
- FinOps: [finops/partnerize-finops.yml](finops/partnerize-finops.yml) — FinOps Framework 1.0 alignment covering cost allocation, reporting, optimization, and governance for Partnerize platform spend.

## Timestamps

- Created: 2026-06-13
- Modified: 2026-06-13

## Common

| Type | URL |
|------|-----|
| Website | https://partnerize.com |
| Documentation | https://api-docs.partnerize.com/ |
| GitHub Org | https://github.com/performancehorizongroup |
| LinkedIn | https://www.linkedin.com/company/partnerize |
| Blog | https://partnerize.com/resources/blog |
| Pricing | https://partnerize.com/pricing |
| X | https://twitter.com/partnerize |

## Maintainers

- Kin Lane (kin@apievangelist.com)
