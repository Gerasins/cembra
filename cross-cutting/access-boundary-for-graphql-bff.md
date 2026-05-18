# Access Boundary For GraphQL BFF

Status: Mandatory

## Purpose

Define the canonical access-control boundary for the shared GraphQL BFF layer.

## Scope

This rule applies to the shared supergraph, gateway, and subgraphs serving internal and external UIs.

## Rule

- Requests must pass through an auth proxy before entering the shared GraphQL BFF layer.
- The auth proxy is responsible for validating incoming JWT access tokens.
- Authorization decisions for GraphQL operations and data access must be enforced in subgraphs using role-based authorization.
- Token validation and domain-level permission checks must not be treated as the same concern.

## Rationale

The system serves multiple UIs with different authorization models through one shared supergraph. A clear separation between authentication at the edge and authorization inside subgraphs keeps the access model explicit and consistent across the BFF layer.

## Exceptions

Any exception to this rule should be documented explicitly and justified in an ADR if it changes the architectural boundary.

## Related ADRs

None yet.
