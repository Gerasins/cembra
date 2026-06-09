# GraphQL Health Checks

Status: Mandatory

## Purpose

Define the canonical operational health-check model for the shared GraphQL gateway and all subgraphs.

## Scope

This rule applies to `graphql-main` and to every GraphQL subgraph in the shared supergraph.

## Rule

- Every subgraph must expose `/healthz`.
- A subgraph `/healthz` is a readiness-style operational check.
- A subgraph `/healthz` must verify that the application has started successfully.
- A subgraph `/healthz` must verify that the GraphQL runtime is operational by executing an internal-only `ping query` through the real GraphQL runtime path.
- The internal `ping query` must not be part of the public supergraph contract and must not be exposed to external clients.
- A subgraph `/healthz` must verify all required runtime dependencies of that subgraph.
- At present, all subgraphs verify Redis and database availability because both are required dependencies everywhere.
- `graphql-main` must expose `/health-life` for gateway-local health.
- `graphql-main` must expose `/healthz` as an aggregate system health endpoint.
- Gateway `/healthz` must query all subgraphs and report the overall GraphQL system state.
- Gateway `/healthz` uses strict aggregate semantics: if any subgraph is unhealthy, the aggregate `/healthz` is unhealthy.
- Gateway `/healthz` is the endpoint exposed to external monitoring for whole-system GraphQL health.

## Rationale

This gives every subgraph a consistent readiness contract while also giving the gateway a reliable whole-system signal for monitoring. The internal `ping query` checks real GraphQL request handling rather than only process availability, and strict aggregate semantics keep the monitoring meaning of gateway `/healthz` unambiguous.

## Exceptions

Exceptions should be documented explicitly when a subgraph has different dependency semantics or when gateway aggregation behavior changes.

## Related ADRs

None yet.
