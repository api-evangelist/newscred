---
name: newscred-upload-asset-to-library
description: Upload a file into the Optimizely CMP (Newscred/Welcome) Digital Asset Management library, using either the single pre-signed upload URL or the multipart flow for large files, then register it as a library asset and set its fields and permissions.
api: Optimizely CMP Open API v3
base_url: https://api.cmp.optimizely.com/v3
generated: '2026-08-26'
method: generated
source: openapi/newscred-cmp-open-api-openapi.json
operations:
  - getUploadUrl
  - createMultipartUpload
  - completeMultipartUpload
  - getMultipartUploadStatus
  - createAsset
  - listAssets
  - getImage
  - updateImage
  - listAssetFields
  - updateAssetFields
  - listAssetPermissions
  - addAssetPermissions
  - createAssetVersion
---

# Upload an asset to the CMP library

Every operationId below is taken from the published CMP OpenAPI. Do not invent endpoints.

## Before you start

- Get an OAuth 2.0 access token from `https://accounts.cmp.optimizely.com/o/oauth2/v1/token`
  (`authorization_code` or `client_credentials`). Scopes: `openid profile offline_access`.
- Send it as `Authorization: Bearer <access_token>` against `https://api.cmp.optimizely.com/v3`.
- **There is no idempotency key.** If a `createAsset` call times out you cannot safely blind-retry
  it — list first (`listAssets`) and check whether the asset already landed.

## Small files — single upload URL

1. `getUploadUrl` (`GET /upload-url`) — returns a pre-signed URL to PUT the bytes to.
2. PUT the file bytes to that URL directly. This is object storage, not the CMP API; do not send
   the CMP bearer token to it.
3. `createAsset` (`POST /assets`) — register the uploaded object as a library asset. Returns `201`.

## Large files — multipart

1. `createMultipartUpload` (`POST /multipart-uploads`).
2. Upload each part to the URLs returned.
3. `completeMultipartUpload` (`POST /multipart-uploads/{id}/complete`).
4. Poll `getMultipartUploadStatus` (`GET /multipart-uploads/{id}/status`) until it reports done.
5. `createAsset` (`POST /assets`).

## After the asset exists

- Read it back with `getImage` / `getVideo` / `getRawFile` / `getArticle` by type.
- `listAssetFields` / `updateAssetFields` (`PUT /assets/{asset_id}/fields`) to set organization
  custom fields. `updateAssetField` sets a single field.
- `listAssetPermissions` / `addAssetPermissions` to grant access; `removeAssetPermission` revokes.
- `createAssetVersion` (`POST /assets/{asset_id}/versions`) to add a new version of an existing asset.
- `updateImage` accepts `allow_se_indexing` (added 2026-07-22) to control search-engine indexing.

## Errors and retries

- `401` — token expired. Refresh once at the token endpoint and retry.
- `403` — the app or user lacks permission in this CMP organization. Do **not** retry.
- `404` — the id is not visible to this organization; re-resolve it from a list endpoint.
- `422` — validation failure. The body is `{detail: [{loc, msg, type}]}`; read `loc` for the field.
- The error envelope is `{message, errors}` — **not** RFC 9457 problem+json, and there is no
  machine-readable error code. Branch on the HTTP status.
- **No `429` is declared anywhere in the contract** even though limits are published
  (10 requests/second and 200,000 requests/month per organization) and no `X-RateLimit-*` or
  `Retry-After` headers are returned. Self-throttle to 10 rps; you will get no warning.

## Reversibility

`deleteImage` / `deleteVideo` / `deleteRawFile` / `deleteFolder` have **no documented undo and no
recovery window**. There is no restore, untrash or undelete operation in the API. Treat every
delete as permanent and confirm with a human before calling one.
