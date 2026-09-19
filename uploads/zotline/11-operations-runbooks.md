---
title: "11 — Operations, upgrades, recovery and troubleshooting"
description: "11 — Operations, upgrades, recovery and troubleshooting — imported from 11-operations-runbooks.md and published as part of these docs."
status: generated
version: "0.1"
---

# 11 — Operations, upgrades, recovery and troubleshooting

# 11 — Operations, upgrades, recovery and troubleshooting

[Handbook index](README.md)

All procedures describe required release behavior. Exact service names and executable tooling must be supplied and tested by implementation. Use customer-controlled access and synthetic requests for checks; never paste production credentials into diagnostic logs.

## Operational ownership and cadence

| Cadence | Owner | Work |
|---|---|---|
| Continuous | Automated monitoring | Protection readiness, traffic errors, spool pressure, identity and policy health |
| Daily | Customer operations | Failed backups, delivery backlog, unready/stale nodes and inference saturation |
| Weekly | Security administrator | Exceptions, unsupported coverage, policy drift and unusual decision changes |
| Each release | Release/operations | Stage, canary, compatibility, backup and rollback rehearsal |
| Scheduled drill | Customer operations | Isolated restore, node/AZ loss, certificate and IdP recovery |
| Before expiry | Trust owner | Certificates, licenses, credentials and signing-key transitions |

## Telemetry contract

Collect connection count, new TLS sessions, active inspected requests, decision latency, parser errors, actions by coverage/mode, policy assignment lag, model queue depth, spool bytes/age, ingestion/export lag, database errors and backup age. Keep cardinality bounded: no raw URLs, prompt strings, user emails or request IDs as metric labels.

Suggested initial alerts are engineering defaults: any fleet-wide traffic-unready state is critical; oldest unshipped event age approaching the agreed audit objective is urgent; spool 70% is warning and 90% urgent; certificate/credential expiry alerts at 30/14/7 days. Tune thresholds to actual capacity and support response times. Monitor missing telemetry as well as bad values.

## Runbook: blocked traffic or gateway outage

**Trigger:** clients cannot complete supported AI requests or every node is unready.

1. Identify affected routes, clients and time window using safe request references.
2. Check load-balancer reachability, DNS and per-gateway traffic readiness.
3. Inspect reason codes: invalid policy, expired certificate, audit-full, model-required-unavailable or resource overload.
4. Verify a synthetic allowed and blocked request on a healthy node.
5. Remove only faulty nodes from new traffic; retain capacity on healthy nodes.
6. Replace from the last approved compatible image/configuration if necessary.
7. Verify upstream TLS and policy version before rejoining.

**Do not:** disable authentication, trust verification or enforcement as an undocumented workaround. If the customer authorizes an emergency bypass, record its exact scope, owner, expiry and loss of coverage, then remove it after recovery.

## Runbook: management unavailable

Gateways should keep using valid cached policies. Check policy validity window and warn before expiry. Recover database connectivity, migrations and API readiness before resuming publication. Do not restart all gateways while fixing the dashboard. After recovery compare assigned versus loaded versions and detect missed revocations/configuration changes.

## Runbook: inference unavailable

Check model readiness, queue depth, private network, credentials and artifact compatibility. Distinguish slow queueing from invalid output. Required semantic rules block according to policy. Reducing to deterministic-only inspection needs an authorized policy change that shows lost capability. After recovery run semantic positive/negative fixtures before clearing the incident.

## Runbook: audit backlog or full spool

Check ingestion endpoint, node credentials, database capacity, worker leases and destination response codes. Determine whether events are failing to leave a gateway or failing to leave the control plane. Fix the specific stage, then watch oldest-event age decline.

Expand capacity only after confirming the expected replay rate and available disk. Do not delete spool files to suppress an alert. At full capacity apply configured protected-traffic behavior. Replay with original event IDs so downstream counts do not inflate. Export a dead-letter report with safe metadata and track unresolved losses explicitly.

