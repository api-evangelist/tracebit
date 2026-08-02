---
name: Monitor Tracebit canary alerts
description: Poll canary alerts from the Tracebit Community API and pull the cursor-paginated logs for any alert that fires.
api: openapi/tracebit-community-openapi-original.json
operations: [ListAlerts, GetAlertLogs]
generated: '2026-07-21'
method: generated
source: openapi/tracebit-community-openapi-original.json
---

# Monitor Tracebit canary alerts

Ground rules (from `conventions/tracebit-conventions.yml`):
- Authenticate with a bearer API token holding `alerts:all:list` (list) and `alerts:all:get` (logs) permissions.
- `ListAlerts` returns at most 1000 alerts; `GetAlertLogs` is cursor-paginated.
- 4xx errors are plain text; a 404 from `GetAlertLogs` means the alert id (UUID) does not exist.

## Steps

1. **List alerts** — `GET /api/v1/alerts` (operationId `ListAlerts`), optionally `?order=asc|desc` (default `desc`, newest first). Each alert carries `severity` (Info/Medium/High), `classification` (Unclassified/TruePositive/BenignPositive/FalsePositive), the triggering `provider`/`provider_account_id`, `indicators[]`, and a `tracebit_portal_url` for humans.
2. **Triage** — treat `TruePositive` or high-severity unclassified alerts as potential intrusions; surface `title`, `start_time`, and indicators to the operator.
3. **Fetch logs** — `GET /api/v1/alerts/{id}/logs` (operationId `GetAlertLogs`) for the alert's UUID. Page backwards with the `beforeTime` (date-time) and `beforeLog` (UUID) query params; follow the `next_page` URL in the response until it is null.
4. **Correlate** — each log line links the touched `canary`, the decoy `canary_credential`, the acting `principal` (AWS/Okta/Azure/Google Cloud detail), and the provider `event` (operation, request IP/user-agent, outcome).
