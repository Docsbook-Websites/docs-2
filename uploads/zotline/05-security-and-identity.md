---
title: "05 — Identity, certificates, security and privacy"
description: "05 — Identity, certificates, security and privacy — imported from 05-security-and-identity.md and published as part of these docs."
status: generated
version: "0.1"
---

# 05 — Identity, certificates, security and privacy

# 05 — Identity, certificates, security and privacy

[Handbook index](README.md)

## Trust boundaries and threats

| Boundary | Threat | Required control | Validation |
|---|---|---|---|
| Client → proxy | Open proxy abuse, identity spoofing | Authentication, network restrictions, destination ACL | Unauthorized CONNECT/identity tests |
| Proxy → provider | Spoofed upstream, DNS rebinding | Certificate/hostname verification and resolution policy | Invalid certificate and rebinding tests |
| User → policy | Selecting a more permissive team | Server-derived membership and explicit precedence | Cross-team requests |
| Model → decision | Prompt injection, invalid findings | Fixed schema, constrained output, deterministic policy action | Adversarial model-output fixtures |
| Gateway → management | Stolen node token, replay | Per-node credentials, scope, rotation/revocation | Revoked node cannot publish/read other scope |
| Admin → configuration | Misconfiguration or compromise | Least privilege, history, approval for trust changes | RBAC and rollback tests |
| Worker → integrations | SSRF, credential leakage | Destination allowlist and secret isolation | Redirect/private-address tests |
| Release → customer | Tampered image/model/update | Signatures, pinned digests, provenance | Tampered bundle rejection |
| Tenant → tenant | Shared cache/restoration leak | Scope in every key/query | Concurrent isolation tests |

The appliance operator controls the host. Do not claim encryption protects data from a malicious root administrator on that same host. The design reduces accidental exposure and separates privileges; stronger operator-isolation requirements need a separate hardware/confidential-computing design.

## Console identity

Use customer OIDC with issuer, audience, signature, expiry and nonce/state validation, secure cookies and CSRF protection. Map immutable issuer/subject pairs to users; email is a display/contact attribute, not the sole stable identity. Explicitly allow issuers and callback origins. Never derive trusted redirect URLs from an arbitrary request Host header.

The identity adapter must cover invitations/provisioning, membership, deactivation and recovery, not just JWT verification. For a disconnected site, the IdP must also be local. SCIM is optional future provisioning unless implemented and tested.

Proposed application roles:

| Role | Permissions |
|---|---|
| Deployment administrator | Integration and infrastructure settings, enrollment, backup/update operations |
| Security administrator | Organization rules, publication, exceptions and security review |
| Team policy editor | Scoped drafts and simulation; no global mandatory-rule override |
| Auditor | Read scoped metadata and change history; export only when granted |
| Gateway node | Read assigned policy/config, heartbeat, append audit |
| Integration worker | Read its leased jobs and destination secrets only |

Require a customer-controlled recovery path with hardware/key-backed or equivalent strong authentication appropriate to the environment. Log recovery use. Do not keep a universal vendor administrator or hardcoded recovery password.

## Traffic identity

Choose a tested identity mode per client class: authenticated proxy protocol, client certificate for managed services, or signed/authenticated identity from a trusted corporate upstream. Browser login cookies for the admin console do not automatically authenticate arbitrary CONNECT connections.

If no trustworthy employee identity is available, apply an explicit unassigned/network-segment policy and show the attribution limit. Never attribute all gateway traffic to the administrator who enrolled it. Membership caches need TTL/revocation behavior; security-critical removals must not depend on an indefinite cache.

## Gateway enrollment and credentials

Proposed lifecycle: administrator creates a short-lived single-use enrollment grant → node generates a key pair locally → node proves the grant and submits its public key → control plane binds deployment/node identity → grant is consumed → node receives scoped credentials/config.

Replay, exhausted grants and deployment mismatch fail closed. Only credential hashes/public keys are stored where possible. Rotation uses overlap with bounded lifetime, then retires the previous identity. Revocation is enforced at management access immediately and at gateway local behavior according to a documented lease. An offline node cannot learn a new revocation instantly; policy/identity lease length defines that risk window.

## Certificate hierarchy

Separate three purposes: public/private management TLS, gateway service identity, and destination impersonation for customer-authorized inspection. Do not reuse one key for all three.

Prefer a customer-controlled root and a limited-lived intermediate dedicated to Zotline inspection. Generate keys within customer infrastructure. Where feasible, issue separate intermediates per node or bounded fleet group so compromise can be contained. A shared root trust anchors leaf certificates; distributing a private root key to every gateway is not required.

Plan:

1. Create/approve the inspection hierarchy and record fingerprints and owners.
2. Distribute trust through approved device management, including application-specific stores.
3. Validate the new trust chain with synthetic destinations before routing production traffic.
4. Rotate by deploying new trust first, changing gateway issuance second, and retiring old trust after the compatibility window.
5. On compromise, revoke/replace affected issuers and remove old trust using customer management tools.

Hardware-protected signing may be appropriate, but arbitrary cloud KMS keys are not automatically compatible with the current Go TLS certificate issuance path. Prototype signer interfaces, throughput and availability before choosing a key service.

## Secret lifecycle

Store secret references in configuration, not literal credentials. Separate database, IdP, SIEM, node and signing secrets. Limit read permission to the process needing each secret. Redact environment dumps, command arguments, support archives and startup errors. Back up required encryption keys independently, with customer recovery ownership.

Use encryption in transit and customer-controlled encryption at rest. Encrypted disks do not replace application authorization. Disable content-bearing core dumps or protect them with a controlled diagnostic workflow. Temporary files and swap behavior require an explicit privacy decision.

## Privacy classification

| Data | Default behavior |
|---|---|
| Original prompt/file | Process in bounded memory; do not persist by default |
| Masked content | May still be sensitive; do not treat as public |
| Restoration mapping | Sensitive original values; scoped, expiring, encrypted if persisted |
| Audit metadata | Minimal categories/counts/decision/version; retention configured |
| Identity/IP/destination | Customer data; restrict and retain deliberately |
| URLs/query strings | Strip secrets and content-bearing parameters |
| HTTP headers | Never log credentials/cookies by default |
| Model input/output | Inference only; no provider/application debug logging by default |
| Support bundle | Sanitized metadata, customer-reviewed export |

Metadata mode must cover exception paths, trace logs, SIEM payloads, exports and backups. Field names alone do not establish privacy. A field called `reason` can contain a copied prompt if the model or exception formatter supplies it.

## Hardening and incident response

Run services as dedicated non-root users, with read-only application files, explicit writable mounts, resource limits and minimal capabilities. Protect management endpoints from the proxy network. Keep dependency/model license inventories and patch ownership. Review parser/format libraries as untrusted-input code.

For a suspected compromise: isolate the affected node, preserve customer-approved evidence, revoke its identity, rotate affected credentials/issuers, replace from a verified image, and validate policy/trust state before return to service. Do not upload customer traffic captures to vendor systems as an automatic diagnostic step.
