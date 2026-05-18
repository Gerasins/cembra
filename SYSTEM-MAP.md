# System Map

## Purpose

Describe the system at a glance:

- what the system is for
- where its boundaries are
- which domain contexts exist
- which project repositories implement them
- how the major integrations work

## Scope

This document should stay short and overview-oriented. Detailed design belongs in context, integration, cross-cutting, and ADR documents.

## System Boundary

The system includes:

- 4 user interfaces
- a shared `Presentation / BFF Context` built around a GraphQL supergraph
- multiple underlying domain systems exposed to the BFF layer through subgraphs and backend APIs

The architecture repository currently documents the shared GraphQL BFF layer as one of the primary system contexts.

## Domain Contexts

- [`Presentation / BFF Context`](contexts/presentation-bff/README.md)
  Shared UI-facing GraphQL layer built around one supergraph, one gateway repository (`graphql-main`), and 12 subgraphs.
- `CRM System`
  Source of truth for CRM data and behavior. Exposed through `crm-api`, `crm-mq-client`, and approved read paths used by adapter subgraphs.

## Project Repositories

- `graphql-main`
  Gateway repository for the shared GraphQL supergraph in the `Presentation / BFF Context`.
- `graphql-crm`
  Adapter subgraph representing CRM capabilities in the shared supergraph.
- `crm-api`
  Backend API interface of the `CRM System` used for shared mutation logic and direct synchronous operations.

## Major Integrations

- 4 UIs call the shared GraphQL BFF layer through an `auth-proxy`.
- The `auth-proxy` validates JWT access tokens before requests enter `graphql-main`.
- `graphql-main` composes one supergraph from 12 subgraphs.
- Subgraphs enforce role-based authorization for GraphQL operations and data access.
- `graphql-crm` reads approved CRM data paths directly for queries and delegates shared mutation logic to `crm-api`.
- `graphql-crm` does not use `crm-mq-client`.
- Every subgraph exposes `/healthz` as a readiness-style operational health check.
- `graphql-main` exposes `/health-life` for gateway-local health and `/healthz` as a strict aggregate health endpoint for the whole GraphQL system.

Architecture view:

- [`docs/diagrams/c4-presentation-bff-architecture.puml`](docs/diagrams/c4-presentation-bff-architecture.puml)

## Cross-Cutting Rules

- [`cross-cutting/access-boundary-for-graphql-bff.md`](cross-cutting/access-boundary-for-graphql-bff.md)
- [`cross-cutting/graphql-query-data-access.md`](cross-cutting/graphql-query-data-access.md)
- [`cross-cutting/graphql-mutation-placement.md`](cross-cutting/graphql-mutation-placement.md)
- [`cross-cutting/graphql-crm-does-not-use-mq-client.md`](cross-cutting/graphql-crm-does-not-use-mq-client.md)
- [`cross-cutting/graphql-health-checks.md`](cross-cutting/graphql-health-checks.md)

## Known Divergences

No architecture-level divergences documented here yet for the GraphQL BFF layer.
