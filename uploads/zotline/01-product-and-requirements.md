---
title: "01 — Product, users, scope and requirements"
description: "01 — Product, users, scope and requirements — imported from 01-product-and-requirements.md and published as part of these docs."
status: generated
version: "0.1"
---

# 01 — Product, users, scope and requirements

# 01 — Product, users, scope and requirements

[Handbook index](README.md)

## What Zotline is

Zotline applies company data-sharing policies to supported AI traffic passing through a company-controlled network gateway. It can allow a request, transform sensitive fields before forwarding, or prevent the request. It also creates decision evidence for administrators.

An employee should normally keep using the same supported browser or application. The network configuration changes; installing a Zotniq endpoint agent is not required for traffic already routed through Zotline. Certificate trust and corporate routing still require management. Unmanaged devices, off-network users and applications that cannot use the route are not automatically covered.

An appliance AMI is one distribution format. Installing the AMI in AWS does not route office traffic into it automatically. The customer must establish VPN/Direct Connect or another supported network path, configure clients, and enforce egress restrictions.

## Product boundaries

| Capability | Agent | SDK | Zotline |
|---|---|---|---|
| Enforcement location | Employee device | Application process/integration | Corporate network path |
| User attribution | Device enrollment and local context | Service/API identity | Authenticated proxy or trusted network identity |
| Local process inventory | Endpoint capability | Not generally available | Not available from network traffic alone |
| Inspect off-network traffic | Depends on installed agent health | Depends on app integration | Only if routed through gateway |
| API-specific integration | Protocol adapters | Application wrappers | Protocol adapters |
| Central policy administration | Reusable control plane | Reusable control plane | Customer-local control plane |

Zotline is not a replacement for the customer's firewall, identity provider, endpoint management or SIEM. It needs explicit integration boundaries with them. It cannot guarantee interpretation of arbitrary encrypted applications, unsupported file formats, malicious encodings or every possible AI destination.

## Personas and work

**Deployment administrator:** installs the appliance, connects identity, configures trust, storage, updates and recovery. Does not automatically need access to captured content.

**Security administrator:** defines inspection scope and policy, approves exceptions, reviews coverage and incidents.

**Team owner:** proposes team rules and reviews scoped decisions. Cannot weaken organization-wide mandatory rules.

**Auditor:** reads configuration history and decision metadata; cannot modify policies or infrastructure.

**Application owner:** configures an approved proxy/service identity, validates functionality and handles blocked-request errors.

**Employee:** receives a useful explanation and request identifier when blocked, without having to understand infrastructure details.

**Customer support operator:** receives customer-approved diagnostics. The vendor has no default account or remote access into an installation.

## End-to-end example

A customer installs two gateways in its private AWS network. Its finance users have managed proxy configuration and trust the inspection CA. A finance policy prohibits sending bank account numbers to public AI services while permitting general business text.

On a supported request, Zotline authenticates the connection, resolves finance membership, parses the prompt, detects the account field, applies the active rule and sends a transformed request or a block response. The event records the rule and policy version. An authorized reviewer can see why the decision occurred without needing the original account number.

The same customer can permit a private internal AI destination under a different policy. Destination rules must distinguish that exact service from arbitrary private addresses; an internal hostname is not automatically trusted.

## Functional requirements

| ID | Requirement | Evidence for completion |
|---|---|---|
| FR-01 | Route and inspect documented supported traffic | Client/protocol test matrix |
| FR-02 | Resolve user, service or restrictive unknown identity | Identity spoof/revocation tests |
| FR-03 | Apply organization and team policy consistently | Signed bundle and precedence fixtures |
| FR-04 | Prevent blocked original content reaching upstream | Capturing synthetic upstream tests |
| FR-05 | Transform supported payloads without breaking protocol | Parser, stream and adapter conformance |
| FR-06 | Keep content in customer environment during inspection | Egress-denied private-mode tests |
| FR-07 | Persist minimal decision evidence and forward reliably | Crash/replay/deduplication tests |
| FR-08 | Install without vendor operator intervention | Fresh-account/fresh-VM test |
| FR-09 | Upgrade and recover safely | Previous-release upgrade and restore drill |
| FR-10 | Expose gaps and degraded protection visibly | Failure-injection dashboard checks |
| FR-11 | Enforce chosen license model without hidden bypass | Entitlement/outage test matrix |
| FR-12 | Remove deployment and trust safely | Uninstall/revocation drill |

## Nonfunctional requirements

- Bound request memory, parser work, model queues, database connections and audit storage.
- Keep deterministic traffic decisions independent of a healthy management API.
- Validate upstream TLS; interception is not a reason to disable destination certificate checks.
- Keep tenant/user context on every policy lookup, cache entry and restoration map.
- Detect and report unsupported inspection, not just successful proxy forwarding.
- Maintain version compatibility across gateway, control plane, schema, policy and model artifacts.
- Document actual availability and data loss behavior; a redundant topology alone is not an SLA.
- Support customer-controlled update windows, data retention and diagnostic export.

Performance objectives are established using chapter 12. No throughput, accuracy, zero-downtime or compliance certification claim is established by this document.

## First-release scope

Release one supports an explicit proxy, a small named set of AI browser/API adapters, private control plane, deterministic rules, optional validated private inference, single-node and AWS HA profiles, and metadata audit export. Finish an exact adapter matrix before announcing support.

Deferred capabilities: transparent GWLB appliance, Linux routed/bridge appliance, certified Zscaler/Netskope connectors, arbitrary upload formats/OCR, ARM images, and a Kubernetes distribution. Deferral does not prevent design preparation, but their code and validation must not be assumed present.

## Customer responsibility agreement

The customer supplies network authority, DNS, identities, approved inspection scope, device trust distribution, capacity and backups ownership. Zotniq supplies signed software, supported deployment definitions, compatibility guidance, tested upgrades and support procedures. Assign one named owner per responsibility before a pilot.

For legal/compliance needs, provide a configurable evidence and retention system. Do not present use of Zotline as automatic compliance with a regulation or certification.
