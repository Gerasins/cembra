# GraphQL Fusion

## Purpose

This document defines how GraphQL Fusion should be understood in this project.
Use it as the canonical description of the Fusion model, its terminology, and
the decision rules behind lookup selection.

## Problem Statement

Schema composition becomes expensive when every cross-service field requires a
manually attached resolver. Fusion addresses that problem by composing data
sources around shared types instead of around field-by-field stitching code.

## Core Idea

Fusion merges compatible types from multiple sources by:

1. discovering lookups that return the same type;
2. comparing the type shape;
3. selecting an authoritative source when fields overlap;
4. using additional lookups only when the requested fields require them.

Fusion is therefore type-centric, not resolver-centric.

## Glossary

- `lookup`: a query or resolver that returns an object of a known GraphQL type.
- `external lookup`: a lookup exposed to API consumers and eligible to act as an authoritative source.
- `inner lookup`: a lookup used only inside the composed graph and not exposed directly to clients.
- `authoritative source`: the lookup Fusion prefers when multiple sources can provide the same type or field set.
- `composed type`: the unified type produced after Fusion aligns compatible source types.

## Decision Rules

Fusion should follow these rules:

1. Match types by GraphQL type name first.
2. Validate compatibility by comparing field names and field types.
3. Prefer an `external lookup` when an authoritative source must be chosen.
4. Use `inner` lookups only to complete fields that are missing from the selected external source.
5. Base lookup selection on the actual client field selection, not on static schema presence alone.

## Execution Model

When a client requests a composed type, Fusion works as follows:

1. Resolve the requested root lookup.
2. Inspect the return type and the requested field set.
3. Determine whether one source can satisfy the full selection.
4. If not, augment the response with compatible secondary lookups.
5. Return a single composed object to the client.

The important consequence is that Fusion may use multiple backing sources while
still presenting one logical GraphQL type.

## Example

Two source schemas expose the same logical entity:

```graphql
# Source A
type User {
  id: ID!
  name: String
}

type Query {
  userById(id: ID!): User
}

# Source B
type User {
  id: ID!
  email: String
}

type Query {
  userEmailById(id: ID!): User
}
```

A client requests:

```graphql
query {
  userById(id: "123") {
    id
    name
    email
  }
}
```

Expected Fusion behavior:

1. Treat both lookups as candidates for the `User` type.
2. Use `userById` as the entry lookup.
3. Detect that `email` must be completed from `userEmailById`.
4. Return one composed `User` object to the client.

## When Fusion Fits Well

Fusion is a good fit when:

- multiple sources expose the same business entity;
- type names and field types can stay consistent across schemas;
- the team wants to reduce field-level boilerplate;
- the unified graph should choose data sources dynamically based on requested fields.

## Risks and Constraints

Fusion depends on schema discipline.

- Type names must be intentionally shared.
- Field definitions must remain compatible.
- Ownership conflicts need explicit rules.
- Debugging requires tracing because lookup selection is not always obvious from the client query alone.

## Comparison With Stitching

Fusion differs from Stitching in one important way:

- Stitching says: attach a resolver to a field.
- Fusion says: declare compatible sources for a type and let the engine compose them.

If the integration problem is mostly about shared entity shapes, prefer Fusion.
If the integration problem is mostly about custom field-level delegation, prefer
Stitching.

## Copilot Notes

When extending docs or code related to Fusion, keep these assumptions explicit:

- name the authoritative source;
- distinguish `external` and `inner` lookups;
- document compatibility requirements for shared types;
- describe behavior in terms of requested fields and lookup selection.
