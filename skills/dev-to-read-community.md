---
name: dev-to-read-community
description: Read DEV (dev.to) content safely and without authentication — search and list articles, read comments, resolve users and organizations, and walk tags — using the public read surface of the Forem API V1.
api: dev-to:dev-to-articles-api
generated: '2026-09-17'
method: generated
source: openapi/dev-to-articles-api-openapi.yml, openapi/dev-to-comments-api-openapi.yml, openapi/dev-to-users-api-openapi.yml, openapi/dev-to-tags-api-openapi.yml
operations:
  - getArticles
  - getLatestArticles
  - getArticleById
  - getArticleByPath
  - getCommentsByArticleId
  - getCommentById
  - getUser
  - getOrganization
  - getOrgArticles
  - getTags
---

# Read DEV without an account

Most read endpoints on `https://dev.to/api` need **no API key**. You still must send
`Accept: application/vnd.forem.api-v1+json`, or you get the deprecated V0 contract instead.

## Finding articles

- `getArticles` — `GET /articles`. Filters: `tag`, `tags`, `tags_exclude`, `username`,
  `state`, `top` (most popular in the last N days), `collection_id`.
- `getLatestArticles` — `GET /articles/latest`, newest first, unranked.
- `getArticleById` — `GET /articles/{id}` and `getArticleByPath` —
  `GET /articles/{username}/{slug}`. Only the show endpoints return `body_markdown`; the index
  endpoints return a leaner shape without the body.
- `getOrgArticles` — `GET /organizations/{username}/articles` for one organization's output.

## Paging

Page-number paging only: `page` (1-based, default 1) and `per_page`. Per-endpoint defaults vary
(10 / 24 / 30 / 80) and the ceiling is 1000. **There is no total count, no `next` link and no cursor** —
the response is a bare JSON array. You know you have reached the end when a page comes back short or
empty. Do not assume a fixed page size; read what you got.

## Comments, people, tags

- `getCommentsByArticleId` — `GET /comments?a_id={article_id}` returns the threaded tree;
  `getCommentById` — `GET /comments/{id}` returns one subtree.
- `getUser` — `GET /users/{id}`, `getOrganization` — `GET /organizations/{username}`.
- `getTags` — `GET /tags`, ordered by popularity.

## Behave like a good client

- **Honor `https://dev.to/robots.txt`.** It disallows `/search?q=*`, `/mod/*`, `/admin/*`,
  `/reactions?*`, `/connect/@*` and the OAuth callback paths. Use the API, not those pages.
- **Identify your client accurately** in the User-Agent, per `dev.to/llms.txt`.
- **Do not infer private endpoints or evade access controls.** If a resource needs a key, it needs a key.
- **Expect no rate-limit headers.** Nothing tells you how much budget is left. Read conservatively and
  back off hard on `429`.
- `404` here means "not found or not visible to you" — do not retry it.
