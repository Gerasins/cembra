# system-architecture

Canonical architecture repository for a single complex system implemented across multiple project repositories.

## Purpose

This repository is the source of truth for:

- system boundaries and target architecture
- canonical domain terminology
- architectural decisions
- cross-cutting architectural rules
- known divergences between the target architecture and current implementation

This repository is not a backlog, delivery plan, or implementation handbook for individual project repositories.

## Entry Points

- [`SYSTEM-MAP.md`](SYSTEM-MAP.md) for the top-level view of the system
- [`CONTEXT.md`](CONTEXT.md) for canonical system-wide terminology
- [`contexts/`](contexts/) for domain context definitions
- [`integrations/README.md`](integrations/README.md) for system integration mapping
- [`cross-cutting/README.md`](cross-cutting/README.md) for system-wide architectural rules
- [`principles/README.md`](principles/README.md) for foundational architecture principles
- [`docs/adr/`](docs/adr/) for architecture decision records
- [`governance/README.md`](governance/README.md) for repository usage and change rules

## Working Model

- The architecture described here is normative.
- Current implementation gaps are documented explicitly instead of being hidden.
- Text is canonical; diagrams are supportive.
- Domain contexts are the primary navigation model.
- Architectural changes should be captured here before or alongside implementation changes in project repositories.
