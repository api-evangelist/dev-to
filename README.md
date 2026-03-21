# Dev.to (dev-to)
Dev.to is a community platform for software developers to share articles, tutorials, and discussions, built on the open-source Forem framework. Their developer platform provides a RESTful API and webhooks for programmatic access to articles, comments, users, organizations, tags, and real-time event notifications.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/dev-to/refs/heads/main/apis.yml)

## Scope

- **Type:** Contract
- **Position:** Consuming
- **Access:** 3rd-Party

## Tags:

 - Blogging, Developer Community, Articles, Content Management, Webhooks

## Timestamps

- **Created:** 2026-03-20
- **Modified:** 2026-03-20

## APIs

### Dev.to Forem API
The Dev.to Forem API (v1) is a RESTful API that provides programmatic access to the Dev.to developer community platform, which is built on the open-source Forem framework. The API enables developers to create, read, update, and manage articles (blog posts, discussions, help threads), comments, users, organizations, tags, followers, listings, and webhooks. It uses API key authentication, requires an accept header of application/vnd.forem.api-v1+json, and returns JSON responses. Unauthenticated endpoints are CORS-enabled, making it possible to fetch public content directly from browser-based applications.

**Human URL:** [https://developers.forem.com/api/v1](https://developers.forem.com/api/v1)


#### Tags:

 - Blogging, Developer Community, Articles, Content Management, Social, Tags

#### Properties

- [Documentation](https://developers.forem.com/api/v1)
- [OpenAPI](openapi/dev-to-forem-api-openapi.yml)

### Dev.to Webhooks API
The Dev.to Webhooks API allows developers to subscribe to real-time notifications for events occurring on the Dev.to platform. By creating webhook subscriptions, applications can receive HTTP callbacks when specific events happen, such as new articles being published. This enables event-driven integrations and automation workflows that respond immediately to activity on the platform without the need to continuously poll the REST API for changes.

**Human URL:** [https://developers.forem.com/api/v1](https://developers.forem.com/api/v1)


#### Tags:

 - Webhooks, Events, Notifications, Automation

#### Properties

- [Documentation](https://developers.forem.com/api/v1)
- [AsyncAPI](asyncapi/dev-to-webhooks-asyncapi.yml)

## Common Properties

- [Developer Portal](https://developers.forem.com)
- [Documentation](https://developers.forem.com/api/v1)
- [Website](https://dev.to)
- [PrivacyPolicy](https://dev.to/privacy)
- [TermsOfService](https://dev.to/terms)
- [Blog](https://dev.to/devteam)
- [Login](https://dev.to/enter)

## Maintainers

**FN:** API Evangelist

**Email:** info@apievangelist.com
