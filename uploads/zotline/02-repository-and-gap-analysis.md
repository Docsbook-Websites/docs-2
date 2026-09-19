---
title: "02 — Repository evidence and reuse plan"
description: "02 — Repository evidence and reuse plan — imported from 02-repository-and-gap-analysis.md and published as part of these docs."
status: generated
version: "0.1"
---

# 02 — Repository evidence and reuse plan

# 02 — Repository evidence and reuse plan

[Handbook index](README.md)

## Evidence ledger

The following links point to repository files inspected for this design. The observations describe code structure, not results of executing the system.

| Source | Observed | Consequence |
|---|---|---|
| [Platform doors](../../website/components/ui/PlatformDoors.tsx) | Zotline is the network-layer offering at corporate egress | Preserve separation from Agent and SDK |
| [Deployment onboarding](../../ui/src/app/onboarding/deployment/page.tsx) | Zotline routes to a sales conversation | Build actual appliance onboarding |
| [Go entry point](../../aegis-macos-app/Vendor/agp/cmd/agp/main.go) | Policy bootstrap/cache, proxy and desktop bridge startup | Reuse engine lifecycle; introduce a gateway-specific supervisor |
| [Go proxy](../../aegis-macos-app/Vendor/agp/internal/proxy/server.go) | CONNECT, generated certificates and bypass tunneling | Add enterprise auth, destination ACL and scoped identity |
| [Go configuration](../../aegis-macos-app/Vendor/agp/internal/config/config.go) | Loopback defaults and desktop-related settings | Existing flags are not a hardened network-appliance configuration |
| [Certificate authority](../../aegis-macos-app/Vendor/agp/internal/ca/authority.go) | Local certificate machinery | Add customer intermediate lifecycle and fleet key isolation |
| [Redaction engine](../../aegis-macos-app/Vendor/agp/internal/redact/engine.go) | Multiple engine/runtime paths | Release must pin supported engine and artifact versions |
| [Python detection](../../sdk/zotniq/detection/detector.py) | Separate Python implementation | Establish parity where promised with fixtures |
| [API startup](../../api/app/main.py) | API plus in-process background jobs and hardcoded CORS entries | Externalize configuration and separate worker scheduling |
| [Database layer](../../api/app/db.py) | SQLite/PostgreSQL adaptation; production requires PostgreSQL | Supported production appliance uses PostgreSQL |
| [Agent identity](../../api/app/auth/agent_key.py) | Per-device credentials bound to enrolling user | Gateway identity must not represent all traffic as installer identity |
| [Kinde auth](../../api/app/auth/kinde.py) | Hosted identity coupling | Add portable IdP integration across login and lifecycle operations |
| [Desktop router](../../api/app/routers/desktop_agents.py) | Enrollment, updates and contextual redaction service | Extract reusable services; avoid gateway masquerading as desktop |
| [Team evaluator](../../api/app/services/team_nlp_evaluation.py) | Optional model-assisted rules, default-on toggle | Missing model must produce explicit capability/error state |
| [File abstraction](../../api/app/storage/files.py) | Local disk/file-store operations | Use durable paths and verified customer object-store adapters |
| [Audit worker](../../api/app/storage/audit_stream_worker.py) | Declared single-worker assumption | Add leases and idempotency before HA |
| [URL guard](../../api/app/storage/audit_stream_url_guard.py) | Private SIEM destinations rejected | Replace with controlled customer allowlisting, not unrestricted bypass |
| [ECR Compose](../../deploy/aws/docker-compose.ecr.yml) | nginx/messenger sidecars; API/UI managed separately | Vendor production deployment is not customer installation automation |
| [Database Terraform](../../infra/modules/database/main.tf) | Single-AZ and permissive deletion choices | New production templates need retention and resilience decisions |
| [Main CI](../../.github/workflows/ci.yml) | Placeholder build job | Do not assume repository-level CI proves an appliance release |

## Reuse boundaries

Extract the Go parser/detector/rewriter and policy interfaces into a shared Go module, preserving import boundaries and tests. Both desktop builds and the gateway consume a pinned revision. Keep desktop process capture, OS certificates and window management outside that module.

Add a `zotline-gateway` entry point that owns server configuration, connection identity, health/readiness, fleet enrollment, durable spool and production limits. Do not expose the existing loopback proxy to a network simply by changing its bind address.

Keep the FastAPI application as a modular control plane initially. New gateway routers should call shared policy and audit services rather than copy desktop routers. Introduce identity/storage/inference/license interfaces so appliance mode does not import hosted-only behavior on startup.

The Next.js UI needs runtime deployment discovery or a consistently relative API origin. Configuration baked into `NEXT_PUBLIC_*` during build is unsuitable for arbitrary customer hostnames without a deliberate routing strategy. Serve same-origin management where possible and test auth callbacks/reverse-proxy headers.

## Cross-cutting migration checklist

1. Inventory every outbound dependency: authentication, management APIs, model calls, notifications, email, update downloads, telemetry and catalog refresh.
2. Make deployment mode explicit and validated. Refuse production startup with auth bypass or unresolved mandatory settings.
3. Remove hardcoded vendor hostnames from functional flows in appliance mode. Preserve legacy API compatibility for existing hosted clients.
4. Move migrations to a controlled job. Avoid competing startup migrations when adding replicas.
5. Move scheduled jobs out of every web-worker lifespan. Each durable job must have one claimant or idempotent processing.
6. Replace local temporary storage assumptions with durable, quota-controlled paths where persistence is required.
7. Verify logging and diagnostics along every redaction path, including error branches.
8. Add gateway-specific fleet and coverage UI; do not imply endpoint inventory from network observations.

## Existing behavior that needs explicit decisions

The Go engine has policy caching, but the proposed signed-bundle protocol and revocation rules must be implemented and tested. The API health endpoint confirms process response, not complete service readiness. The private SIEM development bypass is intentionally forbidden in production; a customer-safe configuration path is new work.

`OPENAI_BASE_URL` is a useful existing extension point. It does not prove compatibility with arbitrary local models, output schemas, token limits or error semantics. A model profile needs its own acceptance suite.

The desktop recorder includes evidence-building paths. Audit privacy must be checked at the actual serialization boundary; documentation saying “metadata” does not prove every string is free of customer content.

## Exit from repository exploration

Before implementation, create a dependency inventory and record exact source revision, Go/Python engine versions, supported test commands and CI results. This handbook intentionally avoids claiming tests were run or product coverage was certified during the documentation review.
