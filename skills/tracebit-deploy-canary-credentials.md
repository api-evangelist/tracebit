---
name: Deploy Tracebit canary credentials
description: Issue decoy (canary) credentials via the Tracebit Community API, deploy them, and confirm the deployment so Tracebit starts monitoring them.
api: openapi/tracebit-community-openapi-original.json
operations: [IssueCredentials, ConfirmCredentials]
generated: '2026-07-21'
method: generated
source: openapi/tracebit-community-openapi-original.json
---

# Deploy Tracebit canary credentials

Ground rules (from `conventions/tracebit-conventions.yml`):
- Authenticate every call with a Tracebit API token as an HTTP bearer token (`Authorization: Bearer <token>`). The token needs the `canary-credentials:all:create` permission.
- There is no idempotency-key header; safety comes from the two-phase issue/confirm flow. Never re-use a confirmationId.
- 4xx errors return plain-text bodies (see `errors/tracebit-problem-types.yml`).

## Steps

1. **Issue credentials** — `POST /api/v1/credentials/issue-credentials` (operationId `IssueCredentials`) with a JSON body: required `name` (identifier for the credentials), `source` (system deploying them, e.g. `custom` or `script`), `sourceType` (e.g. `endpoint`, `container`, `ci/cd`); optional `types` array from `aws`, `ssh`, `gitlab-cookie`, `gitlab-username-password`, and optional `labels[]` name/value metadata.
2. **Deploy the returned decoys** — the 200 response contains per-type credential material (`aws` with `awsAccessKeyId`/`awsSecretAccessKey`/`awsSessionToken`, `ssh` with base64 keys and server IP, `http` with perimeter credentials). Place them where a real credential of that type would live. Each block carries its own `confirmationId` and an expiration timestamp.
3. **Confirm deployment** — `POST /api/v1/credentials/confirm-credentials` (operationId `ConfirmCredentials`) with `{"id": "<confirmationId>"}` for each issued credential set. A 204 confirms monitoring is active; a 404 means the confirmationId is unknown — re-issue rather than retry blindly.
