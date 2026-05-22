# ADR-0012: Tenant isolation for the customer portal

- **Status**: Accepted
- **Date**: 2025-03-14
- **Deciders**: J. Reeves (Engineering, since left CargoCats), C. Patel (Platform)

## Context

The customer portal was being unified onto a single set of FrontGate routes in early 2025. Several downstream services accept customer-identifying parameters as part of the request path (`/api/customers/{customerId}/...`). Without a consistent isolation check at the service tier, a route reachable for one customer could return data scoped to another.

This ADR set the per-service pattern that was current at the time.

## Decision

Each downstream service exposes a `withTenantPrefix(req)` helper that compares the `tenant_id` JWT claim against the path's `customerId` segment and rejects on mismatch. Services compose this helper into route handlers.

## Consequences

- Consistent rejection behaviour across services at the time of adoption.
- The helper is per-service-language; each service ships its own copy of the pattern.

## Alternatives rejected

- **Middleware at the FrontGate ingress only.** Rejected at the time on the grounds that downstream services were also reachable from internal entry points that bypass FrontGate.

## References

- Customer portal unification work (Q1 2025).
