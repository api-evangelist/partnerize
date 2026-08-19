---
name: Submit bulk conversions to Partnerize and monitor the job
description: Submit a bulk conversion create/update to the Partnerize v3 API, then poll the asynchronous job and its tasks to completion, replaying only the tasks that failed.
api: openapi/partnerize-campaign-conversions-api-openapi.yml
operations:
  - v3-conversion-bulk-update
  - get_jobs
  - get_job
  - get_job_tasks
  - get_task
  - replay_job
  - Export Conversions
  - Export Clicks
  - Export Conversion Items
generated: '2026-08-13'
method: generated
source: openapi/*.yml + https://api-docs.partnerize.com/brand/
---

# Submit bulk conversions to Partnerize and monitor the job

Bulk conversion submission is the highest-consequence operation in the Partnerize
API: each item can create a commission liability, and there is **no idempotency
key**. Read the retry rule at the bottom before you send anything.

## Before you start

`Authorization: Basic base64(<application_key>:<user_api_key>)` over HTTPS,
against `https://api.partnerize.com`. See
`authentication/partnerize-authentication.yml`.

## Steps

1. **Build the payload.** `v3-conversion-bulk-update`
   (`POST /v3/brand/campaigns/{campaignID}/conversions/bulk`) accepts exactly
   **one** of `conversions`, `conversion_items` or `conversion_references` per
   request, each capped at **100,000 items**. Sending more than one collection in
   a single request is invalid.

2. **Send only recognised fields.** v3 ignores unrecognised headers and
   unrecognised query-string parameters, but returns `400 Bad Request` for an
   unrecognised parameter **in the request body**. An extra field you thought was
   harmless fails the whole request.

3. **Capture the job id.** A successful submission returns `202 Accepted` with a
   job id, not the created resources. Treat the id as the receipt — if you lose
   it you cannot tell whether the work landed.

4. **Poll the job.** `get_job` (`GET /v3/jobs/{jobID}`) returns `status`,
   `percentage_complete`, `created_at`, `started_at`, `completed_at` and `type`.
   The response also carries `hypermedia.links.tasks` pointing at the task
   collection — follow that link rather than constructing the URL.

5. **Inspect the tasks.** `get_job_tasks` (`GET /v3/jobs/{jobID}/tasks`) lists the
   individual units of work and `get_task`
   (`GET /v3/jobs/{jobID}/tasks/{taskID}`) returns detail on one. A job can be
   `complete` with individual tasks having failed — check the tasks, not just the
   job status.

6. **Replay only what failed.** `replay_job` exists at two paths: the whole job
   (`POST /v3/jobs/{jobID}/replay`) and a single task
   (`POST /v3/jobs/{jobID}/tasks/{taskID}/replay`). Both share the operationId
   `replay_job` in the published spec, so **select by path, not by operationId**.
   Prefer the task-level replay: replaying the whole job re-runs items that
   already succeeded.

7. **Find historic jobs.** `get_jobs` (`GET /v3/jobs`) lists the jobs for the
   authenticated user — use it to recover a lost job id or to audit what ran.

## Bulk exports

The same asynchronous pattern covers exports: `Export Conversions`
(`GET /reporting/export/export/conversion.csv`), `Export Clicks`
(`/click.csv`) and `Export Conversion Items` (`/conversion_item.csv`).

## Rules

- **Retry rule — this is the important one.** Partnerize publishes no
  `Idempotency-Key` header or parameter anywhere in the API. If a bulk submission
  times out or the connection drops, **do not resend it**. Call `get_jobs`, find
  whether a job was created for that campaign in that window, and resolve from
  the job state. Resending can double-post up to 100,000 conversions and every
  duplicate carries a commission.
- **Never run this unattended without a human-approved payload.** Bulk conversion
  create/update writes financial records. See
  `agentic-access/partnerize-agentic-access.yml`.
- Poll with backoff and honour `X-RateLimit-Remaining` /
  `X-RateLimit-Retry-After`; polling a long job tightly is the fastest way to a
  `429`.
- v3 error bodies put a UUID in `error.errors[].code`; resolve it through
  `errors/partnerize-error-codes.yml`. The validation UUIDs are exact and stable,
  which makes them the most reliable thing to branch on in the whole error
  surface.
- `503 Service Unavailable` is declared on twelve operations, six of them named
  "Fraud Service Unavailable" — a named downstream that can fail on its own.
  Treat `503` as retryable with backoff.
