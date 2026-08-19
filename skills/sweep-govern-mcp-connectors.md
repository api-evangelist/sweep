---
name: Govern outbound MCP connectors and tool policies
description: List, register and remove the third-party MCP servers a Sweep account connects to, run their
  OAuth, and read or set per-tool policies — the governance surface for what Sweep's agents may call.
api: openapi/sweep-api-openapi.yml
generated: '2026-08-14'
method: generated
source: openapi/sweep-api-openapi.yml (harvested from https://api.sweep.io/api-json) + conventions/sweep-conventions.yml
  + errors/sweep-problem-types.yml
operations:
- McpConnectorsController_getConnectors
- McpConnectorsController_createConnector
- McpConnectorsController_getOAuthAuthorizationUrl
- McpConnectorsController_handleOAuthCallback
- McpConnectorsController_getConnectionTools
- McpConnectorsController_updateConnectionToolPolicies
- McpConnectorsController_deleteConnection
- McpConnectorsController_deleteConnector
- AgenticFeaturesController_getAgenticFeatureSettings
- AgenticFeaturesController_getDataAccessSettings
- AgenticFeaturesController_updateDataAccessSettings
- MCPController_getTemporaryApikey[0]
---

# Govern outbound MCP connectors and tool policies

List, register and remove the third-party MCP servers a Sweep account connects to, run their OAuth, and read or set per-tool policies — the governance surface for what Sweep's agents may call.

## Before you start

- Base URL: `https://api.sweep.io`
- Auth: `Authorization: Bearer <JWT>` on every operation below. There is no API-key header and no OAuth flow on the REST API; a 401 comes back as `{"message":"Unauthorized","statusCode":401}` with no `WWW-Authenticate` hint.
- Content type: `application/json` in both directions.
- The contract is at `https://api.sweep.io/api-json` (Swagger UI at `https://api.sweep.io/api`). It declares **no** 4xx or 5xx responses, so treat every error as undocumented and read the status code, not the schema.
- **There is no idempotency contract.** Do not blind-retry a POST/PUT/PATCH/DELETE — a repeat may create or deploy a second object. Re-read state with the matching GET before retrying.
- **There are no rate-limit headers.** Nothing tells you how close you are to a limit; back off on your own schedule.
- Pagination is inconsistent: some list operations take `limit`/`offset`, most return an unbounded array with no total. Never assume a page envelope.

## Steps

Sweep is on both sides of MCP: it publishes a hosted MCP server at `https://sweepmcp.com/sse`, and its REST API lets an account
**consume** other MCP servers. This skill covers the consuming side, which is the governance surface.

1. **What is connected** — `GET /mcp-connectors` (`McpConnectorsController_getConnectors`).
2. **Register a server** — `POST /mcp-connectors` (`McpConnectorsController_createConnector`).
3. **Authorize it** — `GET /mcp-connectors/{serverId}/oauth/url` (`McpConnectorsController_getOAuthAuthorizationUrl`), hand the URL to a human, then `POST /mcp-connectors/{serverId}/oauth/callback` (`McpConnectorsController_handleOAuthCallback`).
4. **Read the tool policy** — `GET /mcp-connectors/connections/{connectionId}/tool-policies` (`McpConnectorsController_getConnectionTools`). This is the allow-list of what the connected server's tools may do.
5. **Change the tool policy** — `PUT /mcp-connectors/connections/{connectionId}` (`McpConnectorsController_updateConnectionToolPolicies`). Treat this as a privileged change: it widens or narrows what agents can execute. Read step 4 first and diff.
6. **Revoke** — `DELETE /mcp-connectors/connections/{connectionId}` (`McpConnectorsController_deleteConnection`) or `DELETE /mcp-connectors/{serverId}` (`McpConnectorsController_deleteConnector`).

**Account-level agent settings** — `GET /agentic-features` (`AgenticFeaturesController_getAgenticFeatureSettings`) and
`GET /agentic-features/data-access-settings` (`AgenticFeaturesController_getDataAccessSettings`) tell you what agents are allowed to
reach at all. `PATCH /agentic-features/data-access-settings` (`AgenticFeaturesController_updateDataAccessSettings`) changes it —
an escalation of agent data access, and never something an agent should do to itself without a human in the loop.

**Connecting to Sweep's own MCP server** is a separate thing: point an MCP client at `https://sweepmcp.com/sse` and complete the
OAuth flow advertised at `https://sweepmcp.com/.well-known/oauth-authorization-server`. `GET /mcp/apikey`
(`MCPController_getTemporaryApikey[0]`) issues a temporary key from the REST side; treat anything it returns as a secret and never
log it.
