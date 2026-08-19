---
name: Recruit and onboard a partner onto a Partnerize campaign
description: Find partners on a Partnerize network, invite or approve them onto a campaign, confirm the participation, and hand them the tracking links and creatives they need to start promoting.
api: openapi/partnerize-network-partners-api-openapi.yml
operations:
  - listAllPartnersOnTheNetwork
  - get-publishers
  - get-publisher-details
  - inviteDryRun
  - post-v2-networks-network_id-invite-network-partners
  - partner-requests-invites
  - accept-campaign-requests
  - decline-campaign-requests
  - update-partner-participation
  - List all Participating Partners
  - Retrieve Tracking Links
  - Create Tracking Links
  - List all Creatives for a Campaign
generated: '2026-08-13'
method: generated
source: openapi/*.yml + https://api-docs.partnerize.com/brand/
---

# Recruit and onboard a partner onto a Partnerize campaign

The recruitment flow spans three API versions on one host. v1 owns the campaign
and its participations, v2 owns partner discovery and the invite/request queue.
Follow the version segments exactly as written below.

## Before you start

`Authorization: Basic base64(<application_key>:<user_api_key>)` on every request,
over HTTPS. Base host `https://api.partnerize.com`. See
`authentication/partnerize-authentication.yml`.

You need the `network_id` and the `campaign_id`. Note the vocabulary split: the
current term is **partner**, but v1 and v2 paths and payloads say **publisher**.
Both refer to the same entity.

## Steps

1. **Find candidate partners.** `get-publishers`
   (`GET /v2/networks/{network_id}/publishers`) lists partners on the network with
   filtering. `get-publishers-ids` returns just the ids when you only need to
   diff against a list you already hold, and `export-publishers`
   (`POST /v2/networks/{network_id}/publishers.csv`) produces a CSV.

2. **Read the profile before inviting.** `get-publisher-details`
   (`GET /v2/networks/{network_id}/publishers/{publisher_id}`) returns the
   partner's profile — registered websites, promotional methods, traffic sources
   and partnership models. Check these against the campaign's brand-safety rules
   *before* the invite, not after the first conversion.

3. **Dry-run the invite.** `inviteDryRun`
   (`GET /v2/networks/{network_id}/invite/network-partners`) tells you who would
   actually be invited without sending anything, and `exportDryRun` returns the
   same as CSV. Always run this first on a bulk invite.

4. **Send the invite.** `post-v2-networks-network_id-invite-network-partners`
   (`POST /v2/networks/{network_id}/invite/network-partners`).

5. **Work the inbound queue.** Partners also apply to you. `partner-requests-invites`
   (`GET /v2/networks/{networkId}/campaign-requests-invites`) lists requests and
   invites; `partner-requests-invites-filters` returns the available filters and
   `partner-requests-invites-detail` the per-partner view. Then
   `accept-campaign-requests` (`POST /v2/networks/{networkId}/campaign-requests/accept`)
   or `decline-campaign-requests` (`.../decline`).

6. **Set the participation on the campaign.** `update-partner-participation`
   (`POST /campaign/{campaign_id}/publisher`) sets a partner's participation
   status on a campaign. Confirm with `List all Participating Partners`
   (`GET /campaign/{campaign_id}/publisher`) or `Retrieve a Participating Partner`
   for the single record.

7. **Check what they will earn.** Before telling a partner they are live, read
   `get-active-publisher-commissions`
   (`GET /v2/campaigns/{campaignId}/publishers/{publisherId}/commissions/active`).
   A participation with no active commission is a partner promoting for nothing.

8. **Hand over the promotional assets.**
   - Tracking links: `Retrieve Tracking Links`
     (`GET /v2/publishers/{publisher_id}/links`), and `Create Tracking Links`
     (`POST /v2/publishers/{publisher_id}/links`) for a new destination.
   - Creatives: `List all Creatives for a Campaign`
     (`GET /campaign/{campaign_id}/creative`), filtered by tag with
     `List all Creatives for a Campaign by Tags`.
   - Product feeds: `List all Product Feeds`
     (`GET /campaign/{campaign_id}/feed`).
   - Voucher codes: `List all Voucher Codes`
     (`GET /campaign/{campaign_id}/voucher_code`).

9. **Terms and conditions.** If the campaign requires acknowledgement, the
   campaign and network terms resources and their acknowledgment endpoints
   (`openapi/partnerize-campaign-terms-and-conditions-acknowledgments-api-openapi.yml`,
   `openapi/partnerize-network-terms-and-conditions-acknowledgments-v3-api-openapi.yml`)
   record whether a given partner has accepted.

## Rules

- **There is no idempotency key.** Steps 4, 5, 6 and 8 are writes and Partnerize
  publishes no `Idempotency-Key` contract, so a retried invite or participation
  update may be applied twice. Use the dry-run in step 3, and on a timeout
  **re-read state (step 6) rather than re-sending the write**.
- **Approval is a commercial act.** Approving a partner creates a commission
  liability. Do not run steps 4–6 unattended without a human decision on each
  partner — see `agentic-access/partnerize-agentic-access.yml`.
- Bulk invites may return `202 Accepted` with a job id; poll `get_job` rather
  than assuming the invite completed.
- On `403` the message names the exact missing permission and resource id
  (`"You do not have 'read' permission on campaign with id '1'"`). On `404` you
  may be looking at the same problem wearing a different status code — Partnerize
  returns `404` when the user is not allowed to know the resource exists.
