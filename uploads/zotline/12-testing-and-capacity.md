---
title: "12 — Testing, sizing and release acceptance"
description: "12 — Testing, sizing and release acceptance — imported from 12-testing-and-capacity.md and published as part of these docs."
status: generated
version: "0.1"
---

# 12 — Testing, sizing and release acceptance

# 12 — Testing, sizing and release acceptance

[Handbook index](README.md)

## Evidence, not architecture promises

A topology diagram cannot prove prevention, privacy or resilience. Qualify each advertised release/profile against the matrix below. Record source revision, artifact digests, configuration, environment, date and result. Use synthetic sensitive data. Retain failed cases as regressions.

## Test layers

| Layer | Coverage |
|---|---|
| Unit | Parsers, offsets, validators, policy precedence, identity resolution, signature checks |
| Shared conformance | Same fixtures through Go/Python surfaces where parity is promised |
| Protocol | Real proxy/TLS path to controlled upstream, HTTP versions, streams, adapters |
| Security | Spoofing, open-proxy/SSRF, tenant isolation, replay, secret exclusion |
| Integration | IdP, database, policy assignment, inference, spool and SIEM |
| Appliance | Clean install, interruption/retry, backup, upgrade, uninstall |
| Infrastructure | Fresh buyer account, subnet/DNS/IAM, AZ loss and data retention |
| Commercial | Entitlement, node replacement, outage and metering reconciliation |

## Mandatory acceptance matrix

| ID | Scenario | Required result | Requirements |
|---|---|---|---|
| T-01 | Clean supported request | Upstream receives expected content; correct metadata | FR-01, FR-03 |
| T-02 | Supported secret with BLOCK | No original body reaches upstream | FR-04 |
| T-03 | Supported PII with MASK | Upstream receives transformation; response remains valid | FR-05 |
| T-04 | Two users share NAT/proxy | Correct distinct policy identities | FR-02 |
| T-05 | Client injects team/identity header | Header cannot elevate privilege | FR-02 |
| T-06 | Malformed/compressed/oversize input | Bounded resource use and declared protected behavior | FR-01, FR-10 |
| T-07 | Spoofed placeholder/cross-user conversation | No other user's original data restored | FR-05, FR-06 |
| T-08 | Upstream TLS invalid | Connection rejected; no verification bypass | FR-06 |
| T-09 | Metadata/management/private target abuse | Unauthorized target denied | FR-06 |
| T-10 | Policy signature invalid/expired/replayed | No unsafe policy activation | FR-03 |
| T-11 | Management down, valid cached policy | Protection continues; degraded status visible | FR-10 |
| T-12 | Model timeout/invalid output | No clean-result fallback for required semantic rules | FR-03, FR-10 |
| T-13 | Crash before/after audit ACK | Safe replay and deduplication; loss bound documented | FR-07 |
| T-14 | SIEM down and spool full | Bounded storage, alert, configured enforcement behavior | FR-07, FR-10 |
| T-15 | Internet denied in strict private mode | Supported local workflows function without hidden vendor calls | FR-06 |
| T-16 | Fresh image launched twice | Different keys/identities; no customer/test secrets | FR-08 |
| T-17 | Install interrupted at every checkpoint | Resume without destructive reinitialization | FR-08 |
| T-18 | Previous supported version upgrade | Data/policy preserved; compatibility validated | FR-09 |
| T-19 | Isolated restore | Measured recovery and decryption; no duplicate external effects | FR-09 |
| T-20 | Gateway/AZ loss during streams | Documented failures; new traffic on surviving capacity | FR-10 |
| T-21 | License endpoint outage/expiry | Correct distinct states, no hidden bypass/double billing | FR-11 |
| T-22 | Uninstall after active use | Safe routing/trust removal and deliberate data retention | FR-12 |
| T-23 | IPv6/QUIC/pinning/bypass paths | Each protected, blocked or visibly excluded as documented | FR-01, FR-10 |
| T-24 | Monitor-only mode | Clearly distinguishes evaluated from enforced action | FR-03, FR-10 |
| T-25 | Inspect logs/export/support bundle | No synthetic secret/cookie/key leakage | FR-06, FR-07 |

For T-02/T-03, inspect the controlled upstream's actual received bytes, not only the Zotline decision response. For privacy tests, seed unique canary strings and search every configured persistence/export destination, including exception logs and model diagnostics.

## Client support matrix template

Before GA, replace TBD entries with named versions and evidence:

| Client and version | OS | Proxy auth | Trust configuration | Protocol/adapter | Upload/stream support | Result |
|---|---|---|---|---|---|---|
| Browser + selected AI web app: TBD | TBD | TBD | TBD | TBD | TBD | Not certified |
| Coding assistant: TBD | TBD | TBD | TBD | TBD | TBD | Not certified |
| AI API client: TBD | TBD | TBD | TBD | TBD | TBD | Not certified |
| Internal model client: TBD | TBD | TBD | TBD | TBD | TBD | Not certified |

Do not promote a working demo to broad compatibility. Provider payloads evolve; maintain synthetic adapter fixtures and a controlled compatibility check process. Separate “connects through proxy” from “content inspection and rewriting are certified.”

## Performance workload model

Collect expected users, peak requests/second, concurrency, median/p95/max body size, upload distribution, TLS handshake rate, response streaming duration, semantic-rule fraction, retention and acceptable latency. Separate burst rate from daily average.

Measure deterministic overhead separately from model queueing/inference and provider response time. Report p50/p95/p99 under sustained and burst load, with error/coverage rate. Throughput claims are valid only for the tested payload, rules, model and hardware profile.

Approximate steady-state inspected concurrency as arrival rate × inspection duration. This relationship is a starting estimate; burst queues, long streams and TLS state also consume resources. Estimate in-flight body memory using concurrent buffered requests × bounded decompressed size plus parser/rewriter overhead. A 10 MiB cap and 100 simultaneously buffered bodies already imply roughly 1 GiB of body storage before overhead.

## Worked audit sizing example

Assume 1,000,000 events/day at 1,500 bytes/event and 30-day retention. Raw event payload is 45,000,000,000 bytes, about 45 GB decimal. This excludes indexes, row overhead, WAL, replicas, archives and backups. Measure those factors with the chosen schema rather than assigning an unexplained universal multiplier.

At a peak of 100 events/second and an 8-hour ingestion outage, raw queued events occupy 4.32 GB decimal. Add record/encryption/filesystem overhead and headroom; also test replay throughput while normal traffic continues. These are illustrative inputs, not capacity guarantees for Zotline.

## Initial lab sizing

A single-node deterministic test environment can start at 4 vCPU, 16 GiB RAM and 100 GiB durable storage. This is not a supported minimum until measured. Semantic inference hardware depends on model, precision, context, batching and concurrency; size it separately. Reserve capacity for one-node failure in HA profiles rather than using 100% of aggregate capacity during normal operation.

## Accuracy evaluation

Use labeled synthetic/publicly authorized fixtures covering true positives, benign lookalikes, multilingual text, long contexts, escaped/encoded strings and adversarial instructions. Report per-category false-positive and false-negative rates with sample sizes and known gaps. Natural-language policy quality and regex detection quality are different measurements.

Any model/detector change can change enforcement. Treat it as a versioned release with regression analysis and customer canary, not a transparent dependency patch.

## Release evidence package

Include completed requirement-to-test mapping, support matrix, security scan findings/disposition, privacy egress capture, protocol artifacts, upgrade/restore timings, regional installation results and commercial-flow results. Security and product owners sign off remaining limitations. No critical unmitigated prevention/isolation failure can be hidden as an operational note.
