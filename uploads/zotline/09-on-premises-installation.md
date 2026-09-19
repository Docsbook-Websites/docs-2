---
title: "09 — On-premises, offline and customer onboarding"
description: "09 — On-premises, offline and customer onboarding — imported from 09-on-premises-installation.md and published as part of these docs."
status: generated
version: "0.1"
---

# 09 — On-premises, offline and customer onboarding

# 09 — On-premises, offline and customer onboarding

[Handbook index](README.md)

These are target installation procedures. There is no released `zotline install` command or supported appliance image established by this repository review. The installer must implement these steps and pass chapter 12 before they become executable customer instructions.

## Prerequisites worksheet

| Item | Customer supplies/decides |
|---|---|
| Platform | Supported Linux VM host, architecture and capacity |
| Network | Client routing, firewall rules, DNS, IPv4/IPv6 coverage |
| Trust | Inspection CA approval, management TLS, device/application trust distribution |
| Identity | Local/reachable IdP, issuer/client details, group mapping, recovery owner |
| Storage | Durable encrypted data, backup target, retention and capacity |
| Database | Bundled single-node PostgreSQL or supported customer-managed service |
| Inference | Deterministic-only policy scope or supported private model endpoint |
| Integrations | Approved SIEM endpoint, customer credentials, destination allowlist |
| Commercial | Signed offline license or supported connected licensing arrangement |
| Operations | Update windows, alert recipients, incident and restore owners |

Use a representative pilot client and a synthetic upstream receiver. Never validate masking for the first time with real customer secrets sent to a public provider.

## Distribution contents

Release bundle contains a signed manifest, pinned service artifacts, supported runtime prerequisites, schema migrations, default catalogs, SBOM/license inventory, configuration schema, trust-verification instructions and installation/restore tools. An offline bundle also includes everything normally downloaded, including selected model/tokenizer artifacts when licensed for redistribution.

Verify the manifest signature with a trust key obtained through an independent authenticated channel, then verify every artifact digest. A checksum delivered alongside a tampered file does not authenticate its publisher. Record release ID and verified manifest in the deployment evidence.

## Filesystem design

Proposed layout:

```text
/opt/zotline/releases/<version>/   immutable binaries/images and manifest
/etc/zotline/                     validated configuration and trust references
/etc/zotline/secrets/             protected local secrets when no vault is used
/var/lib/zotline/policy/          verified policy cache
/var/lib/zotline/spool/           encrypted pending audit events
/var/lib/zotline/postgres/        bundled database, single-node profile only
/var/lib/zotline/artifacts/       customer-approved retained files and updates
/var/log/zotline/                 content-free bounded operational logs
```

Separate service ownership and permissions. Do not put durable state under `/tmp`, inside a disposable container layer or on an unbacked root disk. Validate free space, mount identity and writable permissions before becoming ready; a missing mount must not silently redirect writes onto the root filesystem.

## Single-node installation procedure

1. Allocate the supported VM and durable data storage. Set DNS and verified time synchronization.
2. Verify/import the release bundle and its exact prerequisite versions.
3. Generate deployment identity and customer-local secrets; never copy identity from a previous VM image.
4. Configure listeners, private mode, database, storage and integration references.
5. Initialize PostgreSQL and run the release migration once under a lock.
6. Start management services in unclaimed mode, reachable only through the intended admin network.
7. Claim using the one-use setup credential; bind customer IdP and assign initial administrators.
8. Establish management TLS and inspection CA; configure certificate renewal ownership.
9. Select supported detector/model profile and publish a tested initial policy.
10. Start gateway and background worker; verify role-specific readiness.
11. Configure one pilot client, establish trust, run allow/mask/block checks, and confirm metadata-only audit.
12. Back up and restore into an isolated test environment before broad rollout.

Bootstrap must be idempotent: after a power failure it resumes without replacing keys, creating a new deployment or reinitializing an existing database. On a rerun, compare existing state to the manifest and require explicit migration/recovery where they differ.

## HA installation procedure

Provision database HA and storage under named customer ownership first. Install management nodes with a common deployment configuration and separate node identities. Run the migration once, then start workers using leases. Enroll each gateway independently and place it behind the approved TCP load balancer.

Validate health-based routing, drain behavior, policy distribution and the surviving capacity with one gateway offline. Test database failover and the customer's load-balancer failure separately. Two gateway VMs on the same physical host or storage array may still share a failure domain; document this.

## Disconnected operation

The strict profile must use local identity, DNS/time, inference, documentation and update/license verification. Block outbound internet access during qualification and exercise every administrator workflow. Remove hosted avatar/fonts/analytics dependencies from the console if they are otherwise required for rendering or functionality.

Import updates via a customer-approved transfer process. Verify signatures/digests inside the isolated environment. Scan and approve the transfer medium according to customer process, stage artifacts locally, then run the standard canary/update workflow. Offline licenses require clock and expiry handling; detect clock rollback without pretending a customer-controlled host provides tamper-proof time.

Public AI destinations are unavailable in a completely disconnected network. If the management plane is offline but selected AI egress remains available, document that as a restricted-egress deployment rather than air-gapped.

## Daily user experience

Employees see normal allowed requests, a transformed result where supported, or a useful block message with a safe reason and request reference. Avoid showing raw detected secrets in the block page. A request for an exception goes to the customer's administrator with scope and expiry, not directly to vendor sales.

Administrators review protection coverage, pending exceptions, stale nodes, policy versions, inference saturation, failed audit exports and certificate expiry. A monitor-only pilot must clearly display that prevention is not active.

## Moving from another deployment

Inventory team/rule identities and translate with an explicit mapping. Do not blindly copy cloud credentials, vendor user IDs or license rows into a self-hosted deployment. Export approved configuration, migrate through versioned tools, re-enroll node identities and confirm data residency. Dual-run in monitor mode may help compare decisions, but only one component should actively rewrite a given request unless the chain is tested.

## Uninstall and decommission

1. Export customer-required records and confirm their readability.
2. Move clients/routing to an approved alternative; verify no proxy dependency remains.
3. Drain gateways and stop accepting new traffic.
4. Revoke gateway/integration credentials and remove distributed inspection trust as appropriate.
5. Stop services and inventory retained data, backups, keys and infrastructure.
6. Apply customer-approved retention/deletion decisions; do not automatically destroy evidence.
7. Remove unneeded network/DNS rules and validate normal traffic behavior.
8. Record decommission completion and responsibility for retained backups.
