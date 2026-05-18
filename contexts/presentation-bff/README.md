# Presentation / BFF Context

## Purpose

Provide the unified UI-facing GraphQL layer for multiple user interfaces through a shared supergraph, gateway, and subgraphs.

## Target Boundary

This context is responsible for:

- exposing the shared GraphQL contract used by the UIs
- composing the supergraph from subgraphs
- aggregating and orchestrating access to underlying domain systems
- adapting domain capabilities into UI-facing GraphQL shapes

This context is not responsible for being the source of truth of the underlying business domains.

## Current State / Known Divergences

- A shared GraphQL supergraph is used by 4 UIs.
- The gateway repository is `graphql-main`.
- The supergraph is composed from 12 subgraphs.
- At least one subgraph, `graphql-crm`, acts as an adapter subgraph over the CRM system.
- `graphql-crm` reads CRM data from the CRM database for queries.
- `graphql-crm` routes mutations through `crm-api`.
- `graphql-crm` does not use `crm-mq-client`.
- 3 UIs are internal and share one authorization model.
- 1 UI is external and uses a separate authorization model.
- Every subgraph exposes `/healthz`.
- `graphql-main` exposes `/health-life` and aggregate `/healthz`.

## Target

- The shared GraphQL layer is a first-class `Presentation / BFF Context`.
- Adapter subgraphs may read from underlying domain data sources directly for queries.
- If a GraphQL mutation belongs to a capability that already has a separate backend API and the mutation logic should be shared, the GraphQL layer should call that backend API.
- If no separate backend API is needed for shared mutation logic, the mutation logic may live in the GraphQL layer.
- GraphQL query paths may use `IQueryable`-based direct database access to support efficient reads and read-only replica usage.
- Health checks are standardized across the gateway and all subgraphs.

## Key Terms

- BFF GraphQL Service
- GraphQL Supergraph
- GraphQL Subgraph
- Gateway
- Adapter Subgraph
- Internal UI
- External UI
- Auth Proxy
- Role-Based Authorization

## Incoming Integrations

- Requests from internal and external UIs enter this context through the auth boundary in front of `graphql-main`.
- External monitoring observes whole-system GraphQL health through gateway `/healthz`.

## Outgoing Integrations

- Subgraphs call or represent underlying domain systems.
- `graphql-crm` integrates with the CRM system through direct reads and mutation calls.
- `graphql-crm` does not integrate through `crm-mq-client`.
- Gateway `/healthz` polls subgraph health endpoints to represent aggregate GraphQL system health.

## Related Project Repositories

- `graphql-main`
- `graphql-crm`
- `crm-api`

## Related ADRs

- [`0001-graphql-query-direct-read-and-shared-mutation-api`](../../docs/adr/0001-graphql-query-direct-read-and-shared-mutation-api.md)

## Related Cross-Cutting Rules

- [`access-boundary-for-graphql-bff`](../../cross-cutting/access-boundary-for-graphql-bff.md)
- [`graphql-query-data-access`](../../cross-cutting/graphql-query-data-access.md)
- [`graphql-mutation-placement`](../../cross-cutting/graphql-mutation-placement.md)
- [`graphql-crm-does-not-use-mq-client`](../../cross-cutting/graphql-crm-does-not-use-mq-client.md)
- [`graphql-health-checks`](../../cross-cutting/graphql-health-checks.md)
