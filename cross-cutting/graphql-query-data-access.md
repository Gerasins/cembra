# GraphQL Query Data Access

Status: Mandatory

## Purpose

Define the canonical read-path model for GraphQL queries in the shared BFF layer.

## Scope

This rule applies to GraphQL queries exposed by the shared supergraph and its subgraphs.

## Rule

- GraphQL query paths may access underlying databases directly instead of going through backend APIs.
- This direct query access may use `IQueryable`-based composition so that GraphQL queries translate efficiently into database queries.
- Direct query access may use read-only replicas to reduce load on the underlying systems.
- Direct query access should be limited to approved read paths such as agreed read models, replicas, or explicitly allowed database schemas.
- Direct query access should not be treated as blanket permission to read arbitrary operational tables of a domain system.
- This rule applies to read paths only. It does not change the mutation placement rule.

## Rationale

Direct database access for GraphQL queries enables efficient read behavior, supports read-only replicas, and reduces load on backend systems that would otherwise serve read traffic through APIs not optimized for GraphQL query composition.

## Exceptions

Exceptions should be documented explicitly when a query path must go through a backend API instead of direct data access.

## Related ADRs

- [`0001-graphql-query-direct-read-and-shared-mutation-api`](../docs/adr/0001-graphql-query-direct-read-and-shared-mutation-api.md)
