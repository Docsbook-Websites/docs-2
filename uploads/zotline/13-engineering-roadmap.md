---
title: "13 — Implementation backlog and architecture decisions"
description: "13 — Implementation backlog and architecture decisions — imported from 13-engineering-roadmap.md and published as part of these docs."
status: generated
version: "0.1"
---

# 13 — Implementation backlog and architecture decisions

# 13 — Implementation backlog and architecture decisions

[Handbook index](README.md)

## Delivery approach

Build a narrow end-to-end vertical slice before expanding compatibility: one authenticated client, one supported provider API, deterministic masking/blocking, customer-local control plane and durable metadata audit. Then harden and package the same path. This provides real evidence for the network, identity and privacy contracts.

The phases below are dependency gates, not calendar estimates. Estimate after a runnable Linux gateway spike and an agreed client/inference support matrix.

## Proposed source layout

```text
gateway/
  cmd/zotline-gateway/       gateway entry point
  internal/identity/        authenticated caller context
  internal/fleet/           enrollment, assignments, health
  internal/spool/           durable metadata delivery
  internal/config/          schema validation and reload
shared/go/                  extracted inspection/policy interfaces
api/app/routers/gateways.py  proposed gateway control API
api/app/services/            shared policy, identity, license adapters
deploy/appliance/            service definitions and lifecycle tooling
infra/marketplace/           image build and CloudFormation
tests/zotline/               conformance, install and failure fixtures
docs/zotline/                this handbook
```

These paths describe planned code, except the existing API/services tree and this handbook. They have not been scaffolded by the documentation task.

## Backlog

| ID | Work package | Owner role | Dependencies | Acceptance |
|---|---|---|---|---|
| ZL-01 | Confirm first client/protocol and commercial scope | Product + gateway lead | None | Signed support/scope matrix |
| ZL-02 | Extract shared Go engine without desktop regression | Gateway | ZL-01 | Desktop/gateway conformance fixtures |
| ZL-03 | Gateway lifecycle/config/readiness/limits | Gateway | ZL-02 | Restart, malformed config and overload tests |
| ZL-04 | Proxy identity and destination security | Gateway + security | ZL-03 | T-04/05/08/09 |
| ZL-05 | Customer CA issuance/rotation integration | Security + gateway | ZL-03 | Trust/rotation/revocation drill |
| ZL-06 | Gateway fleet and scoped credentials | Backend | ZL-03 | Enrollment replay and rotation tests |
| ZL-07 | Portable console IdP and user lifecycle | Backend/UI | ZL-01 | Login/RBAC/deactivation/recovery |
| ZL-08 | Signed policy compiler/assignment and simulation | Backend + gateway | ZL-04/06 | T-10/11 and precedence fixtures |
| ZL-09 | Private inference profile | Detection/ML | ZL-01/03 | Semantic quality, schema and timeout tests |
| ZL-10 | Durable event API/spool/outbox | Gateway + backend | ZL-06 | T-13/14/25 |
| ZL-11 | Private storage/SIEM/notification adapters | Backend | ZL-07/10 | Egress-denied privacy suite |
| ZL-12 | Customer setup and coverage UI | UI + backend | ZL-06/07/08/10 | New admin completes pilot unaided |
| ZL-13 | Backup/restore/update/migration tooling | Infrastructure + backend | ZL-10/11 | T-18/19 |
| ZL-14 | Signed single-node/offline appliance | Release | ZL-12/13 | T-15/16/17/22 |
| ZL-15 | AWS HA templates and recovery | Infrastructure | ZL-13/14 | Fresh-account and AZ-loss evidence |
| ZL-16 | Licensing adapter and offer definition | Backend + commercial | ZL-01/14 | T-21 and accounting reconciliation |
| ZL-17 | Marketplace scan/limited buyer validation | Release | ZL-15/16 | Independent buyer acceptance |
| ZL-18 | Transparent/SSE integrations | Network + gateway | Stable GA explicit proxy | Separate integration conformance |

## Milestone gates

**M1: developer vertical slice.** Synthetic data is blocked/masked before a controlled upstream. Identity and policy are explicit. No production readiness claim.

**M2: private pilot.** Customer IdP, trust, local inference where needed, event durability, failure semantics and setup are implemented. No undocumented vendor dependencies.

**M3: supported appliance.** Install/upgrade/restore/uninstall are automated and tested; security review and supported-client matrix complete.

**M4: Marketplace production.** Clean buyer-account installation, offer/license validation, regional tests, security scan and listing review complete.

**M5: expanded networking.** Transparent routing or vendor connectors become separate tested options, with accurate coverage claims.

## Architecture decision register

| ADR | Baseline decision | Rationale | Revisit when |
|---|---|---|---|
| 001 | Explicit proxy first | Closest to reusable Go code and testable identity/TLS boundaries | First customer cannot route required clients |
| 002 | Shared Go engine; separate gateway command | Prevent divergent desktop/network protection | Shared module proves impractical after spike |
| 003 | Customer-local control plane | Matches private/self-hosted product requirement | Customer explicitly selects hosted management |
| 004 | Modular API plus separate workers | Avoid premature service proliferation while fixing concurrency | Measured scaling/isolation needs differ |
| 005 | PostgreSQL outbox initially | Durable jobs without another cluster | Backlog/throughput measurements justify broker |
| 006 | Metadata-only default | Minimize sensitive persistent data | Customer explicitly needs governed payload capture |
| 007 | Signed policy assignments | Authenticity, rollback control and offline validity | Cryptographic review changes format |
| 008 | No silent semantic downgrade | Missing model cannot satisfy a semantic rule | Customer approves explicit reduced policy |
| 009 | No cross-request restoration by default | Avoid premature shared sensitive state | Named adapter requires it and isolation is tested |
| 010 | x86-64 Linux initial packaging | Limit qualification scope | Demonstrated ARM/customer platform demand |
| 011 | Customer-approved updates | Respect network availability and change management | Contracted managed-update service is added |

## Open decisions before build completion

| Decision | Needed by | Owner | Default until resolved |
|---|---|---|---|
| Initial named clients and upload formats | M1 | Product + gateway | Minimal explicit matrix, no broad claims |
| User identity mechanism for each client | M1 | Security + customer IT | Restrictive unknown identity |
| Linux name detector and semantic model | M2 | Detection/ML | Only validated capabilities enabled |
| Certificate custody/signing integration | M2 | Security + customer PKI | Customer-generated local intermediate with protected keys |
| Offline policy/identity validity window | M2 | Security + product | Must be explicit before deployment |
| Retention, audit loss bound, RPO/RTO | M2 | Customer + operations | No contractual guarantees assumed |
| Billing unit and HA paid-node treatment | M3 | Commercial + engineering | No in-app Marketplace billing behavior assumed |
| Supported OS/hypervisor/DB version bounds | M3 | Release | Qualification required before publishing |

## Risk register

Highest risks are client incompatibility with TLS interception, incorrect user attribution, provider protocol changes, semantic-model latency/quality, cross-user restoration leakage, audit loss during node replacement, and complex first-boot networking. Each has named acceptance tests and an owner above. A risk becomes a release limitation or blocker, not an unsupported marketing promise.

## Definition of done

Code, fixtures, implementation documentation and compatibility manifest agree. Fresh installation succeeds using public/customer documentation only. Threat-model findings are resolved or explicitly accepted. Release artifacts are signed and reproducible from recorded inputs. Support has tested recovery procedures. Product claims match the tested support matrix. No sample schema/command is presented as executable unless its implementation exists and has passed acceptance.
