---
name: Audit automations before deploying one
description: Enumerate an org's Sweep automations, alerts and assignment rules, read version history and
  execution order, and understand which operations actually push change into Salesforce.
api: openapi/sweep-api-openapi.yml
generated: '2026-08-14'
method: generated
source: openapi/sweep-api-openapi.yml (harvested from https://api.sweep.io/api-json) + conventions/sweep-conventions.yml
  + errors/sweep-problem-types.yml
operations:
- AutomationController_getAutomations
- AutomationController_getAutomationById
- AutomationController_getVersionHistory
- AutomationController_listAutomationsExecutionOrder
- AutomationController_listAlertsExecutionOrder
- AutomationController_listAssignmentsExecutionOrder
- AutomationController_createReadOnlyAutomation
- AutomationController_deployAutomation
- AutomationController_toggleActivation
---

# Audit automations before deploying one

Enumerate an org's Sweep automations, alerts and assignment rules, read version history and execution order, and understand which operations actually push change into Salesforce.

## Before you start

- Base URL: `https://api.sweep.io`
- Auth: `Authorization: Bearer <JWT>` on every operation below. There is no API-key header and no OAuth flow on the REST API; a 401 comes back as `{"message":"Unauthorized","statusCode":401}` with no `WWW-Authenticate` hint.
- Content type: `application/json` in both directions.
- The contract is at `https://api.sweep.io/api-json` (Swagger UI at `https://api.sweep.io/api`). It declares **no** 4xx or 5xx responses, so treat every error as undocumented and read the status code, not the schema.
- **There is no idempotency contract.** Do not blind-retry a POST/PUT/PATCH/DELETE — a repeat may create or deploy a second object. Re-read state with the matching GET before retrying.
- **There are no rate-limit headers.** Nothing tells you how close you are to a limit; back off on your own schedule.
- Pagination is inconsistent: some list operations take `limit`/`offset`, most return an unbounded array with no total. Never assume a page envelope.

## Steps

1. **Enumerate** — `GET /automations` (`AutomationController_getAutomations`), then `GET /automations/{automationId}` (`AutomationController_getAutomationById`) for the one you care about.
2. **History** — `GET /automations/{automationId}/history` (`AutomationController_getVersionHistory`). Automations are versioned; edit and deploy operations both take `{automationId}/{versionId}`, so always name the version explicitly.
3. **Execution order** — `GET /automations/order/Default` (`AutomationController_listAutomationsExecutionOrder`), `GET /automations/order/Alert` (`AutomationController_listAlertsExecutionOrder`), `GET /automations/order/Assignment` (`AutomationController_listAssignmentsExecutionOrder`). Order changes behaviour; read it before you reason about a conflict.
4. **Draft safely** — `POST /automations/read-only` (`AutomationController_createReadOnlyAutomation`) creates an automation that observes without acting. Prefer this when an agent is proposing rather than shipping.

**The line an agent should not cross unattended.**
- `POST /automations/deployment/{automationId}/{versionId}` (`AutomationController_deployAutomation`) pushes the automation into the connected Salesforce org. It is the operation that changes a production system.
- `PATCH /automations/activation/{automationId}/{versionId}` (`AutomationController_toggleActivation`) turns it on or off.
Both are unsafe methods with **no idempotency key**. Require an explicit human confirmation for each, one at a time, and re-read `GET /automations/{automationId}` afterwards instead of retrying on a timeout. The same pattern repeats for alerts, assignments, dedupe-matching, marketing-attributions and booking — each has its own `/deployment/` and `/activation/` operation.
