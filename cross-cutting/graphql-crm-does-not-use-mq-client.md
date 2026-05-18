# GraphQL CRM Does Not Use MQ Client

Status: Mandatory

## Purpose

Define the allowed integration paths between `graphql-crm` and the CRM system.

## Scope

This rule applies to the `graphql-crm` adapter subgraph and its integration with CRM capabilities.

## Rule

- `graphql-crm` must not use `crm-mq-client`.
- `graphql-crm` may read approved CRM read paths directly for queries.
- `graphql-crm` must route shared mutation logic through `crm-api`.

## Rationale

This keeps `graphql-crm` aligned with the canonical split between direct read paths for GraphQL queries and backend API ownership of shared mutation logic. It also prevents `graphql-crm` from taking on event-driven integration responsibilities that belong elsewhere in the CRM system.

## Exceptions

Any exception should be documented explicitly and justified in an ADR because it changes the integration boundary of the CRM adapter subgraph.

## Related ADRs

- [`0001-graphql-query-direct-read-and-shared-mutation-api`](../docs/adr/0001-graphql-query-direct-read-and-shared-mutation-api.md)
