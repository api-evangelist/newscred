---
name: newscred-run-content-task-workflow
description: Drive a content production task through Optimizely CMP (Newscred/Welcome) — create it from a work request or directly, attach assets and drafts, move workflow substeps, comment, run brand compliance, and publish — using only published operationIds.
api: Optimizely CMP Open API v3
base_url: https://api.cmp.optimizely.com/v3
generated: '2026-08-26'
method: generated
source: openapi/newscred-cmp-open-api-openapi.json
operations:
  - listWorkRequests
  - createWorkRequest
  - createTaskFromWorkRequest
  - createTask
  - listTasks
  - getTask
  - updateTask
  - getTaskBrief
  - addAssetToTask
  - listTaskAssets
  - addDraftToTaskAsset
  - addCommentToTask
  - listWorkflows
  - getWorkflow
  - updateTaskStep
  - getTaskSubStep
  - updateTaskSubStep
  - addCommentToTaskSubStep
  - getTaskAssetDraftBrandCompliance
  - updateTaskAssetDraftBrandCompliance
  - createTaskPublishingIntent
  - listPublishingChannels
---

# Run a content task through the CMP workflow

## Intake

- `listWorkRequests` (`GET /work-requests`) / `createWorkRequest` (`POST /work-requests`).
- `createTaskFromWorkRequest` (`POST /work-requests/{id}/tasks`) turns an approved request into a
  task. Its optional `inherit_fields_from` body field (added 2026-02-25) takes `workflow` to
  inherit fields from the workflow rather than the work request.
- `createCampaignFromWorkRequest` (`POST /work-requests/{id}/campaigns`) does the campaign
  equivalent.
- Or create standalone with `createTask` (`POST /tasks`).

## Work the task

- `getTask` / `listTasks`. `listTasks` accepts a repeatable `status` filter (added 2026-07-19):
  `Archived`, `Completed`, `Overdue`, `Not Started`, `In Progress`, `On Hold`.
- `updateTask` (`PATCH /tasks/{id}`) accepts only `campaign_id`, `due_at`, `labels`, `owner_id`,
  `start_at`, `title`, `workflow_id`. **`is_archived` is readable and filterable but is not in the
  PATCH body** — you cannot un-archive a task through this API.
- `getTaskBrief`, `addAssetToTask` (`POST /tasks/{id}/assets`), `listTaskAssets`,
  `addDraftToTaskAsset`, `addCommentToTask`, `addCommentToTaskAsset`.

## Move the workflow

- `listWorkflows` / `getWorkflow` to read the workflow definition (both **Experimental**).
- `updateTaskStep` (`PATCH /tasks/{task_id}/steps/{step_id}`, **Experimental**).
- `getTaskSubStep` / `updateTaskSubStep` for substeps;
  `addCommentToTaskSubStep`, `updateTaskSubStepComment`, `deleteTaskSubStepComment` for review threads.
- External systems (Jira, Aha, Veeva PromoMats) plug in via
  `getTaskSubStepExternalWork` / `updateTaskSubStepExternalWork`.

## Brand compliance

- `listBrandComplianceCategories`, then `getTaskAssetDraftBrandCompliance` /
  `updateTaskAssetDraftBrandCompliance` on a specific draft.

## Publish

- `listPublishingChannels` (`GET /publishing-channels`).
- `createTaskPublishingIntent` (`POST /tasks/{task_id}/publishing-intents`, **Experimental**) —
  takes a `channel_id` and returns the intent id.
- Track the result with `getPublishingEvent` and `listPublishingEventMetadata`.

## Watch out

- Operations marked **Experimental** in the reference may be non-operational and their shape may
  change without a version bump. Do not build unattended automation on them.
- No idempotency key exists. A retried `createTask` or `addCommentToTask` creates a duplicate.
- Self-throttle to 10 requests/second; nothing in the response tells you how close you are.
