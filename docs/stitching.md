# GraphQL Stitching

## Purpose

This document defines Stitching as used in this project and contrasts it with
Fusion. Use it when discussing manual schema composition and field-level
delegation.

## Problem Statement

When multiple services expose related but separate GraphQL models, the unified
API sometimes needs custom joins that cannot be inferred from shared type
shapes. Stitching solves that problem by attaching explicit resolvers to fields
in the composed schema.

## Core Idea

Stitching is resolver-centric.

The composition model is:

1. define a field on the unified type;
2. attach a resolver for that field;
3. delegate from that resolver to the appropriate remote schema or service.

In short: take a resolver and attach it to the type field that needs it.

## Glossary

- `base schema`: the schema that defines the unified API surface.
- `delegation`: forwarding part of a field resolution to another schema or service.
- `stitched field`: a field added or completed through an explicit resolver.
- `resolver attachment`: the act of wiring a field on a type to custom execution logic.

## Execution Model

Stitching usually works as follows:

1. A base type is declared in the unified schema.
2. A related field is added to that type.
3. The field resolver extracts the parent key or context it needs.
4. The resolver delegates to the downstream schema.
5. The downstream response is mapped back into the unified result.

This makes the integration path explicit and easy to reason about.

## Example

```js
const schema = makeExecutableSchema({
  typeDefs: `
    type User {
      id: ID!
      name: String
      profile: Profile
    }

    type Profile {
      bio: String
      avatarUrl: String
    }

    type Query {
      userById(id: ID!): User
    }
  `,
  resolvers: {
    User: {
      profile: {
        fragment: "fragment UserFragment on User { id }",
        resolve(user, args, context, info) {
          return delegateToSchema({
            schema: profileSchema,
            operation: "query",
            fieldName: "profileByUserId",
            args: { id: user.id },
            context,
            info,
          });
        },
      },
    },
  },
});
```

In this example, `User.profile` does not emerge automatically from a shared
type. It exists because the unified schema explicitly attaches a resolver to it.

## When Stitching Fits Well

Stitching is a good fit when:

- the integration is field-specific rather than type-wide;
- the join logic is custom and must stay explicit;
- downstream services expose related data through different shapes;
- traceability matters more than reducing boilerplate.

## Benefits

- Explicit control over how each stitched field is resolved.
- Straightforward debugging because the delegation path is manually defined.
- Strong fit for custom joins, enrichments, and one-off integrations.

## Risks and Constraints

- Manual resolver attachment increases maintenance cost.
- Similar joins may be reimplemented across multiple types or fields.
- Schema growth can produce duplicated mappings and duplicated ownership logic.
- Performance work such as batching and caching often needs to be added manually.

## Comparison With Fusion

Stitching and Fusion solve related but different problems:

- Stitching composes fields by explicit resolver attachment.
- Fusion composes types by compatible shape and lookup selection.

If the team needs deterministic field-level delegation, prefer Stitching.
If the team needs automatic composition across shared entity types, prefer
Fusion.

## Copilot Notes

When extending docs or code related to Stitching, keep these assumptions
explicit:

- identify the stitched field;
- identify the parent key used for delegation;
- identify the downstream schema or service;
- describe whether batching, caching, or fallback behavior is required.
