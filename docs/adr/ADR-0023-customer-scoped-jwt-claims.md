# ADR-0023: Customer-scoped JWT claims

- **Status**: Proposed (in PR)
- **Date**: 2026-05-12
- **Author**: P. Gujarati (Security Architecture)

## Context

Customer-scope checking across services currently relies on a mix of claim names: `tenant_id` in older code (per ADR-0012), `customer_id` in some services, a service-local override in at least one place. The drift is real and reviewed quarterly with no improvement. With the new self-service customer surfaces planned for 2026, a single claim shape is required before further customer-touching routes ship.

## Decision

Standardise customer-scope claims under a `cus_` prefix:

- `cus_id` — the customer-scope identifier. Required on every customer JWT.
- `cus_scope` — sub-scope identifier used when an operation is bounded to a portion of the customer's tenancy. Optional; absence means the operation is unbounded within the customer.

Claims are issued with dual-key rotation: tokens are signed by either the current or previous signing key during a 30-day rotation window, after which only the current key is accepted. Verifiers MUST accept both keys during the window.

Service-local claim names (`tenant_id`, `customer_id`, etc.) are deprecated. Services migrating to this ADR's claim shape must reject the old claim names in their JWT middleware once cutover lands.

## Consequences

- Consistent claim names eliminate the per-service mapping currently maintained.
- Dual-key rotation lets verifiers tolerate key rolls without coordinated downtime.
- All services consuming customer JWTs must update once `jwt-middleware` v2.8+ ships (in progress).

## Alternatives rejected

- **Stay on `customer_id` everywhere.** Rejected: collides with an internal-routing claim that some services already populate independently.
- **Single mega-claim with structured value.** Rejected: harder to write conformant verifier checks across four language stacks; flat claims compose better with the middleware already deployed.

## References

- ADR-0012 — to be retired by this ADR.
- `SECURITY-POLICY.md` §3.1.
