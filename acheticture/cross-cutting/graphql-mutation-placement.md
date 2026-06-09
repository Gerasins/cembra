# GraphQL Mutation Placement

Status: Mandatory

## Purpose

Define where mutation logic belongs when the GraphQL BFF layer sits in front of separate backend services.

## Scope

This rule applies to GraphQL mutations exposed by the shared supergraph and its subgraphs.

## Rule

- If a GraphQL mutation corresponds to a capability that already has a separate backend API and the mutation logic should be shared, the GraphQL layer must invoke that backend API.
- In that case, the backend API remains the canonical home of the shared mutation logic.
- If no separate backend API is needed for shared mutation logic, the mutation logic may live in the GraphQL layer.

## Rationale

This preserves one reusable home for business logic when that logic must be shared outside the GraphQL layer, while still allowing the GraphQL layer to own mutations that are specific to the BFF context.

## Exceptions

Exceptions should be documented explicitly when they introduce a different ownership model for mutation logic.

## Related ADRs

- [`0001-graphql-query-direct-read-and-shared-mutation-api`](../docs/adr/0001-graphql-query-direct-read-and-shared-mutation-api.md)
