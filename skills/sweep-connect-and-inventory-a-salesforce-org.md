---
name: Connect a Salesforce org and inventory its metadata
description: Register a Salesforce org with Sweep, complete the OAuth handshake, then walk the org's objects,
  record types, users and queues so an agent has a grounded picture of the org before proposing any change.
api: openapi/sweep-api-openapi.yml
generated: '2026-08-14'
method: generated
source: openapi/sweep-api-openapi.yml (harvested from https://api.sweep.io/api-json) + conventions/sweep-conventions.yml
  + errors/sweep-problem-types.yml
operations:
- CrmOrgsController_getOrgs
- CrmOrgsController_createOrgs
- CrmOrgsController_getSFDCOAuthPath
- CrmOrgsController_handleSFDCOAuthAuthorizationCode
- CrmOrgsController_getOrg
- CrmOrgsController_getSalesforceObjectTypes
- CrmOrgsController_getObjectTypes
- CrmOrgsController_getAllFunnelsAndRecordTypes
- CrmOrgsController_listUsers
- CrmOrgsController_listQueues
---

# Connect a Salesforce org and inventory its metadata

Register a Salesforce org with Sweep, complete the OAuth handshake, then walk the org's objects, record types, users and queues so an agent has a grounded picture of the org before proposing any change.

## Before you start

- Base URL: `https://api.sweep.io`
- Auth: `Authorization: Bearer <JWT>` on every operation below. There is no API-key header and no OAuth flow on the REST API; a 401 comes back as `{"message":"Unauthorized","statusCode":401}` with no `WWW-Authenticate` hint.
- Content type: `application/json` in both directions.
- The contract is at `https://api.sweep.io/api-json` (Swagger UI at `https://api.sweep.io/api`). It declares **no** 4xx or 5xx responses, so treat every error as undocumented and read the status code, not the schema.
- **There is no idempotency contract.** Do not blind-retry a POST/PUT/PATCH/DELETE — a repeat may create or deploy a second object. Re-read state with the matching GET before retrying.
- **There are no rate-limit headers.** Nothing tells you how close you are to a limit; back off on your own schedule.
- Pagination is inconsistent: some list operations take `limit`/`offset`, most return an unbounded array with no total. Never assume a page envelope.

## Steps

1. **List what is already connected** — `GET /crm-orgs` (`CrmOrgsController_getOrgs`). If the target org is present, skip to step 5 with its `orgId`. Connecting a second time is not idempotent.
2. **Register the org** — `POST /crm-orgs/new` (`CrmOrgsController_createOrgs`).
3. **Get the Salesforce OAuth URL** — `GET /crm-orgs/org/{orgId}/salesforce-oauth` (`CrmOrgsController_getSFDCOAuthPath`). This is a human step: a Salesforce admin must approve it in a browser. An agent must hand the URL over, not attempt the flow.
4. **Exchange the authorization code** — `POST /crm-orgs/salesforce-oauth2-code` (`CrmOrgsController_handleSFDCOAuthAuthorizationCode`). If the org uses a customer-owned External Client App, the callback must be exactly `https://app.sweep.io/salesforce-oauth2-redirect` with scopes `api` and `refresh_token` (see the provider's connection guide).
5. **Confirm the connection** — `GET /crm-orgs/org/{orgId}` (`CrmOrgsController_getOrg`).
6. **Inventory** — `GET /crm-orgs/org/{orgId}/objects` (`CrmOrgsController_getSalesforceObjectTypes`), then `GET /crm-orgs/org/{orgId}/object-types` (`CrmOrgsController_getObjectTypes`) and `GET /crm-orgs/org/{orgId}/funnels-and-record-types` (`CrmOrgsController_getAllFunnelsAndRecordTypes`).
7. **People and routing targets** — `GET /crm-orgs/org/{orgId}/users` (`CrmOrgsController_listUsers`) and `GET /crm-orgs/org/{orgId}/queues` (`CrmOrgsController_listQueues`).

**Stop conditions.** `POST /crm-orgs/org/{orgId}/connect-anyway` (`CrmOrgsController_connectAnyway`) exists to force a connection past Sweep's own pre-flight objections. An agent should never call it without an explicit human instruction — it is the operation that overrides a safety check.
