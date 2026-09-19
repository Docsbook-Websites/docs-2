---
title: "14 — Checklists, glossary and sources"
description: "14 — Checklists, glossary and sources — imported from 14-reference-and-checklists.md and published as part of these docs."
status: generated
version: "0.1"
---

# 14 — Checklists, glossary and sources

# 14 — Checklists, glossary and sources

[Handbook index](README.md)

## Customer discovery worksheet

Record answers before sizing or offering a delivery date:

- Deployment environment, countries/regions and disconnected-operation requirement.
- Named AI tools/client versions, operating systems, API usage and uploads.
- Number of users/services, peak concurrency and burst request volumes.
- Current proxy/SSE/firewall, remote-access path, IPv6 and direct-egress controls.
- Identity source and how a gateway can authenticate each client type.
- Customer PKI and device/application trust-distribution method.
- Required deterministic/semantic categories, languages and false-positive tolerance.
- Whether raw content may reach customer-local or external inference.
- Retention, SIEM, data residency, backup and recovery objectives.
- Operating team, maintenance window and incident response contacts.
- Procurement model, billable capacity definition and support expectations.

## Before pilot

- [ ] Required client routes and supported adapters identified.
- [ ] Inspection authority and trust distribution approved by customer.
- [ ] Customer-local identity, storage and inference dependencies configured.
- [ ] Mandatory policies and failure behavior reviewed.
- [ ] Protected/uninspected/monitor-only status visible.
- [ ] Synthetic upstream proves block and masking behavior.
- [ ] Direct bypass paths tested and controlled or documented.
- [ ] Metadata exports checked for synthetic secret leakage.
- [ ] Backup and recovery owners assigned.

## Before production

- [ ] Chapter 12 tests completed for the exact release/profile.
- [ ] Capacity measured with failure headroom and realistic payloads.
- [ ] Unknown identity, expired policy, model failure and full spool tested.
- [ ] Gateway and database failure drills completed.
- [ ] Upgrade and isolated restore demonstrated.
- [ ] Certificate/key rotation and recovery demonstrated.
- [ ] Required support matrix and limitations accepted.
- [ ] Alert destinations and escalation owners verified.
- [ ] No vendor credentials, customer data or shared private CA in distribution.
- [ ] Customer can operate without unapproved vendor calls.

## Before Marketplace publication

- [ ] Seller/product setup and commercial model finalized.
- [ ] All paid nodes and infrastructure charges explained.
- [ ] Source AMI and templates satisfy current AWS review requirements.
- [ ] Image and dependency findings remediated.
- [ ] Template regions, parameters and IAM scopes tested.
- [ ] Clean independent buyer account completes launch and setup.
- [ ] License/usage behavior validated for the selected offer.
- [ ] Installation, recovery, update and uninstall guidance complete.
- [ ] Required listing diagram and usage/support artifacts prepared.
- [ ] Limited-release testing completed before public submission.

## Before a release upgrade

- [ ] Manifest/signatures verified and compatibility bounds checked.
- [ ] Backup and decryption/recovery access confirmed.
- [ ] Schema migration/rollback limits understood.
- [ ] Canary cohort and success criteria chosen.
- [ ] Failure capacity maintained during drain/replacement.
- [ ] Audit spool handling verified before deleting/replacing nodes.
- [ ] Post-upgrade policy, identity, traffic and SIEM checks passed.

## Glossary

| Term | Meaning in this design |
|---|---|
| Egress | Network path from customer systems toward external destinations |
| Explicit proxy | Client deliberately connects to a configured proxy endpoint |
| Transparent gateway | Network routes traffic through inspection without explicit client proxy settings |
| CONNECT | HTTP mechanism to establish a tunnel; inspection requires additional TLS handling |
| Inspection CA | Customer-trusted certificate authority used for authorized TLS interception |
| Control plane | Administration, policy, identity and fleet management |
| Data plane | Live traffic processing and enforcement |
| Policy bundle | Immutable versioned rules and capability metadata distributed to gateways |
| Coverage | Whether/how a request was actually inspected |
| Spool | Bounded local durable queue of events awaiting upload |
| Outbox | Database-backed records of asynchronous work to deliver |
| Lease | Time-bounded ownership of work that can be reclaimed after failure |
| RPO | Maximum acceptable data-loss interval, validated through recovery design |
| RTO | Target elapsed recovery time, validated through drills |
| AMI | EC2 machine image distribution format |
| BYOL | Bring your own license; separate from assuming offline compatibility |
| NLB / ALB | AWS network/application load balancers with different roles here |
| GWLB | AWS gateway load balancer for compatible network appliances |
| GENEVE | Encapsulation protocol used by the proposed future GWLB integration |
| IdP / OIDC | Identity provider / OpenID Connect authentication integration |
| SSE / SASE | Enterprise security/network delivery categories; not proof of connector compatibility |
| SBOM | Software bill of materials recording distributed components |
| Private mode | No unapproved vendor/external inspection dependencies; allowed AI egress may remain |
| Disconnected | Required services/destinations are local; public AI services are unreachable |

## Primary sources

External AWS requirements were reviewed on 2026-09-19 and must be rechecked before submission. Design recommendations in this handbook are Zotline proposals, not statements that AWS or an existing partner has approved the architecture.

| Source | Use |
|---|---|
| [AWS AMI product requirements](https://docs.aws.amazon.com/marketplace/latest/userguide/product-and-ami-policies.html) | Image preparation, security and listing constraints |
| [CloudFormation delivery](https://docs.aws.amazon.com/marketplace/latest/userguide/cloudformation.html) | Template and diagram submission |
| [Creating AMI products](https://docs.aws.amazon.com/marketplace/latest/userguide/ami-single-ami-products.html) | Product lifecycle and Limited testing |
| [AMI pricing models](https://docs.aws.amazon.com/marketplace/latest/userguide/pricing-ami-products.html) | Commercial model choices |
| [AMI contract pricing](https://docs.aws.amazon.com/marketplace/latest/userguide/ami-contracts.html) | Contract offer design |
| [License Manager integration](https://docs.aws.amazon.com/marketplace/latest/userguide/ami-license-manager-integration.html) | Contract entitlement integration |
| [AMI custom metering](https://docs.aws.amazon.com/marketplace/latest/userguide/custom-metering-with-mp-metering-service.html) | Instance-role MeterUsage integration |
| [Product submission](https://docs.aws.amazon.com/marketplace/latest/userguide/product-submission.html) | Submission and review process |
| [NLB listeners](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-listeners.html) | TCP/TLS forwarding choice |
| [NLB health checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-health-checks.html) | All-targets-unhealthy behavior |
| [GWLB setup](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/getting-started.html) | GENEVE appliance requirements |
| [HTTP CONNECT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT) | Tunnel versus inspection semantics |

Repository evidence is linked in [chapter 02](02-repository-and-gap-analysis.md). The [original blueprint](../architecture/zotline-self-hosted-blueprint.md) remains a shorter architecture overview.
