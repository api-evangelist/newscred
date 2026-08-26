---
name: newscred-subscribe-to-cmp-webhooks
description: Receive Optimizely CMP (Newscred/Welcome) events over webhooks — register subscriptions, validate the Callback-Secret header, and process the 43 published events correctly given at-least-once, out-of-order delivery.
api: Optimizely CMP Open API v3
base_url: https://api.cmp.optimizely.com/v3
generated: '2026-08-26'
method: generated
source: asyncapi/newscred-cmp-webhooks.yml
operations:
  - getSettings
  - updateSettings
  - getTask
  - listTaskAssets
  - getImage
  - getCampaign
  - getWorkRequest
---

# Subscribe to CMP webhooks

## Registering

- In the product: **Admin > Apps and Webhooks**. Supply an HTTPS callback URL, the event names,
  and an optional secret (max 32 characters).
- Programmatically: `getSettings` (`GET /settings`) and `updateSettings` (`POST /settings`,
  **Experimental**) import/export webhook configuration. Import is merge-by-name: a webhook whose
  name already exists is updated and its events are merged; a new name creates a new webhook.

## Verifying a delivery

- If you registered a secret, CMP echoes it verbatim in the **`Callback-Secret`** request header.
- This is a shared-secret echo, **not an HMAC over the payload**. It proves the sender knows the
  secret; it does not prove the body was not altered. Terminate TLS yourself and compare in
  constant time.

## Responding

- Return **`200` or `202`**. Anything else is a delivery failure.
- Respond within **30 seconds** or the connection is cut and the delivery is failed.
- On failure CMP retries with exponential backoff for **up to three days**.

## Processing correctly

- **Delivery is at-least-once.** The same event can arrive more than once. Key your processing on
  `data.<entity>.id` plus the event name and make it idempotent.
- **Order is not guaranteed.** Do not reconstruct state from the event stream. Payloads are
  reference-style — an id plus a `links` object of absolute `https://api.cmp.optimizely.com/v3/...`
  URLs — so call back into the REST API (`getTask`, `getImage`, `getCampaign`,
  `getWorkRequest`, `listTaskAssets`) to read current state.
- Follow the URL in `links.self` rather than building the path yourself; the docs say so explicitly.

## The 43 events

- **Library** (4): `asset_added`, `asset_removed`, `asset_modified`, `asset_renditions_created`
- **Task** (17): `task_added`, `task_removed`, `task_metadata_modified`, `task_brief_added`,
  `task_brief_removed`, `task_brief_modified`, `task_asset_added`, `task_asset_removed`,
  `task_asset_modified`, `task_asset_draft_added`, `task_asset_draft_removed`,
  `task_custom_field_modified`, `content_preview_requested`, `workflow_sub_step_updated`,
  `workflow_sub_step_comment_added`, `workflow_sub_step_comment_modified`,
  `workflow_sub_step_comment_removed`
- **Campaign** (5): `campaign_added`, `campaign_metadata_modified`, `campaign_brief_added`,
  `campaign_brief_removed`, `campaign_brief_modified`
- **Publish** (3): `asset_published`, `asset_synced`, `asset_unpublished`
- **Work request** (6): `work_request_added`, `work_request_modified`, `work_request_removed`,
  `work_request_comment_added`, `work_request_comment_modified`, `work_request_comment_removed`
- **External work management** (6): `external_sub_step_started`, `external_sub_step_completed`,
  `external_sub_step_modified`, `external_sub_step_comment_added`,
  `external_sub_step_comment_modified`, `external_sub_step_comment_removed`
- **Event** (2): `event_added`, `event_modified`

Note `task_metadata_modified` ships a payload whose `event_name` value is documented as
`task_metadata_modifed` (sic) — match on both spellings defensively.

## Compatibility

CMP considers these webhook changes non-breaking: adding a field to the payload, adding a value to
an enum, adding a request header. Parse permissively and ignore unknown fields.
