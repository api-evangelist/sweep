---
name: Trace a metadata element and its blast radius
description: Find a Salesforce (or Snowflake, HubSpot, ServiceNow, Workato, Data360) metadata element
  in Sweep's documentation platform, read its generated summary and source, and walk dependencies and
  dependents before anyone changes it.
api: openapi/sweep-api-openapi.yml
generated: '2026-08-14'
method: generated
source: openapi/sweep-api-openapi.yml (harvested from https://api.sweep.io/api-json) + conventions/sweep-conventions.yml
  + errors/sweep-problem-types.yml
operations:
- DocumentationPlatformController_listSalesforceObjects
- DocumentationPlatformController_listSalesforceElementTypes
- DocumentationPlatformController_getSalesforceObjectDetailElements
- DocumentationPlatformController_listSalesforceElements
- DocumentationPlatformController_getSalesforceElementDependencies
- DocumentationPlatformController_getSalesforceElementDependents
- DocumentationPlatformController_searchElements
- DocumentationPlatformController_findElementByName
- DocumentationPlatformController_getElementSummary
- DocumentationPlatformController_getElementSource
- DocumentationPlatformController_chatWithElement
---

# Trace a metadata element and its blast radius

Find a Salesforce (or Snowflake, HubSpot, ServiceNow, Workato, Data360) metadata element in Sweep's documentation platform, read its generated summary and source, and walk dependencies and dependents before anyone changes it.

## Before you start

- Base URL: `https://api.sweep.io`
- Auth: `Authorization: Bearer <JWT>` on every operation below. There is no API-key header and no OAuth flow on the REST API; a 401 comes back as `{"message":"Unauthorized","statusCode":401}` with no `WWW-Authenticate` hint.
- Content type: `application/json` in both directions.
- The contract is at `https://api.sweep.io/api-json` (Swagger UI at `https://api.sweep.io/api`). It declares **no** 4xx or 5xx responses, so treat every error as undocumented and read the status code, not the schema.
- **There is no idempotency contract.** Do not blind-retry a POST/PUT/PATCH/DELETE — a repeat may create or deploy a second object. Re-read state with the matching GET before retrying.
- **There are no rate-limit headers.** Nothing tells you how close you are to a limit; back off on your own schedule.
- Pagination is inconsistent: some list operations take `limit`/`offset`, most return an unbounded array with no total. Never assume a page envelope.

## Steps

This is the flow that makes Sweep useful to an agent: it answers "what breaks if I change this?" before the change.

**Salesforce**
1. `GET /documentation-platform/salesforce/objects` (`DocumentationPlatformController_listSalesforceObjects`) — the object list.
2. `GET /documentation-platform/salesforce/element-types` (`DocumentationPlatformController_listSalesforceElementTypes`) — which element types exist (fields, flows, validation rules, ...).
3. `GET /documentation-platform/salesforce/objects/{objectName}/elements` (`DocumentationPlatformController_getSalesforceObjectDetailElements`) or `GET /documentation-platform/salesforce/{elementType}` (`DocumentationPlatformController_listSalesforceElements`) to locate the element and its `metadataId`.
4. **Blast radius, both directions** — `GET /documentation-platform/salesforce/{metadataId}/dependencies` (`DocumentationPlatformController_getSalesforceElementDependencies`) for what this element needs, and `GET /documentation-platform/salesforce/{metadataId}/dependents` (`DocumentationPlatformController_getSalesforceElementDependents`) for what needs it. **Read dependents before proposing any edit or deletion.**

**Snowflake / Data360 / HubSpot / ServiceNow / Workato** — the same shape under a different path prefix:
- Search: `GET /documentation-platform/snowflake/search` (`DocumentationPlatformController_searchElements`) or by name `GET /documentation-platform/snowflake/search-by-metadata` (`DocumentationPlatformController_findElementByName`).
- Explain: `GET /documentation-platform/snowflake/{metadataId}/summary` (`DocumentationPlatformController_getElementSummary`) and `GET /documentation-platform/snowflake/{metadataId}/source` (`DocumentationPlatformController_getElementSource`).
- Ask: `POST /documentation-platform/snowflake/{metadataId}/chat` (`DocumentationPlatformController_chatWithElement`) — a write method that only costs a chat turn, but it is still a POST; do not fan it out in a loop.

Every operation in this skill is read-only except the `chat` endpoints. Ground any recommendation in the `dependents` response, and quote the `metadataId` you read it from.
