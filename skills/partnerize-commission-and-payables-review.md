---
name: Review Partnerize commissions and reconcile partner payables
description: Read the commission structures configured on a Partnerize campaign, check what an individual partner actually earns, then reconcile that against the payable report, self-bills and brand invoices.
api: openapi/partnerize-commissions-api-openapi.yml
operations:
  - get-commissions
  - get-commission
  - get-publisher-commissions
  - get-publisher-commissions-default
  - get-publisher-commissions-campaign
  - get-publisher-commissions-voucher
  - get-publisher-commissions-promotion
  - get-publisher-commissions-tier
  - get-active-publisher-commissions
  - partner-payable
  - Partner payments summary
  - List all Selfbills
  - Retrieve a Selfbill
  - List all Invoices
  - Retrieve an Invoice
generated: '2026-08-13'
method: generated
source: openapi/*.yml + https://api-docs.partnerize.com/brand/
---

# Review Partnerize commissions and reconcile partner payables

This is the money path. Everything below is a **read**; the write operations
(`create-commission`, `update-commission`, `migrate-commission`,
`retire-commission`) are named at the end and deliberately not part of the flow.

## Before you start

`Authorization: Basic base64(<application_key>:<user_api_key>)` over HTTPS,
against `https://api.partnerize.com`. Commission operations are v2; payables and
invoices are v1. See `authentication/partnerize-authentication.yml`.

## Steps

1. **List the campaign's commission structures.** `get-commissions`
   (`GET /v2/campaigns/{campaignId}/commissions`) returns every structure
   configured on the campaign. `get-commission`
   (`GET /v2/campaigns/{campaignId}/commissions/{commissionId}`) reads one.

2. **Resolve what a specific partner earns.** A partner's effective rate is
   assembled from several layers, and Partnerize exposes each one separately
   under `/v2/campaigns/{campaignId}/publishers/{publisherId}/commissions`:
   - `get-publisher-commissions-default` — `/default`, the fallback
   - `get-publisher-commissions-campaign` — `/campaign`, campaign-level
   - `get-publisher-commissions-voucher` — `/voucher`, voucher-code driven
   - `get-publisher-commissions-promotion` — `/promotion`, promotional
   - `get-publisher-commissions-tier` — `/tier`, tiered
   - `get-publisher-commissions` — the collection across all of them

   **Then call `get-active-publisher-commissions` (`/active`.)** That is the one
   that answers "what is actually in force right now". Reading only the layers
   and inferring the winner will get the answer wrong.

3. **Get what is owed.** `partner-payable`
   (`GET /reporting/report_publisher/publisher/{publisher_id}/payable`) is the
   payable report for a partner. `Partner payments summary`
   (`GET /user/publisher/{publisher_id}/payment/summary`) summarises the value of
   that partner's conversions.

4. **Tie payables to documents.** `List all Selfbills`
   (`GET /user/publisher/{publisher_id}/selfbill`) and `Retrieve a Selfbill`
   return the self-billing invoices Partnerize raises on the partner's behalf.
   On the brand side, `List all Invoices`
   (`GET /user/advertiser/{advertiser_id}/invoice`) and `Retrieve an Invoice`
   cover what the brand is billed.

5. **Explain the variances.** Where a payable does not match expectation, the
   cause is usually a disputed conversion. `List Transaction Queries`
   (`GET /user/publisher/{publisher_id}/tq`) lists the disputes, and each
   `Transaction_Query` carries a reason, a type and a state history. Fraud-driven
   variances come from `get-fraud-metrics`
   (`GET /v2/campaigns/{campaign_id}/fraud/metrics`).

6. **Reconcile against the conversions themselves.** Use the granular report
   `brand-conversions` or `partner-conversions` — cursor-paginated, `limit` max
   **300**, follow `hypermedia.pagination.next_page`. See the partner-performance
   skill.

## Write operations — name them, do not run them unattended

`create-commission` (`POST /v2/campaigns/{campaignId}/commissions`),
`update-commission` (`PATCH .../{commissionId}`),
`migrate-commission` (`POST .../{commissionId}/migrate`) and
`retire-commission` (`POST .../{commissionId}/retire`) change what partners are
paid. Partnerize publishes **no idempotency-key contract**, so a retried or
duplicated call has no supported deduplication path, and `migrate` moves partners
between structures in bulk. Require an explicit human decision per call, and
re-read with `get-active-publisher-commissions` after any write rather than
trusting the response.

## Rules

- Read `X-RateLimit-Remaining`; back off for `X-RateLimit-Retry-After` seconds on
  `429`. Thresholds are not published.
- Collections here are offset-paginated: send `offset` and `limit`, and keep
  paging while `offset + limit < count`. Only the granular conversion report uses
  cursors.
- v2 responses put the resource at the top level next to `execution_time` and
  `count`; v1 responses key by resource name. Do not write one parser for both.
- On `400`, `error.errors[].code` is a slug on v2 and a UUID on v3 — resolve both
  through `errors/partnerize-error-codes.yml`.
- Partner payout details include IBAN and BIC values. Never log, echo or cache a
  `Payment_Details` payload.
