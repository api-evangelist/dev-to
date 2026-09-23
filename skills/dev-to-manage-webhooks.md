---
name: dev-to-manage-webhooks
description: Subscribe to and manage DEV (dev.to) article lifecycle webhooks through the Forem API, and reconcile the fact that the webhook operations are absent from the currently published V1 OpenAPI.
api: dev-to:dev-to-webhooks-api
generated: '2026-09-17'
method: generated
source: openapi/dev-to-webhooks-api-openapi.yml, asyncapi/dev-to-webhooks-asyncapi.yml
operations:
  - getWebhooks
  - createWebhook
  - getWebhookById
  - deleteWebhook
---

# Manage DEV webhooks

Webhooks let you receive an HTTP POST when article lifecycle events fire, instead of polling
`GET /articles` on a timer.

## Read this first — a contract discrepancy

The webhook operations are documented in the Forem API reference and are described by
`openapi/dev-to-webhooks-api-openapi.yml` in this repo, but they are **not present in the contract
currently served at `https://dev.to/openapi.json`**. If you generate a client from the live V1 spec,
webhooks will be missing from it. Treat the endpoints as real but under-described, and verify against
your account before depending on them in production.

## Steps

1. **List what already exists.** `getWebhooks` — `GET /webhooks`. Do this before every create; there
   is no idempotency mechanism on this API, so a retried create makes a duplicate subscription that
   will double-deliver every event.
2. **Create a subscription.** `createWebhook` — `POST /webhooks` with the target URL and the
   `source`/`events` you want. Returns `201` with the subscription `id`.
3. **Inspect one.** `getWebhookById` — `GET /webhooks/{id}`.
4. **Remove one.** `deleteWebhook` — `DELETE /webhooks/{id}`. This is a hard delete with no restore
   path and no published retention window.

## Receiving events

`asyncapi/dev-to-webhooks-asyncapi.yml` describes the delivery side: DEV POSTs a JSON payload to your
registered URL on article create, update and destroy.

- **Be idempotent on your own side.** Nothing in this API guarantees at-most-once delivery, and there
  is no documented signature header, retry policy or replay window. Key on the article `id` plus the
  event type and discard repeats.
- **Return 2xx quickly** and do the work asynchronously.
- **Do not trust the payload as authorization.** Re-fetch with `getArticleById` if you are about to
  act on the content.

## Auth and limits

`api-key` header plus `Accept: application/vnd.forem.api-v1+json` on every call. Webhook management
carries no separately published rate limit, but the platform-wide `429` applies and no
`RateLimit-*` headers are returned.
