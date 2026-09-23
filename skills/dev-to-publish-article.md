---
name: dev-to-publish-article
description: Publish, update or unpublish an article on DEV (dev.to) via the Forem API V1 on behalf of an authorized account holder, with the AI-disclosure field set correctly.
api: dev-to:dev-to-articles-api
generated: '2026-09-17'
method: generated
source: openapi/dev-to-articles-api-openapi.yml, https://dev.to/llms.txt, https://developers.forem.com/api
operations:
  - createArticle
  - updateArticle
  - unpublishArticle
  - getArticleById
  - getUserAllArticles
---

# Publish an article on DEV

## Before you call anything

1. **You need explicit authorization.** `dev.to/llms.txt` is unambiguous: only publish, edit, comment,
   react or follow "when the account holder has explicitly authorized that action." Publishing is a
   public, attributable act under a human's name.
2. **Set two headers on every request.**
   - `api-key: <the account holder's key>` — generated at <https://dev.to/settings/extensions>.
   - `Accept: application/vnd.forem.api-v1+json` — this is the version selector. Omit it and the same
     URL silently serves the **deprecated V0 API**, which answers with a `299 Warning` header.
3. Base URL is `https://dev.to/api`.

## Disclose AI involvement — this is required, not optional

Send `ai_disclosure_level` on every create and update, in the article payload or in `body_markdown`
front matter. DEV defines three values:

| value | when to use it |
|---|---|
| `no_ai` | written by a human with no meaningful AI assistance |
| `some_ai` | human-authored with meaningful AI assistance (drafting, code generation, major editing, translation) |
| `fully_autonomous` | produced primarily or entirely by an agent or model, even when a human asked for it |

If you drafted the article, it is at minimum `some_ai` and usually `fully_autonomous`. A human merely
reviewing or approving your output does **not** make it human-authored. Omitting the field records
`not_disclosed`, which DEV treats as no disclosure having been provided.

## Steps

1. **Draft, then create as unpublished.**
   `createArticle` — `POST /articles` with `{"article": {"title": ..., "body_markdown": ...,
   "published": false, "tags": [...], "ai_disclosure_level": "..."}}`.
   Returns `201`. Keep the returned `id`.
2. **Show the account holder the draft** and get sign-off before flipping it live.
3. **Publish.** `updateArticle` — `PUT /articles/{id}` with `{"article": {"published": true}}`.
4. **Verify.** `getArticleById` — `GET /articles/{id}`, or list everything you own with
   `getUserAllArticles` — `GET /articles/me/all`.

## Rules that will bite you

- **`createArticle` is NOT idempotent and there is no `Idempotency-Key` header on this API.** A retry
  after a timeout creates a second article. Before retrying a create, call `getUserAllArticles` and
  check whether the title already landed.
- **Rate limits are tight and undiscoverable at runtime.** `POST /articles` is capped at **10 requests
  per 30 seconds** and `PUT /articles/{id}` at **30 per 30 seconds** (per the API reference). No
  `RateLimit-*` or `Retry-After` header is returned, so you cannot read your remaining budget — pace
  yourself and treat `429` as a hard stop with exponential backoff.
- **Publishing is reversible; creating is not.** `unpublishArticle` — `PUT /articles/{id}/unpublish` —
  hides an article. There is **no delete operation for an article anywhere in this API**, and no
  published window on the unpublish. A mistaken create leaves a permanent record.
- **Errors are thin.** Forem returns `{"status": <int>, "error": "<message>"}`, not RFC 9457
  problem+json, and only one 4xx response in the whole contract carries a schema. On `422`, read the
  message, fix the field, and do not retry unchanged. Quote the `x-request-id` response header when
  escalating to `yo@forem.com`.
- **Follow the community rules.** DEV's terms and code of conduct apply: honest, original, relevant,
  no spam or engagement manipulation. Respect any preferences the article's author states.
