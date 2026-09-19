---
title: "07 — API, data and configuration contracts"
description: "07 — API, data and configuration contracts — imported from 07-api-data-and-configuration.md and published as part of these docs."
status: generated
version: "0.1"
---

# 07 — API, data and configuration contracts

# 07 — API, data and configuration contracts

[Handbook index](README.md)

**All gateway-specific routes, schemas and YAML keys in this chapter are proposed contracts. They are not implemented merely by documenting them.** Preserve existing `/v1/preflight/text` and desktop compatibility while introducing the gateway API.

## Gateway API surface

Use a versioned `/v1/gateways` namespace. Management routes require customer RBAC; node routes require a node identity. Binding to tenant/deployment comes from authentication, not a trusted body parameter.

| Method and path | Caller | Semantics |
|---|---|---|
| `POST /v1/gateways/enrollment-grants` | Deployment admin | Create expiring scoped one-use grant |
| `POST /v1/gateways/enroll` | Bootstrap grant + node proof | Atomically consume grant; create node identity |
| `POST /v1/gateways/{id}/heartbeat` | Same node | Report sequence, versions, capacity and health |
| `GET /v1/gateways/{id}/assignment` | Same node | Signed configuration/policy assignment; ETag supported |
| `GET /v1/policy-bundles/{digest}` | Authorized node/admin | Immutable signed bundle |
| `POST /v1/gateways/{id}/events` | Same node | Bounded batch; durable idempotent ingestion |
| `POST /v1/gateways/{id}/credentials/rotate` | Same node + valid proof | Rotate scoped credential with bounded overlap |
| `POST /v1/gateways/{id}/revoke` | Deployment admin | Revoke node and record operator reason |
| `POST /v1/policies/{id}/simulate` | Policy editor | Evaluate synthetic fixtures; no provider forwarding |
| `POST /v1/policies/{id}/publish` | Security admin | Validate, sign and create immutable version |

Policy publication uses optimistic concurrency (`If-Match` or equivalent revision) so one editor cannot overwrite another silently. Use idempotency keys for administrative mutations with retriable clients. Store key, caller scope, operation and request hash; reject reuse with a different payload.

Responses include a request ID. Standard errors contain machine-readable `code`, safe `message`, `retryable` and field errors. Use 401 for unauthenticated, 403 for unauthorized, 409 for conflict, 413 for size limit, 422 for invalid configuration, 429 for capacity and 503 for required dependency unavailable. Never return raw model output or credentials in errors.

## Event contract

Illustrative metadata-only event:

```json
{
  "schema_version": "1",
  "event_id": "evt_example_unique_id",
  "gateway_id": "gw_example",
  "sequence": 1042,
  "observed_at": "2026-09-19T10:00:00Z",
  "principal_ref": "principal_example",
  "team_ref": "team_finance",
  "destination_id": "approved_ai_service",
  "protocol_adapter": "example-api-v1",
  "policy_version": "policy_42",
  "engine_version": "engine_example",
  "evaluated_action": "MASK",
  "enforced_action": "MASK",
  "coverage": "inspected",
  "findings": [{"category": "BANK_ACCOUNT", "count": 1}],
  "reason_code": "TEAM_RESTRICTED_DATA",
  "inspection_ms": 24
}
```

The example latency is illustrative, not a measured claim. Server attaches authenticated deployment/tenant and ingestion time. Validate bounded strings, enum values, counts and timestamps. Do not include prompt, replacement value, credential headers or full content-bearing URL.

Batch response returns accepted/duplicate/rejected event IDs and permanent error codes. Acknowledgement means committed durable storage, not receipt in memory. Gateway removes only acknowledged records. Retry transient failure with bounded exponential backoff/jitter. Partial failures must not cause already accepted events to be duplicated in downstream reports.

## Data model

Proposed entities extend the existing organization/team model:

