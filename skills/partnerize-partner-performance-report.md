---
name: Report on partner performance in a Partnerize campaign
description: Pull click and conversion performance for a Partnerize campaign from the v3 analytics surface, then drill into the granular conversion report when a figure needs explaining.
api: openapi/partnerize-conversions-api-openapi.yml
operations:
  - postConversionsCount
  - postConversionsFilter
  - postConversionsTimeseries
  - postConversionsExplode
  - postClicksCount
  - postClicksTimeseries
  - brand-conversions
  - listAllCampaigns
generated: '2026-08-13'
method: generated
source: openapi/*.yml + https://api-docs.partnerize.com/brand/
---

# Report on partner performance in a Partnerize campaign

Use this when someone asks how a partner program, a campaign, or an individual
partner is performing. The v3 analytics surface answers aggregate questions
cheaply; the v1 granular reporting endpoints answer "which conversions exactly"
and are the ones that hurt if you page them badly.

## Before you start

Authenticate with HTTP Basic. Build the header yourself — there is no SDK:

```
Authorization: Basic base64(<application_key>:<user_api_key>)
```

Both keys come from the Partnerize console under Settings → Account settings
(`application_key` is the *User Application Key*, `user_api_key` is the *User API
key*). See `authentication/partnerize-authentication.yml`. **The published OpenAPI
declares no securityScheme**, so nothing in the spec will tell you this.

Base host is `https://api.partnerize.com`. Which version segment you use depends
on the operation, and the response envelope changes with it — see
`conventions/partnerize-conventions.yml` before you write a parser.

## Steps

1. **Find the campaign.** `listAllCampaigns` (`GET /campaign`) returns every
   campaign the credential can see. If you already have the id, skip to step 2.
   Watch the `testMode` and `hidden` flags — a test-mode campaign will produce
   figures that are not real revenue.

2. **Get the headline numbers.** `postConversionsCount`
   (`POST /v3/brand/analytics/conversions/count`) and `postClicksCount`
   (`POST /v3/brand/analytics/clicks/count`) take a filter body and return a
   count. Run both so you can state a conversion rate rather than a bare number.

3. **Get the shape over time.** `postConversionsTimeseries`
   (`POST /v3/brand/analytics/conversions/timeseries`) and
   `postClicksTimeseries` return the series. Use the same filter body as step 2
   so the totals reconcile.

4. **Break down by dimension.** `postConversionsExplode`
   (`POST /v3/brand/analytics/conversions/explode`) splits the result across a
   dimension — partner, country, device, creative, product, traffic source,
   adref, pubref, custref. This is where "which partners drove it" is answered.

5. **Drill to individual conversions.** `postConversionsFilter` returns the
   filtered set from v3. For the full granular export with every field, use the
   v1 report instead: `brand-conversions`
   (`GET /reporting/report_advertiser/campaign/{campaign_id}/conversion.{format}`),
   with `format` of `json`, `xml` or `csv`.

6. **Page correctly — this is the step people get wrong.** The granular
   conversion report is **cursor-paginated**, not offset-paginated. Send `limit`
   (max **300**) and then follow `hypermedia.pagination.next_page`, which already
   carries the `cursor_id`. Do **not** compute `offset` here. Every other
   collection in the API uses `offset`/`limit` with `offset + limit < count`
   meaning "more to fetch".

## If you are on the partner side

Every analytics operation above exists symmetrically under `/v3/partner/...`
with the *same* operationId, and the granular report becomes `partner-conversions`
(`GET /reporting/report_publisher/publisher/{publisher_id}/conversion.{format}`).
The credential decides which side you can read; the operationId does not.

## Rules

- **Rate limits.** Read `X-RateLimit-Remaining` on every response and stop before
  it reaches zero. On `429`, wait `X-RateLimit-Retry-After` seconds. Partnerize
  publishes no threshold numbers, so the headers are your only signal, and `429`
  is declared on **zero** of the 327 operations in the spec — do not expect a
  generated client to model it.
- **Live data drifts between pages.** Partnerize documents that reports are built
  from live data and can change mid-page. If a total must reconcile, capture the
  `count` from the first page and note the drift rather than re-paging.
- **Errors.** On `400`, read `error.errors[]` — each entry names the offending
  `property` and a `code`. On v3 that code is a UUID; look it up in
  `errors/partnerize-error-codes.yml`. A `404` may mean the resource does not
  exist **or** that this user lacks read permission on it; the two are
  deliberately indistinguishable.
- Large exports run asynchronously: a `202 Accepted` with a job id means poll
  `get_job` — see the bulk-jobs skill.
