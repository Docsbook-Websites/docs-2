---
title: "Zotline engineering and deployment handbook"
description: "Zotline engineering and deployment handbook — imported from README.md and published as part of these docs."
status: generated
version: "0.1"
---

# Zotline engineering and deployment handbook

# Zotline engineering and deployment handbook

**Version:** design baseline 0.1 · **Reviewed:** 2026-09-19 · **Status:** proposed, not a released appliance.

Zotline is Zotniq's customer-operated AI traffic gateway. This handbook specifies what to build, how its parts work together, how customers install and operate it, and what evidence is required before release. It expands the [original blueprint](../architecture/zotline-self-hosted-blueprint.md).

The repository contains reusable proxy, policy, API and UI implementations. It does not yet provide all the gateway interfaces, installer commands, deployment templates or guarantees described here. Configuration keys and interfaces marked **proposed** are engineering contracts, not instructions to invoke an existing product. Installation chapters are target runbooks to validate during implementation.

## Read by role

| Reader | Start here | Then read |
|---|---|---|
| Founder/product owner | 01, 03 | 10, 13 |
| Gateway engineer | 02, 04 | 05, 06, 07, 12 |
| Backend/UI engineer | 02, 03, 07 | 05, 06, 11 |
| Infrastructure engineer | 03, 08, 09 | 10, 11, 12 |
| Security reviewer | 04, 05 | 06, 07, 12 |
| Customer administrator | 01, 09 | 06, 11, 14 |
| Release/Marketplace owner | 08, 10 | 12, 13 |

## Contents

1. [Product, users, scope and requirements](01-product-and-requirements.md)
2. [Repository evidence and reuse plan](02-repository-and-gap-analysis.md)
3. [System architecture and deployment profiles](03-system-architecture.md)
4. [Traffic routing, protocol handling and gateway internals](04-gateway-and-networking.md)
5. [Identity, certificates, security and privacy](05-security-and-identity.md)
6. [Policies, detection, inference and administration](06-policy-and-inference.md)
7. [API, data and configuration contracts](07-api-data-and-configuration.md)
8. [AWS infrastructure and installation](08-aws-deployment.md)
9. [On-premises, offline and customer onboarding](09-on-premises-installation.md)
10. [AMI build, licensing and Marketplace publication](10-marketplace-and-release.md)
11. [Operations, upgrades, recovery and troubleshooting](11-operations-runbooks.md)
12. [Testing, sizing and release acceptance](12-testing-and-capacity.md)
13. [Implementation backlog and architecture decisions](13-engineering-roadmap.md)
14. [Checklists, glossary and sources](14-reference-and-checklists.md)

## How to interpret requirements

- **Observed:** implementation or product wording inspected in this repository. File links accompany key claims.
- **Required design:** a proposed condition for the Zotline release, not an existing capability.
- **Recommended:** the selected starting design, subject to an explicit architecture decision if changed.
- **Deferred:** out of the first release; do not advertise it as available.
- **Target:** a value to validate experimentally, not a measured result or contractual guarantee.

The baseline is an explicit proxy on Linux x86-64, with the control plane in the customer's environment. Private inference is optional only when the active policies do not require it. AWS and on-premises editions share the inspection engine and application contracts. Transparent networking, certified SSE integrations and additional CPU/platform support have their own acceptance gates.

## Document governance

Keep this handbook in the repository with the implementation. An interface change updates its chapter, fixtures and compatibility manifest in the same change. Record approved departures in chapter 13. Release documentation must replace proposed names with tested commands and attach the tested support matrix, image IDs and version bounds.

This directory is an engineering handbook outside the existing public MkDocs `content/` tree. Publishing customer-facing pages is a separate editorial step after features are implemented and verified. No infrastructure, accounts, billing configuration or public site was changed to create this handbook.