| Entity | Key relationships and constraints |
|---|---|
| Deployment | Immutable deployment ID, mode, schema/config version |
| Gateway | Deployment/tenant, unique node identity, lifecycle status, last heartbeat |
| Gateway credential | Node, key fingerprint/hash, validity, revoked time |
| Enrollment grant | Hashed grant, scope, expiry, usage count with atomic consumption |
| Policy version | Tenant/policy/version, content digest, signature, capability requirements |
| Assignment | Gateway/cohort, bundle digest, monotonically increasing generation |
| Decision event | Tenant/gateway/event ID unique; observed and ingested timestamps |
| Outbox job | Event/destination unique, state, attempts, next attempt, lease owner/expiry |
| Integration | Tenant, approved endpoint, encrypted secret reference, enabled state |
| Admin audit | Actor, operation, target, before/after digest, time, reason |
| License state | Provider, entitlement references, last verification, allowed scope |

Index gateway health by deployment/status/time; events by tenant/time and principal/time; jobs by state/next-attempt. Partition event tables only when measured volume and deletion behavior justify it. Scope every query by authenticated tenant; optional row-level security adds defense but does not replace application authorization.

## Worker transaction semantics

Within a short transaction claim eligible jobs using row locks/leases, then commit the claim. Perform remote delivery outside the database lock. On completion, update only if the lease is still owned. Expired leases are reclaimable. A crash after remote acceptance but before local completion can duplicate delivery: provide event IDs and document at-least-once behavior rather than claiming exactly-once SIEM delivery.

Permanent failures enter a visible dead-letter state. Operators can inspect safe metadata and retry after fixing the destination. Retention cleanup must not delete data needed by pending jobs without a deliberate policy and alert.

## Proposed configuration

This example describes a private gateway role. The parser and secret providers must be implemented before use.

```yaml
schema_version: 1
deployment:
  id: customer-assigned-deployment-id
  profile: self_hosted
  role: gateway
network:
  proxy_bind: "0.0.0.0:8443"
  proxy_transport: tls
  client_cidrs: ["10.20.0.0/16"]
  management_bind: "127.0.0.1:9090"
identity:
  mode: trusted_upstream
  trusted_upstream_ca_ref: "file:/etc/zotline/trust/upstream-ca.pem"
  unknown_principal_action: restrictive_policy
control:
  url: "https://zotline-admin.customer.example"
  node_credential_ref: "file:/etc/zotline/secrets/node-credential"
policy:
  cache_path: "/var/lib/zotline/policy"
  verification_keys_path: "/etc/zotline/trust/policy-keys.json"
  invalid_bundle_action: block
inspection:
  mode: deterministic_only
  unsupported_action: block
  max_decompressed_body_bytes: 10485760
  max_inflight_requests: 100
audit:
  mode: metadata
  spool_path: "/var/lib/zotline/spool"
  max_spool_bytes: 10737418240
  full_action: block
updates:
  mode: customer_approved
telemetry:
  vendor_export: false
```

The capacities are sample values needing workload validation. The trusted-upstream example requires an upstream that actually provides authenticated identity; it is not a default for arbitrary laptops. No inference endpoint means policies requiring semantic inspection cannot activate. Separate TLS proxy/inspection certificate references and complete destination ACLs are required before this partial example could form a valid deployment configuration.

## Configuration validation and lifecycle

Reject unknown keys and invalid combinations. Require explicit production authentication, durable storage, trust material and valid policy before readiness. Reject vendor endpoints in strict private mode unless explicitly approved by a customer egress policy. Secret providers must fail safely without printing secret values.

Configuration precedence: packaged defaults → customer configuration → explicitly documented operator overrides. Do not permit arbitrary environment variables to override trust or tenant identity invisibly. Hash effective non-secret configuration and expose its version in health.

Hot reload may cover safe limits, destination catalog and verified policy assignments. Listener addresses, trust hierarchy, database schema and identity authority generally require a staged restart/migration. Validate new configuration before atomically activating it; retain last known good configuration and log the actor/change.

## Health contracts

`/livez` means process/event loop alive. `/readyz` means this role can serve its intended workload; gateway readiness includes valid policy, certificates, usable spool and required engine. `/status` is authenticated and returns component states and safe version information. Readiness checks must not create billable model requests continuously or expose sensitive configuration publicly.

## Compatibility

Every release manifest records application, gateway API major, event schema, policy schema, database migration bounds, engine and model profile compatibility. New optional event fields are additive; breaking changes require a new major schema. Control plane should support an explicitly tested previous gateway version during rolling upgrades. The window is a release decision, not “all old versions forever.”
