# Repository Guide

## What This Repository Is

This repository is the canonical architecture reference for the system.

## What Does Not Belong Here

- sprint tasks
- detailed delivery planning
- service-specific implementation notes with no architectural impact
- team process notes unrelated to architecture

## Canonical Document Types

- `SYSTEM-MAP.md`: top-level system overview
- `CONTEXT.md`: system-wide glossary
- `contexts/*/README.md`: domain context definitions
- `integrations/*`: cross-context interaction descriptions
- `cross-cutting/*`: system-wide architecture rules
- `principles/*`: foundational architecture principles
- `docs/adr/*`: important architecture decisions and their rationale

## Change Discipline

- Update `CONTEXT.md` when canonical terminology changes.
- Update `SYSTEM-MAP.md` when system boundaries, major repos, or major integrations change.
- Add or update context documents when boundaries or responsibilities change.
- Record meaningful architecture decisions in ADRs.
- Document divergences explicitly instead of silently letting documents drift away from reality.

## ADR Guidance

Create an ADR when all of the following are true:

- the decision is hard to reverse
- the decision would be surprising without context
- the decision came from a real trade-off

## Authority

- This repository is normative for architecture.
- Architecture owners control changes to canonical architecture documents.
- Proposed changes should be reviewed before they are treated as system truth.
