---
name: Manage an Edge Delta pipeline configuration via the REST API
description: >-
  Safely read, validate, save, and deploy an Edge Delta telemetry pipeline
  configuration using the Edge Delta REST API, following the
  validate -> save -> deploy loop with rollback.
api: openapi/edge-delta-openapi-original.json
generated: '2026-07-19'
method: generated
operations:
  - GET /v1/orgs/{org_id}/confs
  - GET /v1/orgs/{org_id}/confs/{conf_id}
  - POST /v1/orgs/{org_id}/confs/validate
  - POST /v1/orgs/{org_id}/pipelines/{conf_id}/save
  - POST /v1/orgs/{org_id}/pipelines/{conf_id}/deploy/{version}
  - GET /v1/orgs/{org_id}/pipelines/{conf_id}/history
---

# Manage an Edge Delta pipeline configuration

Operate on Edge Delta telemetry pipeline configurations through the REST API.
Base URL: `https://api.edgedelta.com`.

## Authentication

Every request requires the header `X-ED-API-Token: <token>`. Create a token at
https://docs.edgedelta.com/create-api-token/ and get your `org_id` at
https://docs.edgedelta.com/get-org-id/. All paths below are scoped to that org.

## Steps

1. **List configs** — `GET /v1/orgs/{org_id}/confs` to find the pipeline
   configuration you want to change and note its `conf_id`.
2. **Read the current config** — `GET /v1/orgs/{org_id}/confs/{conf_id}` to
   retrieve the current YAML before editing.
3. **Validate** — `POST /v1/orgs/{org_id}/confs/validate` with the edited YAML.
   This endpoint returns HTTP 200 with `{"valid": false, "reason": ...}` for an
   invalid config (it does NOT return a 4xx), so check the `valid` flag in the body.
4. **Save a new version** — `POST /v1/orgs/{org_id}/pipelines/{conf_id}/save`
   with the validated YAML. Saving does not deploy.
5. **Deploy** — `POST /v1/orgs/{org_id}/pipelines/{conf_id}/deploy/{version}` to
   roll the saved version out to the fleet.
6. **Roll back if needed** — inspect
   `GET /v1/orgs/{org_id}/pipelines/{conf_id}/history` and re-deploy an earlier
   version.

## Rules

- Follow the order validate -> save -> deploy. Never deploy a config that has
  not passed validation.
- Writes are not idempotent — do not blindly retry a save/deploy on a network
  error; re-read state first (see conventions/edge-delta-conventions.yml).
- Handle errors per errors/edge-delta-problem-types.yml: 400 malformed body,
  401 bad/missing token, 404 unknown org_id or conf_id.