## Backup specification

Back up PostgreSQL plus WAL/PITR where supported, configuration revisions, required customer key material or recoverable key references, policy signing trust, retained artifacts and release manifests. Record encryption key dependencies and customer recovery owners. Backup location must not share the sole failure domain of the source.

Database backups and object snapshots may represent different points in time. Define an authoritative database restore point and tolerate/reconcile orphaned objects; never assume two independent snapshots are transactionally consistent. Spool recovery and conversation mappings have separate semantics and must be documented.

Backup schedule and retention derive from the agreed RPO, data retention and legal hold requirements. Example initial HA design targets are 15-minute database RPO and 60-minute management RTO; neither is a guarantee until drills demonstrate it under the customer's topology.

## Restore procedure

1. Declare recovery scope and select a verified backup and compatible software release.
2. Provision an isolated network so restored workers cannot send duplicate production notifications/exports.
3. Restore secrets/key access, database and retained objects according to recorded dependencies.
4. Start with migrations disabled until schema/version compatibility is confirmed.
5. Validate tenant counts, policy hashes, identities, audit ranges and decryption.
6. Reconcile outbox leases and duplicate-delivery risk; keep original event IDs.
7. Run synthetic traffic, policy and authorization checks.
8. Authorize controlled endpoint cutover, then re-enable integrations.
9. Record actual lost interval and recovery duration, including data not recovered from destroyed node spools.

Never run restored and original management systems as independent writers to the same logical deployment without a supported split-brain design.

## Upgrade procedure

Before the window: verify release signatures, compatibility manifest, license compatibility, disk space, backups and recovery access. Test the release on synthetic traffic with customer policies.

Apply additive schema migration once with a lock. Upgrade control components in the tested order, retaining compatibility with the previous gateway version. Upgrade a canary gateway, route a small approved cohort and compare coverage, decision behavior and latency. Drain and replace remaining nodes gradually while preserving failure capacity. Record final versions and close the change only after backlog/health stabilizes.

For destructive schema changes, use a later contract phase after old code is removed. Reverting a container image does not undo an incompatible migration. If rollback is unsupported, the release must state the forward-fix or restore plan before installation.

## Certificate and identity rotation

Prepare new trust, distribute it, confirm acceptance, switch issuance, then remove old trust after the documented overlap. For gateway identity rotation, verify new credentials before retiring old ones and never extend overlap indefinitely. An expired or compromised inspection issuer is a protection incident, not just a dashboard warning.

## Troubleshooting matrix

| Symptom | Likely checks | Safe next action |
|---|---|---|
| Browser certificate warning | Trust distribution, hostname, intermediate, clock | Validate chain on pilot; repair trust |
| App works outside proxy only | Pinning, private trust store, unsupported protocol | Use documented adapter/exception path |
| Wrong team rule | Principal mapping, membership cache, policy generation | Correct identity mapping and republish |
| UI works but proxy fails | Separate listener/routes, node readiness | Test traffic endpoint independently |
| Requests slow | Model queue, payload size, TLS churn, capacity | Apply limits/scale after measurements |
| SIEM test rejects internal host | Customer allowlist and certificate chain | Configure supported private destination |
| Duplicate events | Lease/retry path and receiver deduplication | Preserve IDs, fix claims/consumer behavior |
| Missing events after replacement | Unreplayed node volume, ingestion ACK semantics | Recover spool if available; record loss |
| Offline UI makes external calls | Hosted assets, auth, telemetry or update defaults | Remove dependency; rerun denied-egress suite |
| Rollback fails | Schema bounds or key-format change | Use documented compatible restore/forward fix |

## Support and evidence

Diagnostic export contains version manifest, sanitized effective configuration, health transitions, bounded operational logs and event identifiers. Exclude credentials, private keys, prompt bodies, original values and model input/output. Let the customer inspect before export. Support access, if enabled, is time-limited, scoped and auditable, with a customer-controlled removal step.
