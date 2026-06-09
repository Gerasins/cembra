# Context

System-wide glossary of canonical domain terms.

## Rules

- Keep this file free of implementation details.
- Add only terms that are useful across the whole system.
- Use one canonical term per concept.
- If two words are commonly confused, define both and clarify the distinction.

## Terms

### BFF GraphQL Service

A backend-for-frontend service that exposes a GraphQL API shaped around the needs of one or more user interfaces.

Optional notes:

- it is not automatically a domain context of its own
- it may aggregate data and operations from multiple domain contexts
- it should not be confused with a domain service that happens to expose GraphQL

### GraphQL Supergraph

The shared GraphQL API surface exposed to user interfaces and composed from multiple subgraphs.

Optional notes:

- it represents the unified BFF layer for multiple UIs
- it should not be confused with an individual subgraph

### GraphQL Subgraph

A GraphQL service that contributes part of the schema and behavior to the supergraph.

Optional notes:

- a subgraph may act as an adapter over a domain system instead of being the domain system itself
- a subgraph should not automatically be treated as a domain context

### Gateway

The runtime component that composes the supergraph from subgraphs and serves the unified GraphQL entry point to user interfaces.

Optional notes:

- it is part of the BFF layer
- it should not be confused with the business services behind the subgraphs

### Adapter Subgraph

A GraphQL subgraph in the BFF layer that represents or orchestrates access to an underlying domain system without being the source of truth for that domain.

Optional notes:

- it may read from one system and route writes to another interface of the same domain capability
- it should not be confused with the underlying domain service or domain context

### CRM System

The canonical CRM capability and source of truth for CRM data and behavior in the system.

Optional notes:

- it may expose multiple technical interfaces
- its interfaces should not be confused with the system itself

### CRM API

A direct-call interface of the CRM system used for synchronous operations.

Optional notes:

- it is an interface of the CRM system, not the CRM system itself

### CRM MQ Client

A messaging-oriented interface or component of the CRM system used for event-based operations.

Optional notes:

- it is an interface of the CRM system, not the CRM system itself

### Presentation / BFF Context

The architectural context responsible for UI-facing GraphQL composition, aggregation, and orchestration across underlying domain systems.

Optional notes:

- it exposes the unified contract used by user interfaces
- it may include the supergraph, gateway, and adapter subgraphs
- it is not the source of truth for the underlying business domains

### Internal UI

A user interface intended for internal users and governed by the internal authorization model.

Optional notes:

- multiple internal UIs may share the same BFF layer
- it should not be confused with external or partner-facing interfaces

### External UI

A user interface intended for external users and governed by the external authorization model.

Optional notes:

- it may share the same supergraph with internal UIs while still requiring separate authorization boundaries
- it should not be confused with internal staff-facing interfaces

### Auth Proxy

The boundary component in front of the GraphQL supergraph that validates incoming access tokens before requests enter the shared BFF layer.

Optional notes:

- it performs authentication at the edge of the BFF layer
- it should not be confused with domain-level permission checks

### Role-Based Authorization

An authorization model in which access to operations or data is granted according to assigned roles.

Optional notes:

- in this system it is enforced within subgraphs after requests pass the authentication boundary
- it should not be confused with token validation

### GraphQL Mutation

A write operation exposed through the GraphQL layer.

Optional notes:

- it may delegate execution to an underlying backend API
- it should not automatically become the canonical home of reusable business logic

### GraphQL Query

A read operation exposed through the GraphQL layer.

Optional notes:

- it may access read-optimized data paths directly
- it should not be confused with mutation logic or command handling

### IQueryable

A query composition interface that allows GraphQL query execution to be translated into database queries rather than materializing all data eagerly in the application layer.

Optional notes:

- in this system it supports direct query access patterns from GraphQL subgraphs
- it is used to enable efficient read paths and read-only replica usage

### Health Check

An operational check used to determine whether a component or dependency is functioning as expected.

Optional notes:

- health checks may exist at both subgraph level and gateway level
- they should not be confused with business-level correctness checks

### health-life

A service-local health endpoint that reports whether the component itself and its required dependencies are operational.

Optional notes:

- in this system it exists on individual subgraphs and on the gateway

### healthz

An aggregate health endpoint exposed by the gateway to represent the overall state of the GraphQL system.

Optional notes:

- it aggregates subgraph health state
- it is intended for external monitoring visibility
