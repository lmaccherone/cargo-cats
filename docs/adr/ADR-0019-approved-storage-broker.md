# ADR-0019: Approved storage broker for customer-originating content

- **Status**: Accepted
- **Date**: 2025-09-08
- **Deciders**: P. Mehta (Security Architecture), M. Lin (Platform)

## Context

CargoCats services accept customer-uploaded content from several entry points: photos through ImageService, labels through LabelService, documents through DocService. Each service handled storage independently and the result was drift: tenant scoping derived differently across services, path handling that bypassed normalisation on new routes, inconsistent audit emission. The Q2 2025 compliance review flagged the gap.

`@cargocats/storage-broker` v3.1 centralises tenant-scoped storage, path normalisation, virus scanning, and audit emission. Language SDKs exist for .NET, Node.js, Python, and Java.

## Decision

All customer-originating content must be stored, retrieved, and exposed through `@cargocats/storage-broker` v3.1 or later. Direct filesystem writes, direct object-store SDK calls, and per-service blob clients are prohibited for customer content.

- Uploads call `broker.put(scope, content, metadata)` where `scope = broker.resolveScope(jwt)`.
- Retrieval is by the opaque handle returned by `put`. Routes must not accept caller-supplied paths or keys.
- Audit records are emitted by the broker; services do not add their own.

## Status

In force. DocService migrated 2025-10. LabelService migrated 2025-12. **ImageService migration pending** (PLAT-2841, originally targeted Q1 2026, slipping). New customer-upload features must use the broker from first commit regardless of the host service's overall migration status.

## Consequences

- Tenant-scoping and path-traversal classes of bug close as services finish migration.
- The broker is a stateful cluster service; outages now block uploads cluster-wide.
- SDKs must stay version-aligned across services; coordinated upgrades expected.

## Alternatives rejected

- **Per-service hardening with a shared review checklist.** Tried in 2024 — drift kept reappearing in new features. The problem is structural, not procedural.
- **Direct object-store SDK with shared IAM.** Tenant scoping still lives in per-service code, so the same drift recurs. IAM doesn't model per-customer scope cleanly without per-customer roles, which doesn't scale.

## References

- `SECURITY-POLICY.md` §4.2
- `docs/approved-components.md` — `storage-broker` v3.1
- PLAT-2841 — ImageService migration
