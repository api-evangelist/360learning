---
name: Provision a learner and add them to a group
description: Create a user on the 360Learning platform, activate them, and add them to a learning group so they receive assigned content.
api: openapi/360learning-core-api.json
operations:
  - v2.users.CreateUserController_createUser
  - v2.users.ActivateUserController_activateUser
  - v2.groups.GetGroupsController_getGroups
  - v2.bulk.groups.AddUsersToGroupsController_addUsersToGroups
---

# Provision a learner and add them to a group

Use this flow to onboard a new learner (e.g. from an HR sync) and place them into
a group so they inherit that group's assigned courses and paths.

## Prerequisites
- OAuth 2.0 client credentials with scopes `users:write` and `groups:bulk` (or `groups:write`).
- Get a bearer token from `POST /api/v2/oauth2/token` (grant_type `client_credentials`); it is valid 1 hour.
- Every request needs headers `Authorization: Bearer <token>` and `360-api-version: v2.0`.

## Steps
1. **Create the user** — `v2.users.CreateUserController_createUser` (`POST /api/v2/users`). Requires `users:write`. Supply the user's email and profile fields.
2. **Activate the user** — `v2.users.ActivateUserController_activateUser` (`PUT /api/v2/users/{userId}/activate`) using the id returned in step 1. Requires `users:write`.
3. **Find the target group** — `v2.groups.GetGroupsController_getGroups` (`GET /api/v2/groups`), paginated (Link header, rel="next", 500/page). Requires `groups:read`.
4. **Add the user to the group** — `v2.bulk.groups.AddUsersToGroupsController_addUsersToGroups` (`POST /api/v2/bulk/groups/memberships`). Requires `groups:bulk`. This is asynchronous: it returns a bulk operation id.
5. **Confirm the bulk job** — poll `v2.bulk.operations.GetBulkOperationController_getBulkOperation` (`GET /api/v2/bulk/operations/{bulkOperationId}`) or subscribe to the `bulk.operation.ended` webhook.

## Rules
- 360Learning has **no idempotency key**; guard against duplicate users by checking `GetUsersController_getUsers` before creating.
- On `403` with `"error": "invalid_scope"`, the credential is missing a scope — add it at the credential level.
- Errors return `{code, message}` (camelCase codes); see errors/360learning-problem-types.yml.
