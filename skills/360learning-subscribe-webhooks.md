---
name: Subscribe to platform events via webhooks
description: Create a webhook subscription for 360Learning learning events, verify signatures, and replay failed deliveries.
api: openapi/360learning-webhook-api.json
operations:
  - v2.webhook.subscriptions.CreateSubscriptionController_createSubscription
  - v2.webhook.subscriptions.GetSigningSecretController_getSigningSecret
  - v2.webhook.subscriptions.GetEventsController_getEvents
  - v2.webhook.subscriptions.RetryEventFailuresController_retryEventFailures
---

# Subscribe to platform events via webhooks

Use this flow to receive near-real-time notifications instead of polling — e.g.
react when `user.created`, `user.certificate.awarded`, or `path.updated` fires.

## Prerequisites
- OAuth 2.0 client credentials with scopes `webhooks:write` and `webhookSecrets:read`.
- Bearer token from `POST /api/v2/oauth2/token`; headers `Authorization: Bearer <token>` and `360-api-version: v2.0`.
- A public HTTPS endpoint on your server to receive `POST` payloads.

## Steps
1. **Create a subscription** — `v2.webhook.subscriptions.CreateSubscriptionController_createSubscription` (`POST /api/v2/webhooks/subscriptions`). Requires `webhooks:write`. Supply your endpoint URL and one event type (`resource.action`, e.g. `user.created`).
2. **Fetch the signing secret** — `v2.webhook.subscriptions.GetSigningSecretController_getSigningSecret` (`GET /api/v2/webhooks/subscriptions/{subscriptionId}/secrets`). Requires `webhookSecrets:read`. Use it to verify the signature on every inbound payload.
3. **Verify + process** each POST; respond 2xx quickly and process asynchronously.
4. **Audit deliveries** — `v2.webhook.subscriptions.GetEventsController_getEvents` (`GET /api/v2/webhooks/subscriptions/{subscriptionId}/events`) to inspect delivery status (delivered/failed/pending/sending).
5. **Replay failures** — `v2.webhook.subscriptions.RetryEventFailuresController_retryEventFailures` (`POST /api/v2/webhooks/subscriptions/{subscriptionId}/events/retry-failure`).

## Available event types
user.created/updated/deleted/activated/reactivated/invited, user.assessment.corrected,
user.certificate.awarded, group.created/updated/deleted, path.created/updated/deleted/archived,
path.session.added/removed, course.attempt.closed, xapi.started, bulk.operation.created/ended.
See asyncapi/360learning-webhooks.yml.

## Rules
- Always verify the signature with the per-subscription secret; rotate it via `.../secrets/rotate`.
- Deliveries retry automatically; make your handler idempotent since retries can duplicate.
